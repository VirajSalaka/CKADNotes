# CKADNotes


## Namespaces

### Resource Quota for a namespace

```
apiVersion: v1
kind: ResourceQuota
metadata:
  name: compute-resources
  namespace: dev
spec:
    hard:
        pods: "10"
        requests.cpu: "4"
        requests.memory: 5Gi
        limits.cpu: "10"
        limits.memory: 10Gi
```

### Imperative Commands

#### --dry-run=client
- kubectl run nginx --image=nginx --dry-run=client -o yaml > nginx-pod.yaml

#### pod
- kubectl run nginx --image=nginx

#### deployment with replicas
- kubectl create deployment --image=nginx nginx --replicas=4
- kubectl scale deployment nginx --replicas=4

#### service
- kubectl expose pod redis --port=6379 --name redis-service --dry-run=client -o yaml
    this will automatically assign pods label as selector
- kubectl expose pod nginx --port=80 --name nginx-service --type=NodePort --dry-run=client -o yaml
     Here you cannot mention the nodeport still, hence you have to edit and apply later
- kubectl create service clusterip redis --tcp=6379:6379 --dry-run=client -o yaml
     Here it will assume selectors as `app=redis`


## Configuration

### Docker

docker run -it --entrypoint /bin/bash nginx


