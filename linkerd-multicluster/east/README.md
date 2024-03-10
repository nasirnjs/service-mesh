


## Steps 1: Install linkerd CLI Same version East & West both cluster.

`curl --proto '=https' --tlsv1.2 -sSfL https://run.linkerd.io/install-edge | sh`

`export PATH=$HOME/.linkerd2/bin:$PATH`

`linkerd version`

## Step 2: Validate your Kubernetes cluster both cluster

`linkerd check --pre`

## Step 3: Install Linkerd into both cluster

`linkerd install --crds | kubectl apply -f -`

`linkerd install | kubectl apply -f -`

## Step 4: Install your applications both cluster

`kubectl apply -f adding-service.yaml`

`kubectl get deployments.apps nginx-deployment -o yaml | linkerd inject - | kubectl apply -f -`

## Step 5: Explore Linkerd!

`linkerd viz install | kubectl apply -f -`

`linkerd viz dashboard &`

