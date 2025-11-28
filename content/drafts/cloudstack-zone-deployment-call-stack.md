```
createZone(
    name: "zone1",
    networktype: "Advanced",
    dns1: "8.8.8.8",
    internaldns1: "10.0.113.1",
    guestcidraddress: "10.1.1.0/24"
) → zone_id

createPhysicalNetwork(
    zoneid: zone_id,
    name: "Physical Network 1",
    isolationmethods: "VLAN"
) → physical_network_id

addTrafficType(
    physicalnetworkid: physical_network_id,
    trafficType: "Guest"
)

addTrafficType(
    physicalnetworkid: physical_network_id,
    trafficType: "Management"
)

addTrafficType(
    physicalnetworkid: physical_network_id,
    trafficType: "Public"
)

updatePhysicalNetwork(
    id: physical_network_id,
    state: "Enabled"
)

listNetworkServiceProviders(
    physicalNetworkId: physical_network_id,
    name: "VirtualRouter"
) → vr_provider_id

listVirtualRouterElements(
    nspid: vr_provider_id
) → vr_element_id

configureVirtualRouterElement(
    id: vr_element_id,
    enabled: true
)

updateNetworkServiceProvider(
    id: vr_provider_id,
    state: "Enabled"
)

listNetworkServiceProviders(
    physicalNetworkId: physical_network_id,
    name: "Ovs"
) → (empty)

listNetworkServiceProviders(
    physicalNetworkId: physical_network_id,
    name: "Internallbvm"
) → internallb_provider_id

listNetworkServiceProviders(
    physicalNetworkId: physical_network_id,
    name: "VpcVirtualRouter"
) → vpc_vr_provider_id

listInternalLoadBalancerElements(
    nspid: internallb_provider_id
) → internallb_element_id

listVirtualRouterElements(
    nspid: vpc_vr_provider_id
) → vpc_vr_element_id

configureInternalLoadBalancerElement(
    id: internallb_element_id,
    enabled: true
)

configureVirtualRouterElement(
    id: vpc_vr_element_id,
    enabled: true
)

updateNetworkServiceProvider(
    id: internallb_provider_id,
    state: "Enabled"
)

updateNetworkServiceProvider(
    id: vpc_vr_provider_id,
    state: "Enabled"
)

createPod(
    zoneId: zone_id,
    name: "pod1",
    gateway: "10.0.113.1",
    netmask: "255.255.255.0",
    startIp: "10.0.113.181",
    endIp: "10.0.113.200"
) → pod_id

createVlanIpRange(
    zoneId: zone_id,
    vlan: "untagged",
    gateway: "10.0.113.1",
    netmask: "255.255.255.0",
    startip: "10.0.113.160",
    endip: "10.0.113.180",
    forVirtualNetwork: true
)

updatePhysicalNetwork(
    id: physical_network_id,
    vlan: "100-200"
)

addCluster(
    zoneId: zone_id,
    podId: pod_id,
    hypervisor: "KVM",
    clustertype: "CloudManaged",
    arch: "x86_64",
    clustername: "cluster1"
) → cluster_id

addHost(
    zoneid: zone_id,
    podid: pod_id,
    clusterid: cluster_id,
    hypervisor: "KVM",
    username: "root",
    password: "****",
    url: "http://10.0.113.150"
) → host_id

createStoragePool(
    zoneid: zone_id,
    podId: pod_id,
    clusterid: cluster_id,
    name: "primary",
    scope: "zone",
    hypervisor: "KVM",
    url: "nfs://10.0.113.150/export/primary"
) → storage_pool_id

addImageStore(
    zoneid: zone_id,
    name: "secondary",
    provider: "NFS",
    url: "nfs://10.0.113.150/export/secondary"
) → imagestore_id
```
