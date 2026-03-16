# Macvlan
This project is about the use and configuration of the macvlan.

## Reference
1. https://github.com/containernetworking/plugins/tree/main/plugins/main/macvlan
2. https://www.cni.dev/plugins/current/main/macvlan/
3. https://github.com/nmstate/kubernetes-nmstate

## Check and set up the environment
1. Enable the relevant kernel modules on all nodes of the cluster.
```console
# Check if the modules exist
lsmod | grep -i macvlan

# Load the modules
sudo modprobe macvlan
```
2. Install nmstate-operator in this cluster for node network management  
Reference the ptah 'kube-network/node-network-configuration/kubernetes-nmstate'

3. Install the CNI plugin binary for macvlan if it dose not exist.  
The K8S cluster installed based on RKE does not have the macvlan plugin by default. Check the rke version installed on the cluster.
```console
rke version
```

4. Download the plugin macvlan to the path /opt/cni/bin/ of all nodes
```console
# Download the CNI plugin
CNI_VERSION="v1.6.2"
wget "https://github.com/containernetworking/plugins/releases/download/${CNI_VERSION}/cni-plugins-linux-amd64-${CNI_VERSION}.tgz"  

# Extract to CNI directory
tar -zxvf cni-plugins-linux-amd64-${CNI_VERSION}.tgz ./
sudo cp ./macvlan /opt/cni/bin/
```

5. Install multus-cni
```console
kubectl apply -f https://raw.githubusercontent.com/k8snetworkplumbingwg/multus-cni/master/deployments/multus-daemonset-thick.yml
```

## Use
1. Store a configuration as a Custom Resource
```console
kubectl apply -f ./macvlan-enp4s0-2000-test.yaml
kubectl get network-attachment-definitions
```

2. Create two pods with net1 interface  
The IP of net1 interface in pod busybox-a is 2.1.0.10, The IP of net1 interface in pod busybox-b is 2.1.0.11.
```console
kubectl apply -f ./busybox-pods.yaml
```

3. Test the network connection of the net1 interface between two pods
```console
kubectl exec -it busybox-a -- sh
ping -I net1 2.1.0.11
```

