


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
  
  
  
kubectl --context=west -n test exec -c nginx -it \
  $(kubectl --context=west -n test get po -l app=frontend \
    --no-headers -o custom-columns=:.metadata.name) \
  -- /bin/sh -c "apk add curl && curl http://podinfo-east:9898"



