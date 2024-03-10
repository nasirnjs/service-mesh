
## CHAPTER 1: MEET THE SERVICE MESH


## CHAPTER 2: DEPLOYING LINKERD ON A KUBERNETES CLUSTER

1. Download and run the Linkerd installer by copying and pasting this command into the terminal tab.\
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

**Exam!!!!**

**Q1.** `viz` is an example of what type of Linkerd component?.\
- A data plane proxy
- A core control plane component
- A data plane plugin
- **An extension**

**Q2.** What is the first step of deploying a service mesh to touch the cluster?.\
- **Deploying the control plane**
- Deploying the data plane
- Deploying the CLI
- Deploying an extension

**Q3.** Which service mesh component runs on your local machine, not on the cluster?.\
- The control plane
- The data plane
- **The CLI**
- The application

**Q4.** Which of the following is enabled in Linkerd's HA mode?.\
- **Multiple replicas of critical control plane components**
- Multiple TLS keys for improved security
- Horizontal pod autoscaling of proxies
- All of the above

**Q5.** Where are Linkerd's control plane extensions installed?.\
- Alongside the control plane, in the same namespace
- **Alongside the control plane, in a different namespace**
- Alongside the data plane
- Alongside the CLI, on your local machine

**Q6.** What would we do differently from demo usage if we were deploying Linkerd in a production cluster?.\
- We would use a tool like Helm instead of the CLI.
- We would enable high availability mode.
- We would generate our own key material.
- **All of the above**

## Chapter 3: Meshing your application

1. Let's use method 1 to install, and mesh, the world-famous emojivoto application. Start by running the following in the terminal tab.\
`curl --proto '=https' --tlsv1.2 -sSfL https://run.linkerd.io/emojivoto.yml | kubectl apply -f -`

This will install the emojivoto application in its own namespace (also called emojivoto). Next, wait for it to become ready.\
`kubectl wait --timeout=5m -n emojivoto deployment --all --for=condition=Available`

2. Now that emojivoto is running, we need to include it in the service mesh.\
`kubectl get deployment -n emojivoto -o yaml | linkerd inject - | kubectl apply -f -`

This will restart all the emojivoto Pods, so you should wait for that to finish.\
`kubectl rollout status -n emojivoto deployments`

3. Let's use auto-injection to install the world-famous booksapp application. By default, booksapp wants to live in the `booksapp` namespace, so start by creating that `namespace`.\
`kubectl create namespace booksapp`

Once it's created, use the linkerd/inject annotation to mark it for auto-injection.\
`kubectl annotate ns booksapp linkerd.io/inject=enabled`

This will cause any Pods created in the booksapp `namespace` to automatically have the Linkerd sidecar proxy injected, allowing them to participate in the service mesh. Once that's done, continue by installing `booksapp` itself.\
`curl --proto '=https' --tlsv1.2 -sSfL https://run.linkerd.io/booksapp.yml | kubectl apply -n booksapp -f -`

This will install the booksapp application into the booksapp namespace. Next, wait for it to become ready.\
`kubectl wait --timeout=5m -n booksapp deployment --all --for=condition=Available`

At this point, booksapp will be up and running, and you'll be able to see that each Pod has two running containers.\
`kubectl get pods -n booksapp`

**Questions!!!!**
**Q1.** What happens when a workload is meshed?\
- Service mesh sidecar containers are injected into its application code.
- **Service mesh sidecar containers are added to its pods.**
- Service mesh sidecar containers are injected into namespace annotations.
- Service mesh sidecar containers are added to the proxy injector.

**Q2.** After the injection annotation is applied to a namespace, what must be done to mesh all workloads in that namespace?.\
- **Roll any existing workloads.**
- Roll all new workloads.
- Restart the proxy injector.
- All of the above.

**Q3.** Why does a service mesh use init containers?.\
- To initialize the sidecar proxies.
- To initialize the application.
- To set up networking so that TCP traffic to and from a pod is routed to the application.
- **To set up networking so that TCP traffic to and from a pod is routed to the sidecar proxy.**

## CHAPTER 4: GETTING GOLDEN METRICS FOR YOUR APPLICATION

1. Linkerd to provide statistical information about the services mesh within a specific Kubernetes namespace.\
`linkerd viz stat namespace`

2. Look at metrics for individual Deployments.\
`linkerd viz stat deployment -n emojivoto`

3. We can get metrics for Services as well.\
`linkerd viz stat service -n emojivoto`

4. As a quick recap, when we pulled namespace stats
`linkerd viz stat namespace`

5. Which will show the most common requests being made this deploymen.\
`linkerd viz top -n emojivoto deployment/voting`

**Questions!!!!**

**Q1.** What are some downsides of Linkerd Viz?\
- It requires additional components running in your environment.
- It has no built-in access control or user login.
- Prometheus may require significant resources.
- **All of the above**

**Q2.** What metrics functionality does the Linkerd proxy provide?\
- **Instrumentation.**
- Aggregation.
- Reporting.
- All of the above.

**Q3.** What are the three golden metrics provided by a service mesh?\
- Latency, performance, and reporting
- **Latency, success rate, and traffic volume**
- Instrumentation, aggregation, and reporting
- Instrumentation, aggregation, and traffic volume

**Q4.** What does the p95 latency represent?\
- The latency of 95 percent of requests.
- **The latency at which 95 percent of requests were at or under.**
- The latency at which 95 percent of requests were over.
- The latency at which 5 percent of requests were at or under.

**Q5.** What is unique about latency as a metric?\
- **It doesn't have a fixed set of possible values.**
- It is either true or false.
- It represents a count rather than a value.
- All of the above.

## CHAPTER 5: INTRODUCTION TO MUTUAL TLS

**mTLS in Linkerd**
Linkerd automatically secures communications between workloads using mutual TLS (mTLS).\
Linkerd automatically protects all communications in the cluster using mTLS. We'll take a quick look at how to verify working mTLS in the cluster.\

**Q1.**Which property of a TLS connection means that it is encrypted?
- Authenticity
- **Confidentiality**
- Integrity
- All of the above

**Q2.**In the context of TLS, why is authenticity important?
- It prevents eavesdroppers from getting access to the data.
- It ensures that the message is received without modification.
- It allows the certificate authority to rotate the TLS keys.
- **It ensures that the other identity is who is says it is.**

**Q3.**What of the following is true?
- IP addresses are an example of network identity.
- Workload identity gives you a better security posture than network identity.
- Relying on network identity requires that you trust the network.
- **All of the above.**

**Q3.**Why do we issue short-lived TLS certificates?
- **To reduce the impact of certificate leaks.**
- To reduce load on the CA.
- To speed up pod creation time.
- To provide workload identity that spans the cluster boundary.


## Chapter 6: Traffic reliability
Every distributed system must deal with the concept of failure: components can die, networks can “partition” (lose connectivity between nodes), and so on. A reliable application is one that can successfully serve responses even in the presence of these failures.

While Kubernetes takes care of some of these failure cases for you—for example, it can restart pods if they crash, and reschedule them entirely if a node crashes—Kubernetes can only do so much.

A service mesh provides several mechanisms that can further enhance the reliability of your application beyond what Kubernetes itself provides. These features include:

**Request retries:** Automatically retry requests that have failed for a particular request, potentially sending them to a different pod.\
**Request timeouts:** Cut off individual requests that are taking too long instead of waiting indefinitely for the service to respond.\
**Circuit breaking:** Cut off all requests to a service that is returning errors in order to give it a chance to recover.\
**Intelligent load balancing:** Send requests to the best pod to handle them, e.g. the fastest pod or the one processing the fewest requests.\
**Progressive delivery:** When a new version of a service comes online, gradually send it traffic to assess whether it is functioning as expected.\
**Failover:** If a service in one cluster fails, automatically send traffic to the same service in another cluster.


## CHAPTER 7: NEXT STEPS ON YOUR SERVICE MESH JOURNEY

1. scale the faces workload to two Pods
`kubectl scale -n faces deploy/face --replicas=2`

2. Check  existing ServiceProfiles.\
`kubectl get serviceprofile -n faces smiley.faces.svc.cluster.local -o yaml`

3. Viz dashboard by selecting the faces namespace, picking the smiley Deployment, then clicking Route Metrics rather than Live Calls on the deployment/smiley screen.\
`linkerd viz -n faces routes svc/smiley`

4. To enable retries for the `smiley` service, we need only add isRetryable to the route we've defined in our ServiceProfile.
```bash
kubectl apply -f - <<EOF
---
apiVersion: linkerd.io/v1alpha2
kind: ServiceProfile
metadata:
  creationTimestamp: null
  name: smiley.faces.svc.cluster.local
  namespace: faces
spec:
  routes:
  - condition:
      method: GET
      pathRegex: /
    name: GET /
    isRetryable: true
EOF
```
5. Enabling Retries for color, To get rid of the grey backgrounds, we'll do exactly the same thing for the color workload.
```bash
kubectl apply -f - <<EOF
---
apiVersion: linkerd.io/v1alpha2
kind: ServiceProfile
metadata:
  creationTimestamp: null
  name: color.faces.svc.cluster.local
  namespace: faces
spec:
  routes:
  - condition:
      method: GET
      pathRegex: /
    name: GET /
    isRetryable: true
EOF
```
6. Enforcing Timeouts for smiley. To enable timeouts for the smiley workload, we just need to add a timeout definition to its ServiceProfile.
```
kubectl apply -f - <<EOF
---
apiVersion: linkerd.io/v1alpha2
kind: ServiceProfile
metadata:
  creationTimestamp: null
  name: smiley.faces.svc.cluster.local
  namespace: faces
spec:
  routes:
  - condition:
      method: GET
      pathRegex: /
    name: GET /
    isRetryable: true
    timeout: 300ms
EOF
```

7. Enforcing Timeouts for color, We'll repeat exactly the same thing for the color workload.
```
kubectl apply -f - <<EOF
---
apiVersion: linkerd.io/v1alpha2
kind: ServiceProfile
metadata:
  creationTimestamp: null
  name: color.faces.svc.cluster.local
  namespace: faces
spec:
  routes:
  - condition:
      method: GET
      pathRegex: /
    name: GET /
    isRetryable: true
    timeout: 300ms
EOF
```

**Q1.**One common use case for the service mesh doing load balancing in Kubernetes is...
- **Handling gRPC requests**
- Failing over between clusters
- Measuring the latency of individual pods
- Progressive delivery of applications

**Q2.**Which reliability feature addresses the fact that individual requests can fail?
**Retries**
- Failover
- Timeouts
- Progressive delivery

**Q3.**Why is it important to configure retries on a per-route basis?
-To prevent system overload
- **Because not all requests are safe to retry**
- To better load balance gRPC requests
- Because Kubernetes can only handle L4, not L7.

**Q4.**What does progressive delivery protect against?
- The risk of system overload from retries
- Failover to a different cluster
- Partial failures due to network partitions
- **The risk of rolling out a bad deploy**

