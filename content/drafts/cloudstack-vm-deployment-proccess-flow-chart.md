# CloudStack VM Deployment - Complete Flow

Understanding the sync and async phases of VM deployment.

---

## SYNC PHASE (~250ms)

**Client blocks here. Must complete before job ID is returned.**

```
┌─────────────────────────────────────────────────────────────────────────┐
│ SYNC PHASE (37.044 - 37.296)                                  ~250ms    │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  HTTP POST /deployVirtualMachine                                        │
│           │                                                             │
│           ▼                                                             │
│  ┌─────────────────┐                                                    │
│  │ Authentication  │  2FA check, CIDR validation                        │
│  │ Authorization   │  Role check, resource permissions                  │
│  └────────┬────────┘                                                    │
│           ▼                                                             │
│  ┌─────────────────┐                                                    │
│  │ DB Allocation   │  vm_instance (Stopped), nics, volumes (Allocated)  │
│  │                 │  resource_count incremented                        │
│  └────────┬────────┘                                                    │
│           ▼                                                             │
│  ┌─────────────────┐                                                    │
│  │ Submit Job      │  AsyncJobManager creates job-43                    │
│  └────────┬────────┘                                                    │
│           ▼                                                             │
│  Return: { jobid: "5dc96a21-..." }  ◄── Client released                 │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

**Why sync?** Fast DB operations that RESERVE resources. Quota checks happen here. If user is over limit, fail immediately - don't waste time on expensive operations.

**What's created in DB:**
- `vm_instance` record (state: Stopped)
- `nics` record (IP assigned)
- `volumes` record (state: Allocated, no actual disk yet)
- `resource_count` incremented (vm, cpu, memory, volume, storage)

**Client responsibility:** Poll `queryAsyncJobResult` with job ID until complete.

---

## ASYNC PHASE (~5.7 sec)

**Background execution. Client polls for status.**

```
┌─────────────────────────────────────────────────────────────────────────┐
│ ASYNC PHASE (37.344 - 43.024)                                 ~5.7s     │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  job-43 (DeployVMCmdByAdmin) begins background execution                │
│                                                                         │
│    ┌──────────────┐                                                     │
│    │  PLANNING    │ ─── Find host + storage                             │
│    └──────┬───────┘                                                     │
│           ▼                                                             │
│    ┌──────────────┐                                                     │
│    │ RESERVATION  │ ─── Claim capacity on host                          │
│    └──────┬───────┘                                                     │
│           ▼                                                             │
│    ┌──────────────┐                                                     │
│    │ NETWORK PREP │ ─── Configure VR (DHCP, password, metadata)         │
│    └──────┬───────┘                                                     │
│           ▼                                                             │
│    ┌──────────────┐                                                     │
│    │ VOLUME PREP  │ ─── Copy template to primary storage                │
│    └──────┬───────┘                                                     │
│           ▼                                                             │
│    ┌──────────────┐                                                     │
│    │  VM START    │ ─── libvirt creates and starts VM                   │
│    └──────┬───────┘                                                     │
│           ▼                                                             │
│    ┌──────────────┐                                                     │
│    │ COMPLETION   │ ─── State → Running, job marked success             │
│    └──────────────┘                                                     │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

**Why async?** These operations are SLOW - external I/O to agents, storage, hypervisor. Can fail and need retry logic. Client shouldn't wait.

---

### STAGE 1: PLANNING

```
┌─────────────────────────────────────────────────────────────────────────┐
│ STAGE 1: PLANNING (37.344 - 37.407)                           ~60ms     │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  CapacityManager          StorageAllocatorChain                         │
│  ┌──────────────┐         ┌─────────────────────────────────┐           │
│  │ Host 1:      │         │ LocalStoragePoolAllocator  SKIP │           │
│  │ CPU: 6500 ✓  │         │ ClusterScopeAllocator      SKIP │           │
│  │ RAM: 6.27GB ✓│         │ ZoneWideAllocator    ──►  FOUND │           │
│  └──────────────┘         └─────────────────────────────────┘           │
│                                         │                               │
│                                         ▼                               │
│                           Destination: Host(1) + Pool(1)                │
└─────────────────────────────────────────────────────────────────────────┘
```

**What's happening:** CloudStack needs to figure out WHERE to place this VM. Two questions need answering: which host has enough CPU/RAM, and which storage pool can hold the disk.

**Host selection:** CapacityManager checks each host in the cluster. Host 1 has 6500 MHz CPU free (we need 500) and 6.27 GB RAM free (we need 512 MB). Passes.

**Storage selection:** Allocators run in chain. LocalStoragePoolAllocator skips because service offering doesn't require local disk. ClusterScopeStoragePoolAllocator finds nothing. ZoneWideStoragePoolAllocator finds "primary" NFS pool with enough space.

**Output:** A deployment destination tuple - Host 1 + Storage Pool 1. This gets passed to all subsequent stages.

---

### STAGE 2: RESERVATION

```
┌─────────────────────────────────────────────────────────────────────────┐
│ STAGE 2: RESERVATION (39.100 - 39.114)                        ~14ms     │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  op_host_capacity UPDATE:                                               │
│  ┌────────────────────────────────────┐                                 │
│  │ CPU:  1500 → 2000 MHz   (+500)     │                                 │
│  │ RAM:  2 GB → 2.5 GB     (+512 MB)  │                                 │
│  └────────────────────────────────────┘                                 │
│                                                                         │
│  State: Stopped → Starting                                              │
└─────────────────────────────────────────────────────────────────────────┘
```

**What's happening:** Now that we've picked a host, we need to CLAIM those resources so no other VM deployment grabs them while we're still working.

**Capacity reservation:** `op_host_capacity` table gets updated. This is CloudStack's internal bookkeeping - not the actual hypervisor. Other deployments running in parallel will see reduced capacity and may choose different hosts.

**State machine:** VM transitions from `Stopped` to `Starting`. This is the first state change since allocation. If anything fails after this point, cleanup handlers will see `Starting` state and know to release reservations.

**Why so fast:** Pure database operations, no external I/O.

---

### STAGE 3: NETWORK PREP ⚠️ SLOWEST

```
┌─────────────────────────────────────────────────────────────────────────┐
│ STAGE 3: NETWORK PREP (39.170 - 41.622)                       ~2.4s     │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  Commands to Virtual Router (r-6-VM):                                   │
│                                                                         │
│  ┌──────────────────┐  ┌──────────────────┐  ┌──────────────────┐       │
│  │ DhcpEntryCommand │  │SavePasswordCommand│  │  VmDataCommand   │      │
│  ├──────────────────┤  ├──────────────────┤  ├──────────────────┤       │
│  │ MAC: 02:01:00:cc │  │ Sets VM password │  │ Instance metadata│       │
│  │ IP:  10.1.1.186  │  │ for cloud-init   │  │ for cloud-init   │       │
│  │ GW:  10.1.1.1    │  │                  │  │                  │       │
│  │ DNS: 10.1.1.1    │  │                  │  │                  │       │
│  └────────┬─────────┘  └────────┬─────────┘  └────────┬─────────┘       │
│           │                     │                     │                 │
│           ▼                     ▼                     ▼                 │
│       vm_dhcp_entry.json   vm_password.json    vm_metadata.json         │
│                                                                         │
│  Router IP: 169.254.208.178 (link-local control interface)              │
└─────────────────────────────────────────────────────────────────────────┘
```

**What's happening:** Before the VM boots, the network must be ready. The Virtual Router needs to know about this new VM so it can serve DHCP, provide the password, and offer metadata.

**DhcpEntryCommand:** Tells VR "when MAC 02:01:00:cc asks for DHCP, give it IP 10.1.1.186 with gateway 10.1.1.1". VR writes this to `/etc/dhcphosts.txt` or equivalent config.

**SavePasswordCommand:** VM will have a random password. VR stores it so when VM boots and cloud-init queries the metadata service, it gets the password to set for the default user.

**VmDataCommand:** Instance metadata (VM name, instance ID, SSH keys if any, userdata scripts). Cloud-init fetches this from `http://169.254.169.254/` which VR serves.

**Why so slow:** Three sequential network round-trips to the VR. Each command goes: Management Server → Agent on Host → VR (via control NIC 169.254.x.x) → wait for response → back. Network latency adds up.

---

### STAGE 4: VOLUME PREP

```
┌─────────────────────────────────────────────────────────────────────────┐
│ STAGE 4: VOLUME PREP (41.623 - 41.744)                        ~120ms    │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  ┌─────────────────┐         CopyCommand        ┌─────────────────┐     │
│  │ SECONDARY NFS   │ ─────────────────────────► │  PRIMARY NFS    │     │
│  │                 │                            │                 │     │
│  │ Template 202    │      AncientDataMotion     │ Volume ROOT-7   │     │
│  │ Ubuntu 24.04    │         Strategy           │ d3e16aa2-...    │     │
│  │ Format: RAW     │                            │ Format: QCOW2   │     │
│  │ ~600 MB         │                            │ ~600 MB         │     │
│  └─────────────────┘                            └─────────────────┘     │
│                                                                         │
│  Volume state: Allocated → Ready                                        │
└─────────────────────────────────────────────────────────────────────────┘
```

**What's happening:** The VM needs a disk. We have a template (golden image) on secondary storage. We need to copy it to primary storage where the VM can actually use it.

**Template location:** Secondary storage holds templates, ISOs, snapshots. It's the "library". Primary storage is where running VMs keep their disks.

**CopyCommand:** Agent copies the template file. Format conversion may happen (RAW → QCOW2). QCOW2 is preferred because it supports thin provisioning and snapshots.

**AncientDataMotion:** Yes, that's the actual class name. It's the original/default data motion strategy. Handles simple copy operations.

**Why only 120ms:** This template was already cached or it's a small template. Large templates (10+ GB) can take minutes here. This is often the bottleneck in real deployments.

**State change:** Volume goes from `Allocated` (just a DB record) to `Ready` (actual bytes on disk).

---

### STAGE 5: VM START

```
┌─────────────────────────────────────────────────────────────────────────┐
│ STAGE 5: VM START (41.775 - 42.900)                           ~1.1s     │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  StartCommand → KVM Agent → libvirt                                     │
│                                                                         │
│  ┌─────────────────────────────────────────────────────────────────┐    │
│  │ VM Spec:                                                        │    │
│  │   vCPU: 1 @ 500 MHz                                             │    │
│  │   RAM:  512 MB                                                  │    │
│  │   Disk: /primary/d3e16aa2-... (QCOW2)                           │    │
│  │   NIC:  02:01:00:cc:00:05 → cloudbr0 (VLAN 181)                 │    │
│  └─────────────────────────────────────────────────────────────────┘    │
│                                                                         │
│  libvirt: virDomainDefineXML() → virDomainCreate()                      │
│                                                                         │
│  StartAnswer: success, VNC @ 10.0.113.150                               │
└─────────────────────────────────────────────────────────────────────────┘
```

**What's happening:** Everything is ready. Time to actually boot the VM on the hypervisor.

**StartCommand:** Management server sends full VM specification to the KVM agent. Contains everything: CPU model, RAM size, disk paths, NIC configuration with MAC and VLAN tag.

**Agent processing:** CloudStack agent on the KVM host receives this, translates it into libvirt XML domain definition.

**libvirt calls:**
1. `virDomainDefineXML()` - register the VM definition
2. `virDomainCreate()` - actually start it

**What libvirt does:** Creates the QEMU process, sets up virtio devices, attaches disk, creates tap interface and connects it to cloudbr0 with the right VLAN tag, starts CPU emulation.

**StartAnswer:** Agent reports back success. Includes VNC port so console proxy can connect for remote access.

**Why 1.1s:** Hypervisor needs to allocate resources, QEMU needs to start, VM BIOS begins POST. We're not waiting for full OS boot - just for QEMU process to be running.

---

### STAGE 6: COMPLETION

```
┌─────────────────────────────────────────────────────────────────────────┐
│ STAGE 6: COMPLETION (42.950 - 43.024)                         ~74ms     │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  State: Starting → Running                                              │
│                                                                         │
│  job-44 (VmWorkStart)        → SUCCEEDED                                │
│  job-43 (DeployVMCmdByAdmin) → SUCCEEDED + UserVmResponse               │
│                                                                         │
│  Client polling queryAsyncJobResult gets VM details                     │
└─────────────────────────────────────────────────────────────────────────┘
```

**What's happening:** Cleanup and notification. VM is running, now we update all the bookkeeping.

**State transition:** `Starting` → `Running`. This is the final state. VM is now operational.

**Job completion:** Two jobs were involved:
- job-44 (VmWorkStart) - the inner job that did the actual work
- job-43 (DeployVMCmdByAdmin) - the outer job the client is polling

**UserVmResponse:** Full VM details packaged into response object. Contains UUID, IP addresses, host it's running on, etc.

**Client gets notified:** Next poll to `queryAsyncJobResult` returns jobstatus=1 (succeeded) with the VM response payload.

---

## TIMELINE

```
        SYNC                              ASYNC
    ◄─── 250ms ───►  ◄──────────────────── 5.7s ──────────────────────►

37.0    37.3        38.0    39.0    40.0    41.0    42.0    43.0
 │       │           │       │       │       │       │       │
 │▓▓▓▓▓▓▓│           │       │       │       │       │       │
 │ Auth  │           │       │       │       │       │       │
 │ DB    │           │       │       │       │       │       │
 │ Job   │           │       │       │       │       │       │
 │       │           │       │       │       │       │       │
 │       ├───────────┴───────┴───────┴───────┴───────┴───────┤
 │     JobID         │                                       │
 │    returned       │▓▓│                                    │  PLANNING (60ms)
 │       │           │  │▓│                                  │  RESERVATION (14ms)
 │       │           │   │▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓│             │  NETWORK PREP (2.4s)
 │       │           │                         │▓▓│          │  VOLUME PREP (120ms)
 │       │           │                            │▓▓▓▓▓▓▓▓│ │  VM START (1.1s)
 │       │           │                                     │▓│  COMPLETION (74ms)
 │       │           │                                       │
 │       │           └───────────────────────────────────────┘
 │       │
 ▼       ▼
Client   Client
waiting  polling
```

---

## Summary

| Phase | Duration | What Happens | Client State |
|-------|----------|--------------|--------------|
| **SYNC** | 250ms | Auth, DB reserve, return job ID | Blocking |
| **ASYNC** | 5.7s | Planning → Reservation → Network → Volume → Start → Complete | Polling |

**Key insight:** SYNC reserves cheap (DB records), ASYNC provisions expensive (actual resources).
