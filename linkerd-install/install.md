
**steps1.** Download and run the Linkerd installer by copying and pasting this command into the terminal tab.\
`curl --proto '=https' --tlsv1.2 -sSfL https://run.linkerd.io/install | LINKERD2_VERSION=stable-2.12.3 sh`

2. Make sure that the Kubernetes cluster you're using can actually run Linkerd.\
`linkerd check --pre`

3. After verifying that your cluster can run Linkerd, it's time to install the Linkerd CRDs using the linkerd CLI and kubectl. The command is simple.\
`linkerd install --crds | kubectl apply -f -`

4. Now that you've installed the Linkerd CRDs, the next step is installing Linkerd itself.\
`linkerd install | kubectl apply -f -`

5.  Be sure everything has started up completely is to run `linkerd check`.\
`linkerd check`

6. Linkerd is installed into the linkerd namespace. Take a look at the running pods in that namespace.\
`kubectl get pods -n linkerd`

These are the three main Linkerd workloads:\
`linkerd-identity manages` identity and certificates for Linkerd, and serves as Linkerd's certification authority.\
`linkerd-destination` manages request routing and authorization for Linkerd.\
`linkerd-proxy-injector` is the mutating admission controller that adds the Linkerd sidecar proxy to workloads that are part of the mesh.

7. visualization extension, `Viz` Linkerd Viz makes it easy to see deep into a running application to better understand what's working, what's failing, how long things are taking, etc.\
`linkerd viz install | kubectl apply -f -`

8.  Be sure everything has started up completely is to run `linkerd check`\
`linkerd check`\
`linkerd viz check`

Access Linkerd UI dashboard
`linkerd viz dashboard &`

9. Linkerd is installed into the `linkerd-viz` namespace. Take a look at the running pods in that namespace.\
`kubectl get pods -n linkerd-viz`



## Adding your services to Linkerd

`kubectl apply -f adding-service.yaml`

`kubectl get deployments.apps nginx-deployment -o yaml | linkerd inject - | kubectl apply -f -`