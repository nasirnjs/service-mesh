
## Prerequisites
Two k8s Cluster

Rename Cluster context as your wish hre East & West

Install step for mtls [References](https://smallstep.com/docs/step-cli/installation/#debian-ubuntu)


## Steps1: Install Linkerd CLI [References](https://linkerd.io/2.15/getting-started/)

`curl --proto '=https' --tlsv1.2 -sSfL https://run.linkerd.io/install-edge | sh`

`export PATH=$HOME/.linkerd2/bin:$PATH`

`linkerd version`


## Steps2: Create a shared trust anchor [References](https://linkerd.io/2.15/tasks/multicluster/#install-linkerd)

`step certificate create root.linkerd.cluster.local root.crt root.key --profile root-ca --no-password --insecure`

```bash
step certificate create identity.linkerd.cluster.local issuer.crt issuer.key \
  --profile intermediate-ca --not-after 8760h --no-password --insecure \
  --ca root.crt --ca-key root.key
```

## Steps 3: With a valid trust anchor and issuer credentials, Install Linkerd on your west & east clusters

### Install the Linkerd CRDs in both clusters

```bash
# first, install the Linkerd CRDs in both clusters
linkerd install --crds \
  | tee \
    >(kubectl --context=west apply -f -) \
    >(kubectl --context=east apply -f -)

# then install the Linkerd control plane in both clusters
linkerd install \
  --identity-trust-anchors-file root.crt \
  --identity-issuer-certificate-file issuer.crt \
  --identity-issuer-key-file issuer.key \
  | tee \
    >(kubectl --context=west apply -f -) \
    >(kubectl --context=east apply -f -)

```

### And then Linkerd Viz

```bash
for ctx in west east; do
  linkerd --context=${ctx} viz install | \
    kubectl --context=${ctx} apply -f - || break
done
```

Verify that everything has come up successfully with check
```bash
for ctx in west east; do
  echo "Checking cluster: ${ctx} ........."
  linkerd --context=${ctx} check || break
  echo "-------------"
done
```

## Preparing your cluster
### Install the multicluster components on both west and east

```bash
for ctx in west east; do
  echo "Installing on cluster: ${ctx} ........."
  linkerd --context=${ctx} multicluster install | \
    kubectl --context=${ctx} apply -f - || break
  echo "-------------"
done
```
### Make sure the gateway comes up successfully

```bash
for ctx in west east; do
  echo "Checking gateway on cluster: ${ctx} ........."
  kubectl --context=${ctx} -n linkerd-multicluster \
    rollout status deploy/linkerd-gateway || break
  echo "-------------"
done
```

### Double check that the load balancer was able to allocate a public IP address

```bash
for ctx in west east; do
  printf "Checking cluster: ${ctx} ........."
  while [ "$(kubectl --context=${ctx} -n linkerd-multicluster get service -o 'custom-columns=:.status.loadBalancer.ingress[0].ip' --no-headers)" = "<none>" ]; do
      printf '.'
      sleep 1
  done
  printf "\n"
done
```

## Linking the clusters

### To link the west cluster to the east one

```bash
linkerd --context=east multicluster link --cluster-name east |
  kubectl --context=west apply -f -
```

`linkerd --context=west multicluster check`

Additionally, the east gateway should now show up in the list.\
`linkerd --context=west multicluster gateways`


## Installing the Application & services both clusters

### First Create same Namespace in Both Cluster

### Apply Application & Services

### Exporting the services Est to West Cluster
Export the `webapp-service` service in the east cluster to west cluster, You can do this by adding the mirror.linkerd.io/exported label.

`kubectl --context=east label -n service-mesh-ns svc webapp-service mirror.linkerd.io/exported=true`

Check out the service that was just created by the service mirror controller!.\
`kubectl --context=west -n service-mesh-ns get svc webapp-service-east`


Check the endpoints in west and verify that they match the gateway’s public IP address in east.\
`kubectl --context=west -n service-mesh-ns get endpoints webapp-service-east -o 'custom-columns=ENDPOINT_IP:.subsets[*].addresses[*].ip'`\
`kubectl --context=east -n linkerd-multicluster get svc linkerd-gateway -o "custom-columns=GATEWAY_IP:.status.loadBalancer.ingress[*].ip"`

**Before install Ingress in west cluster create a namespace Imperative way**
`kubectl create namespace service-mesh-ns`

Switch to `service-mesh-ns` namespace
`kubens service-mesh-ns`

Now Install Ingress via helm.
```
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update
helm install any-name ingress-nginx/ingress-nginx
```

Now Apply `namespace.yaml` that enable `linkerd.io/inject`.\
`k apply -f namespace.yaml`

Apply your other applications

Update A Record your Domain

Install Cert Managet

Install TLS Issuer

Install TLS Certificate.


**Check East cluster pod container logs**

`kubectl logs webapp-deployment-7bc5bc6468-nmlfw -c webapp-container -f`



[Reference](https://linkerd.io/2.15/tasks/multicluster/)


[Reference2](https://smallstep.com/docs/step-cli/installation/#macos)


[Reference3](https://buoyant.io/blog/multi-cluster-multi-region-setup-using-linkerd-service-mesh)

  
  



