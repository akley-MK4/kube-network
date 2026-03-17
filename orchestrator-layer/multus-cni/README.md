# Multus CNI
This project is about the use and configuration of the Multus CNI.

## Reference
1. https://github.com/k8snetworkplumbingwg/multus-cni
2. https://www.cni.dev/plugins/current/main/macvlan/

## Add certificates for secure authentication
1. Create the CA issuer
```console
kubectl create namespace nginx-gateway
```
```console
kubectl apply -f ca-issuer.yaml -n nginx-gateway
```
2. Create server and client certificates
```console
kubectl apply -f server-tls.yaml -n nginx-gateway
```
```console
kubectl apply -f agent-tls.yaml -n nginx-gateway
```
3. Confirm the Secrets have been created  
You should see the Secrets created in the nginx-gateway namespace:
```console
kubectl -n nginx-gateway get secrets
```

## Install NGINX Gateway Fabric with Helm
### Deploy
1. Install the ctrl plane from the OCI registry
```console
helm install ngf oci://ghcr.io/nginx/charts/nginx-gateway-fabric --create-namespace -n nginx-gateway -f ./values.yaml --version 2.3.0
```

### Upgrade
```console
helm upgrade ngf oci://ghcr.io/nginx/charts/nginx-gateway-fabric -n nginx-gateway -f ./values.yaml
```

### Uninstall
```console
helm uninstall ngf -n nginx-gateway
```

## Deploy a Gateway for data plane instances  
Create an example in the dataplane-example directory to demonstrate how to deploy a dataplane instance

### Test
```console
export GW_HTTP_PORT=30000
export GW_HTTPS_PORT=30001
curl --resolve nginx-hello.example.com:$GW_HTTP_PORT:$GW_IP http://nginx-hello.example.com:$GW_HTTP_PORT/v1
curl --resolve nginx-echo.example.com:$GW_HTTPS_PORT:$GW_IP https://nginx-echo.example.com:$GW_HTTPS_PORT -k
```
