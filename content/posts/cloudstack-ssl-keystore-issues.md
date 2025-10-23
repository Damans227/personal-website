---
title: "CloudStack KVM Host Add Failure - Part 2"
date: 2025-10-22T15:33:37-05:00
draft: false
tags:
  - cloudstack
  - kvm
---

# CloudStack KVM Host Add Failure - Part 2

This document captures the logs and analysis of a failure while attempting to add a KVM host to a CloudStack zone, due to issues setting up the agent certificate keystore.

---

## Error Summary

```
DiscoveryException: Could not add host at [http://192.168.122.10]
...
Failed to setup certificate in the KVM agent's keystore file,
please see logs and configure manually!
```

```java
2025-10-04 23:41:58,838 INFO [c.c.a.ApiServer] (qtp1438988851-22:[ctx-b8b55cff, ctx-2b748765]) (logid:046ec553) 
Could not add host at [http://192.168.122.10] with zone [Zone {"id": "2", "name": "MyAdvancedZone", "uuid": "30d1810a-9680-49c7-8926-75fcf5e4dfb8"}], pod [HostPod {"id":1,"name":"Pod1","uuid":"996b281e-a200-4a64-9cd2-9f7fc8d0d55a"}] and cluster [Cluster {id: "1", name: "Cluster1", uuid: "c6981176-f03d-45dd-aba8-bbe3ab38e6f4"}] 
due to: [ can't setup agent, due to com.cloud.utils.exception.CloudRuntimeException: Failed to setup certificate in the KVM agent's keystore file, please see logs and configure manually! - Failed to setup certificate 
in the KVM agent's keystore file, please see logs and configure manually!]. 2025-10-04 23:41:58,839 

DEBUG [c.c.a.ApiServlet] (qtp1438988851-22:[ctx-b8b55cff, ctx-2b748765]) (logid:046ec553) ===END=== 192.168.122.1 -- POST addHost 2025-10-04 23:41:58,926 INFO [c.c.c.ClusterManagerImpl] (Cluster-Heartbeat-1:[ctx-b86853cb]) (logid:be4ed135) No inactive management server node found 2025-10-04 23:41:58,927 
DEBUG [c.c.c.ClusterManagerImpl] (Cluster-Heartbeat-1:[ctx-b86853cb]) (logid:be4ed135) Peer scan is finished. profiler: Done. 
```

---

## Root Cause

The management server is unable to import its SSL certificate into the Java keystore of the KVM host's agent. This is necessary for secure agent-to-management-server communication.

CloudStack requires this certificate to be in place before it can register and communicate with the host agent.

---

## Solution — Manual Keystore Setup

### 1. Export or Generate the Management Certificate

If your management server has a certificate available:

```bash
cp /etc/cloudstack/management/server-cert.pem /etc/cloudstack/agent/cloudstack-management-ca.pem
```

If it does **not** exist (as seen in your logs), create a self-signed cert:

```bash
mkdir -p /etc/cloudstack/agent

openssl req -x509 -newkey rsa:2048 -keyout /etc/cloudstack/agent/cloudstack-ca.key \
  -out /etc/cloudstack/agent/cloudstack-management-ca.pem \
  -days 3650 -nodes -subj "/CN=cloudstack-ca"
```

---

### 2. Import the CA Cert into the Agent Keystore

```bash
keytool -importcert \
  -alias cloudstack-ca \
  -file /etc/cloudstack/agent/cloudstack-management-ca.pem \
  -keystore /etc/cloudstack/agent/cloudstack-agent.keystore \
  -storepass cloudstack \
  -noprompt
```

---

### 3. Restart the Agent

```bash
systemctl restart cloudstack-agent
```

Watch logs:

```bash
tail -f /var/log/cloudstack/agent/agent.log
```

---

### 4. Re-attempt Adding Host via UI

Go to:

> CloudStack UI → Infrastructure → Add Host

Use:

- Host: `192.168.122.10`
- Username: `darora`
- Password: (or SSH key)
- Hypervisor: `KVM`

You should now see the host being added successfully.

---

## Optional Debugging

To inspect the keystore:

```bash
keytool -list -keystore /etc/cloudstack/agent/cloudstack-agent.keystore -storepass cloudstack
```

Look for:

```
cloudstack-ca, Oct 4, 2025, trustedCertEntry
```

---

## Alternative: Disable Secure Communication (Not Recommended for Prod)

Only for dev/testing:

Edit `/etc/cloudstack/management/server.properties`:

```
agent.secure.communication=false
```

Then:

```bash
systemctl restart cloudstack-management
```

This bypasses certificate verification but weakens security.

---

## Final Verification

Once the host is added:

- Check `/var/log/cloudstack/management/management-server.log` for `Successfully added host`
- Check `/var/log/cloudstack/agent/agent.log` for agent startup messages
- Try deploying a test VM

---

##  Appendix: Relevant Logs

...
CloudRuntimeException: Failed to setup certificate in the KVM agent's keystore file, please see logs and configure manually!
...
AddHostCmd: Could not add host at [http://192.168.122.10] due to keystore error