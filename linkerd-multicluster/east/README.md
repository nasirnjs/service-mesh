
##  Steps 1: Start cluster and get right contexts

`linkerd  kubectl config get-contexts`

`linkerd  kubectl config use-context east`

## Steps 2: Install linkerd CLI both cluster
If this is your first time running Linkerd, you will need to download the linkerd CLI onto your local machine. The CLI will allow you to interact with your Linkerd deployment

### Steps 2.1: Install linkerd CLI Same version East & West both cluster.

`curl --proto '=https' --tlsv1.2 -sSfL https://run.linkerd.io/install-edge | sh`

`export PATH=$HOME/.linkerd2/bin:$PATH`

`linkerd version`

### Step 2.2: Validate your Kubernetes cluster both cluster

`linkerd check --pre`

### Step 2.3: Install Linkerd into both cluster

`linkerd install --crds | kubectl apply -f -`

`linkerd install | kubectl apply -f -`

### Step 2.4: Install your applications both cluster

`kubectl apply -f adding-service.yaml`

`kubectl get deployments.apps webapp-deployment -o yaml | linkerd inject - | kubectl apply -f -`

### Step 2.5: Explore Linkerd!

`linkerd viz install | kubectl apply -f -`

`linkerd viz dashboard &`

