```
HTTP POST /deployVirtualMachine
├── ApiServlet.processRequest()
├── Authentication & Authorization
│   ├── 2FA Check
│   ├── CIDR Validation
│   ├── DynamicRoleBasedAPIAccessChecker
│   └── ApiRateLimitService
├── Resource Validation
│   ├── AccountManagerImpl.checkAccess() [multiple]
│   ├── NetworkModelImpl.checkNetworkPermissions()
│   └── UserDataManagerImpl.validateUserData()
├── DB Allocation (Synchronous)
│   ├── UserVmManagerImpl.allocate()
│   ├── VirtualMachineManagerImpl.allocate()
│   ├── NetworkOrchestrator.allocateNic()
│   ├── VolumeOrchestrator.allocateRawVolume()
│   └── ResourceLimitManagerImpl.incrementResourceCount() [5x]
├── AsyncJobManagerImpl.submitAsyncJob(job-43)
│   └── Return job ID to client
└── Background Execution (job-43)
    ├── DeploymentPlanningManagerImpl.planDeployment()
    │   ├── CapacityManagerImpl.checkHostCapacity()
    │   ├── LocalStoragePoolAllocator (skip)
    │   ├── ClusterScopeStoragePoolAllocator (skip)
    │   └── ZoneWideStoragePoolAllocator (found primary)
    ├── VmWorkStart job-44
    │   ├── State: Stopped → Starting
    │   ├── CapacityManagerImpl.allocateVmCapacity()
    │   ├── NetworkOrchestrator.prepare()
    │   │   ├── DhcpEntryCommand → VR
    │   │   ├── SavePasswordCommand → VR
    │   │   └── VmDataCommand → VR
    │   ├── VolumeOrchestrator.prepare()
    │   │   └── CopyCommand (template → volume)
    │   ├── StartCommand → KVM Host
    │   └── State: Starting → Running
    └── Complete job-43 with UserVmResponse
    ```
