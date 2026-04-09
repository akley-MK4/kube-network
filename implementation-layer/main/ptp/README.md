# Macvlan
This project is about the use and configuration of the ptp.

## Reference
1. https://www.cni.dev/plugins/current/main/ptp/

## Flowsheet
```plantuml
@startuml
skinparam backgroundColor #FFFFFF
skinparam component {
  BackgroundColor #E3F2FD
  BorderColor #1565C0
  FontColor #0D47A1
  ArrowColor #1565C0
  ArrowFontColor #0D47A1
}
skinparam note {
  BackgroundColor #FFF8E1
  BorderColor #FF8F00
  FontColor #5D4037
}

title Kubernetes Pod with ptp CNI Network Topology (Simplified)

package "Kubernetes Node" {
  frame "Host Network Namespace" as host {
    component "vethXXXXXX (host end)" as veth_host
    
    component "Physical Interface" as eth0
    
    component "Routing Table" as routing
    
    component "ptp CNI Plugin" as cni_ptp
  }
  
  frame "Pod Network Namespace (ptp-busybox-a)" as pod {
    component "veth Pair (pod end)" as veth_pod
    
    component "Pod Routing Table" as pod_routing
  }
}

' NetworkAttachmentDefinition configuration
component "NetworkAttachmentDefinition" as nad

component "Pod Annotation" as annotation

' External connections
component "Kubernetes Network" as k8s_net

component "External Network" as external

' Connections
veth_host -[hidden]- veth_pod : "veth pair"

cni_ptp --> veth_host : "configures"
cni_ptp --> veth_pod : "configures"
cni_ptp --> pod_routing : "configures"

veth_host --> routing : "packets to/from pod"
routing --> eth0 : "external traffic"

eth0 --> k8s_net : "default CNI traffic"
veth_host --> external : "ptp network traffic"

veth_pod --> pod_routing : "uses"

' Configuration flow
nad --> cni_ptp : "configures"
annotation --> cni_ptp : "triggers"

' Notes
note right of cni_ptp
  <b>ptp CNI Workflow:</b>
  1. Reads NAD configuration
  2. Creates veth pair
  3. Moves one end to pod namespace
  4. Assigns IP (2.1.3.10) from IPAM
  5. Configures host-side gateway (2.1.3.1)
  6. Sets up routing on host
end note

note top of veth_host
  <b>Host-side veth endpoint:</b>
  Interface: vethXXXXXX
  IP: 2.1.3.1/32
  • Acts as gateway for pod
  • /32 address for point-to-point link
  • Kernel routes pod traffic
end note

note top of eth0
  <b>Physical Interface:</b>
  Interface: eth0
  IP: 192.168.1.100/24
end note

note left of routing
  <b>Host Routing Rules:</b>
  • default via 192.168.1.1 dev eth0
  • 2.1.3.10 dev vethXXXXXX scope link
  • Direct route to pod
  • Default traffic via eth0
end note

note bottom of veth_pod
  <b>Pod network interface:</b>
  Interface: net1
  IP: 2.1.3.10/24
  Gateway: 2.1.3.1
  • Created by ptp CNI plugin
  • Uses ptp point-to-point network
  • All traffic goes through host
end note

note right of pod_routing
  <b>Pod Routing Rules:</b>
  • default via 2.1.3.1 dev net1
  • 2.1.3.0/24 dev net1 scope link
  • Default route via host veth
  • Local network via net1
end note

note right of nad
  <b>NetworkAttachmentDefinition:</b>
  name: ptp-test
  type: ptp
  subnet: 2.1.3.0/24
  gateway: 2.1.3.1
  IP range: 2.1.3.10-2.1.3.100
  Defines ptp network configuration
  IPAM: host-local
end note

note right of annotation
  <b>Pod Annotation:</b>
  k8s.v1.cni.cncf.io/networks: ptp-test
  Specifies which network to attach
  Triggers CNI plugin execution
  Result: net1 interface in pod
end note

note right of k8s_net
  <b>Kubernetes Network:</b>
  Cluster network
  Service discovery
  Pod-to-pod communication
  Default network for pods
end note

note right of external
  <b>External Network:</b>
  Internet
  Other networks
  External services
  Outside of Kubernetes cluster
end note

@enduml
```

```plantuml
@startuml
skinparam backgroundColor #FFFFFF
skinparam rectangle {
  BackgroundColor #F5F5F5
  BorderColor #757575
  FontColor #212121
  Shadowing false
  RoundCorner 5
}
skinparam arrow {
  Color #1565C0
  FontColor #0D47A1
}
skinparam note {
  BackgroundColor #FFF8E1
  BorderColor #FF8F00
  FontColor #5D4037
}

title Pod Network Communication Flow (eth0 as default, net1 as ptp)

start

:Pod Application\nsends network request
e.g. ping 8.8.8.8
or curl http://example.com;

:Pod Network Stack
processes request
determines source IP;

:Check Pod Routing Table
for destination IP;

if (Destination is 2.1.3.0/24?) then (yes)
  :Use net1 interface
  (ptp network, same subnet);
  :Source: 2.1.3.10
  Gateway: 2.1.3.1;
  note right
    <b>net1 interface</b>
    Local ptp network traffic
  end note
else (other destinations)
  if (Destination is cluster network?) then (yes)
    :Use eth0 interface
    (default CNI network);
    :Source: eth0 IP
    Gateway: via eth0;
    note right
      <b>eth0 interface</b>
      Cluster network traffic
    end note
  else (external network)
    :Use eth0 (default route)
    via default gateway;
    note right
      <b>eth0 default route</b>
      All external traffic
    end note
  endif
endif

if (Using net1 interface?) then (yes)
  :Prepare network packet
  Source: 2.1.3.10 via net1;
  :Send via veth pair
  to host veth endpoint (2.1.3.1);
  :Host processes packet
  in kernel network stack;
  if (Destination is 2.1.3.0/24?) then (yes)
    :Deliver within ptp network
    via host routing;
  else (other destinations)
    :Forward to external network
    via physical interface;
  endif
else (using eth0)
  :Send via eth0
  to cluster/external network;
endif

:Destination receives packet
processes request;

:Send response
reverse the same path
back to pod;

:Pod receives response
delivers to application;

stop

@enduml

```

## Limitations on use
1. Avoid using PTP for Pod to Pod communication, PTP is designed for point-to-point connections and is not suitable for multi Pod shared networks.
2. Each vet pair is completely independent and does not share a broadcast domain.
3. After the packet sent from Pod through net1 reaches the host, the host will not perform source address translation (SNAT) for packets from 2.1.3.0/24 by default. The external network is unable to route the reply back to this private address.
