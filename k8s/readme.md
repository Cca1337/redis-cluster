# Redis on Kubernetes

Create a cluster 

```
minikube start
```
## Namespace

```
kubectl create ns redis
```

## Storage Class

```
kubectl get storageclass
NAME                 PROVISIONER             RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
standard (default)   rancher.io/local-path   Delete          WaitForFirstConsumer   false                  84s
```

## Deployment: Redis nodes

```
cd /redis/
kubectl apply -n redis -f ./redis-configmap.yaml
kubectl apply -n redis -f ./redis-statefulset.yaml

kubectl -n redis get pods
kubectl -n redis get pv

kubectl -n redis logs redis-0
kubectl -n redis logs redis-1
kubectl -n redis logs redis-2
```

## Test replication status

```
kubectl -n redis exec -it redis-0 sh
redis-cli 
auth a-very-complex-password-here
info replication
```

## Deployment: Redis Sentinel (min 3 instances)
if master dies sentinel will promote a new master

```
cd /sentinel/
kubectl apply -n redis -f ./sentinel-statefulset.yaml

kubectl -n redis get pods
kubectl -n redis get pv
kubectl -n redis logs sentinel-0
```