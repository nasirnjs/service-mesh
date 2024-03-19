
**Before install Ingress in west cluster create a namespace Imperative way**
`k create namespace service-mesh-ns`

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

