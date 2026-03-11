# kubernetes-nmstate
This project is about the use and configuration of the kubernetes-nmstate. Declarative node network configuration driven through Kubernetes API.

## Reference
1. https://nmstate.io/
2. https://github.com/nmstate/kubernetes-nmstate
3. https://nmstate.io/kubernetes-nmstate/deployment/arbitrary-cluster/

## Install
### 1. Install network-manager on all nodes of the cluster  
#### 1.1. Install the service.  
```console
sudo apt update
sudo apt install network-manager
```  

#### 1.2. Replace systemd-networkd with NetworkManager if systemd-networkd is enabled.  
1.2.1. Check if systemd-networkd is enabled.
```console
systemctl status systemd-networkd
```
1.2.2. Edit /etc/netplan/00-installer-config.yaml to add renderer: NetworkManager if it is in the Ubuntu.
```console
network:
  version: 2
  renderer: NetworkManager
```
1.2.3. Update modifications to netplan.
```console
sudo netplan apply -f /etc/netplan/00-installer-config.yaml
```
1.2.4. Enable NetworkManager.
```console
sudo systemctl enable NetworkManager
```
1.2.5. Disable systemd-networkd if it is enabled.
```console
sudo systemctl disable --now systemd-networkd
```
1.2.6. Restarting after completing these operations may be a good choice.
```console
sudo reboot
```

#### 1.3. Check if the network interface has been taken over by the NetworkManager.
```console
systemctl status NetworkManager
nmcli device status
```

### 2. Install nmstate-operator in this cluster for node network management
The currently installed version of NetworkManager is 1.46.0, so it is necessary to install 0.15.0 or above. Please refer to the connection for details  
https://github.com/nmstate/kubernetes-nmstate/blob/main/CONTRIBUTING.md#networkmanager-compatibility   

```console
# Query the currently installed version of NetworkManager
nmcli -v

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

## CRD
1. NodeNetworkConfigurationPolicy (nncp)
1. NodeNetworkConfigurationEnactment (nnce)
3. NodeNetworkState (nns)

## Use
### Create a network policy using CRD NodeNetworkConfigurationPolicy
1. Create a network interface with vlan.
```console
# Check if kernel module '8021q' is enabled
lsmod | grep -i 8021q
sudo modprobe 8021q

# The following yaml file will trigger the reconcile of nmstate-operator, which will create the vlan interface on the nodes.
kubectl apply -f ./vlan-enp4s0-2000-test.yaml
```

2. Check the normal status of the interface
```console
# Check the status of the nncp
kubectl get nncp vlan-enp4s0-2000-test

# Check the status of ip link
ip -d link show ip -d link show enp4s0.2000
```

3. Check the error status of this interface
```console
# Check the logs of the nmstate-handler pod 
kubectl logs nmstate-handler-jflhd -n nmstate -f

# Check if it has been taken over by NetworkManager
nmcli device status |grep 'enp4s0.2000'
```
