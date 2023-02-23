# kubernetes manual expansion


# k8s manual expansion

We find k8s-master node.Input the Command：

1. expand

```sh
kubectl scale --replicas=3 deploy my-test-deploy
```

2. shrink

```sh
kubectl scale --replicas=1 deploy my-test-deploy
```

## trouble cleaning

1. get resource list

```sh
kubectl get deployment
kubectl get pods
kubectl get nodes
# exists in the namespace
kubectl api-resources --namespaced=true
# not exists in the namespace
kubectl api-resources --namespaced=false
```

2. show info

```sh
kubectl describe pod my-test-pod
kubectl describe deployment my-test-pod

```

3. exec container

```sh
kubectl exec -ti my-test-pod /bin/bash
```

