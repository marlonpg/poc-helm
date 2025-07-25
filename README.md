# poc-helm

## Requirments
- Docker installed and running
- Minikube running
- kubectl working
- Helm installed

## Setup
### Create a Project
```bash
helm create my-first-chart
```

### Install it into your Minikube cluster:
```bash
helm install my-nginx ./my-first-chart --namespace default
```

### Check that the pods and service were created:
```bash
kubectl get pods
## Output: 
## $ kubectl get pods
## NAME                                       READY   STATUS    RESTARTS   AGE
## my-nginx-my-first-chart-7f94d65b4d-qs7zr   1/1     Running   0          59s
```

### Check that the pods and service were created:
```bash
kubectl get svc
## Output: 
## $ kubectl get svc
## NAME                      TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
## kubernetes                ClusterIP   10.96.0.1    <none>        443/TCP   14d
## my-nginx-my-first-chart   ClusterIP   10.98.38.7   <none>        80/TCP    106s
```

### Made a change to replicaCount: 2
```bash
# Apply the changes, upgrade the release
helm upgrade my-nginx ./my-first-chart

# Check the changes
kubectl get pods
## Output: 
## $ kubectl get pods
## NAME                                       READY   STATUS    RESTARTS   AGE
## my-nginx-my-first-chart-7f94d65b4d-6cg6x   1/1     Running   0          7s
## my-nginx-my-first-chart-7f94d65b4d-qs7zr   1/1     Running   0          2m31s
```

### Uninstall Helm chart
```bash
helm uninstall my-nginx
```

