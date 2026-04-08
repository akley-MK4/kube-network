# Macvlan
This project is about the use and configuration of the vlan.

## Reference
1. https://www.cni.dev/plugins/current/main/vlan/

## Limitations on use
1. The same VLAN type of NetworkAttachmentDefinition cannot be used by multiple Pods simultaneously,   
as they will compete for the same host level VLAN sub interface.

