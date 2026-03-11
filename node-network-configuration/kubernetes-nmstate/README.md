# kubernetes-nmstate
This project is about the use and configuration of the kubernetes-nmstate. Declarative node network configuration driven through Kubernetes API.

## Reference
1. https://nmstate.io/
2. https://github.com/nmstate/kubernetes-nmstate
3. https://nmstate.io/kubernetes-nmstate/deployment/arbitrary-cluster/

## Install
1. Install network-manager on all nodes of the cluster.
```console
# Install the service
sudo apt update
sudo apt install network-manager

# Check the status of service
systemctl status NetworkManager
nmcli -v
nmcli device status
```

2. Install nmstate-operator in this cluster for node network management.  
The currently installed version of NetworkManager is 1.46.0, so it is necessary to install 0.15.0 or above. Please refer to the connection for details  
https://github.com/nmstate/kubernetes-nmstate/blob/main/CONTRIBUTING.md#networkmanager-compatibility   

```console
# First, install kubernetes-nmstate operator:
kubectl apply -f https://github.com/nmstate/kubernetes-nmstate/releases/download/v0.85.1/nmstate.io_nmstates.yaml
kubectl apply -f https://github.com/nmstate/kubernetes-nmstate/releases/download/v0.85.1/namespace.yaml
kubectl apply -f https://github.com/nmstate/kubernetes-nmstate/releases/download/v0.85.1/service_account.yaml
kubectl apply -f https://github.com/nmstate/kubernetes-nmstate/releases/download/v0.85.1/role.yaml
kubectl apply -f https://github.com/nmstate/kubernetes-nmstate/releases/download/v0.85.1/role_binding.yaml
kubectl apply -f https://github.com/nmstate/kubernetes-nmstate/releases/download/v0.85.1/operator.yaml

# Once that's done, create an NMState CR, triggering deployment of kubernetes-nmstate handler:
cat <<EOF | kubectl create -f -
apiVersion: nmstate.io/v1
kind: NMState
metadata:
  name: nmstate
EOF

# Check the network status of all nodes. 
kubectl get NodeNetworkState

# Search for related podcasts.
kubectl get po -n nmstate
```

## Use
### Create a network policy using CRD NodeNetworkConfigurationPolicy
1. Create a network interface with macvlan
```yaml
apiVersion: nmstate.io/v1
kind: NodeNetworkConfigurationPolicy
metadata:
  name: vlan100
spec:
  nodeSelector:
    kubernetes.io/hostname: node01
  desiredState:
    interfaces:
    - name: eth1.100
      type: vlan
      state: up
      vlan:
        base-iface: eth1
        id: 100
```
