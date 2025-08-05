## Command
```shell
kubectl cluster-info

kubectl config view

kubectl config current-context
```

## API access
### using kubectl proxy
```shell
kubectl proxy --port=8080 &

curl localhost:8080/apis/apps/v1/deployments
```

### using kubectl
```shell
kubectl get --raw /api/v1/namespaces/default/pods

kubectl get deployments -v 6
```

### using curl
```shell
KUBE_API=$(kubectl config view -o jsonpath='{.clusters[0].cluster.server}')

curl --cacert ~/.minikube/ca.crt $KUBE_API/version

curl $KUBE_API/apis/apps/v1/deployments \                
  --cacert ~/.minikube/ca.crt \
  --cert ~/.minikube/profiles/minikube/client.crt \
  --key ~/.minikube/profiles/minikube/client.key
  
# inside pod
curl https://${KUBERNETES_SERVICE_HOST}:${KUBERNETES_SERVICE_PORT}/apis/apps/v1 \
  --cacert /var/run/secrets/kubernetes.io/serviceaccount/ca.crt \
  --header "Authorization: Bearer $(cat /var/run/secrets/kubernetes.io/serviceaccount/token)"
```