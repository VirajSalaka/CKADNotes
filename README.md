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

`docker run --name ubuntu-sleeper ubuntu-sleeper 10`

Here Entrypoint is included in the dockerfile. The argument is 10. Entrypoint is ["sleep"] and the CMD is overriden by 10. 

When we convert it to kubernetes, we have to use command and args. 

```
apiVersion: v1
kind: Pod
metadata:
  name: ubuntu-sleeper
spec:
    containers:
    - name: ubuntu-sleeper
      image: ubuntu-sleeper
      args: ["10"]
```

To override entrypoint, we use `command` in the pod definition. 

### Environment Variables

under containers
```
env:
- name: APP_COLOR
  value: pink
```

### ConfigMap

#### Imperative

- kubectl create configmap app-config --from-literal=APP_COLOR=blue --from-literal=APP_MOD=prod
- kubectl create configmap app-config --from-file=app_config.properties

#### Declarative

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
    APP_COLOR: blue
    APP_MOD: prod
```

Injecting configmap as environment variable
```
apiVersion: v1
kind: Pod
metadata:
  name: ubuntu-sleeper
spec:
    containers:
    - name: ubuntu-sleeper
      image: ubuntu-sleeper
      envFrom:
      - configMapRef:
          name: app-config
```

To inject single environment variable
```
      env:
      - name: APP_COLOR
        valueFrom:
          configMapKeyRef:
            name: app-config
            key: APP_COLOR
```

We can even inject as a volume.

### Secrets

#### Imperative

- `kubectl create secret generic app-secret --from-literal=DB_PASS=password`

From file option is same as in config map

#### Declarative

```
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
data:
    DB_PASS: cGFzc3dvcmQ=
```

Data is base64 encoded.

#### Injecting as environment variable

```
apiVersion: v1
kind: Pod
metadata:
  name: ubuntu-sleeper
spec:
    containers:
    - name: ubuntu-sleeper
      image: ubuntu-sleeper
      envFrom:
      - secretRef:
          name: app-secret
```
SecretRef is used instead of configMapRef

SecretKeyFrom is used to inject single environment variable

```
      env:
      - name: DB_PASS
        valueFrom:
          secretKeyRef:
            name: app-secret
            key: DB_PASS
```

#### Injecting as volume

```
apiVersion: v1
kind: Pod
metadata:
  name: ubuntu-sleeper
spec:
    containers:
    - name: ubuntu-sleeper
      image: ubuntu-sleeper
      volumeMounts:
      - name: app-secret-volume
        mountPath: /etc/app-secret
        readOnly: true
    volumes:
    - name: app-secret-volume
      secret:
        secretName: app-secret
```
 Enabling encryption at rest for secrets is a separate topic. https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/

 ### Security Context
```
apiVersion: v1
kind: Pod
metadata:
  name: ubuntu-sleeper
spec:
    securityContext:
      runAsUser: 1000
      runAsGroup: 3000
    containers:
    - name: ubuntu-sleeper
      image: ubuntu-sleeper
      securityContext:
          runAsUser: 1010
          runAsGroup: 3000
          capabilities:
            add: ["MAC_ADMIN"]
```

### Service Account

Authentication, Authorization, RBAC

User Accounts : admin, Developer
Service Accounts : prometheus, jenkins

`kubectl create serviceaccount dashboard-sa`

when k8s service account is created, it will create a token at the same name. It is stored as a secret.

This token can be used as Authorization Bearer token.







