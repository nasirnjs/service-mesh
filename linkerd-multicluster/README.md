
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


## Installing the test services both clusters

### Create Namespace

### Apply Services

### Exporting the services Est to West Cluster
Export the `webapp-service` service in the east cluster, You can do this by adding the mirror.linkerd.io/exported label.

`kubectl --context=east label svc webapp-service mirror.linkerd.io/exported=true`

Check out the service that was just created by the service mirror controller!

`kubectl --context=west get svc webapp-service-east`


`kubectl --context=west get endpoints webapp-service-east -o 'custom-columns=ENDPOINT_IP:.subsets[*].addresses[*].ip'`

`kubectl --context=east -n linkerd-multicluster get svc linkerd-gateway -o "custom-columns=GATEWAY_IP:.status.loadBalancer.ingress[*].ip"`











[Reference](https://linkerd.io/2.15/tasks/multicluster/)


[References2](https://smallstep.com/docs/step-cli/installation/#macos)




for ctx in east; do
  echo "Installing on cluster: ${ctx} ........."
  linkerd --context=${ctx} multicluster install | \
    kubectl --context=${ctx} apply -f - || break
  echo "-------------"
done

for ctx in east; do
  echo "Checking gateway on cluster: ${ctx} ........."
  kubectl --context=${ctx} -n linkerd-multicluster \
    rollout status deploy/linkerd-gateway || break
  echo "-------------"
done

for ctx in east; do
  printf "Checking cluster: ${ctx} ........."
  while [ "$(kubectl --context=${ctx} -n linkerd-multicluster get service -o 'custom-columns=:.status.loadBalancer.ingress[0].ip' --no-headers)" = "<none>" ]; do
      printf '.'
      sleep 1
  done
  printf "\n"
done



for ctx in west; do
  echo "Installing on cluster: ${ctx} ........."
  linkerd --context=${ctx} multicluster install | \
    kubectl --context=${ctx} apply -f - || break
  echo "-------------"
done

for ctx in west; do
  echo "Checking gateway on cluster: ${ctx} ........."
  kubectl --context=${ctx} -n linkerd-multicluster \
    rollout status deploy/linkerd-gateway || break
  echo "-------------"
done

for ctx in west; do
  printf "Checking cluster: ${ctx} ........."
  while [ "$(kubectl --context=${ctx} -n linkerd-multicluster get service -o 'custom-columns=:.status.loadBalancer.ingress[0].ip' --no-headers)" = "<none>" ]; do
      printf '.'
      sleep 1
  done
  printf "\n"
done


linkerd --context=east multicluster link --cluster-name east 

kubectl --context=west apply -f 




East-Cluster
kubectl --context=east label svc east-webapp-service mirror.linkerd.io/exported=true



kubectl --context=west get svc east-webapp-service-east



kubectl --context=west get endpoints east-webapp-service-east \
  -o 'custom-columns=ENDPOINT_IP:.subsets[*].addresses[*].ip'
  

kubectl --context=east -n linkerd-multicluster get svc linkerd-gateway \
  -o "custom-columns=GATEWAY_IP:.status.loadBalancer.ingress[*].ip"
  
  
  
kubectl --context=west exec -c nginx -it \
  $(kubectl --context=west get po -l app=frontend \
    --no-headers -o custom-columns=:.metadata.name) \
  -- /bin/sh -c "apk add curl && curl http://podinfo-east:9898"



