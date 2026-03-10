# Kubernetes Notes

## Core Concepts

### Pod
The smallest deployable unit in K8s. A Pod runs one or more containers sharing network + storage.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
  labels:
    app: web
spec:
  containers:
  - name: nginx
    image: nginx:alpine
    ports:
    - containerPort: 80
    resources:
      requests:
        cpu: "100m"
        memory: "64Mi"
      limits:
        cpu: "250m"
        memory: "128Mi"
    livenessProbe:
      httpGet:
        path: /healthz
        port: 80
      initialDelaySeconds: 3
      periodSeconds: 10
```

### Deployment
Manages ReplicaSets to ensure desired number of Pods are running.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
      - name: web
        image: myapp:v1.2
        ports:
        - containerPort: 8080
        env:
        - name: ENV
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: environment
```

### Service
Exposes Pods via a stable network endpoint.

| Type | Use case |
|---|---|
| ClusterIP | Internal cluster access (default) |
| NodePort | External access via node IP:port |
| LoadBalancer | Cloud LB (AWS ELB, etc.) |

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-svc
spec:
  selector:
    app: web
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
  type: ClusterIP
```

## Useful kubectl Commands

```bash
# Cluster info
kubectl cluster-info
kubectl get nodes -o wide

# Pods
kubectl get pods -A                          # all namespaces
kubectl describe pod <name>
kubectl logs <pod> -c <container> -f         # follow logs
kubectl exec -it <pod> -- /bin/bash
kubectl top pods --sort-by=cpu

# Deployments
kubectl rollout status deploy/<name>
kubectl rollout history deploy/<name>
kubectl rollout undo deploy/<name>           # rollback
kubectl scale deploy/<name> --replicas=5

# Debugging
kubectl get events --sort-by='.lastTimestamp'
kubectl debug <pod> --image=busybox --copy-to=debug-pod
```

## Helm Basics

```bash
# Install a chart
helm install my-release bitnami/nginx

# With custom values
helm install my-release bitnami/nginx -f values-prod.yaml

# Upgrade
helm upgrade my-release bitnami/nginx --set replicaCount=3

# Rollback
helm rollback my-release 1

# Template rendering (debug)
helm template my-release ./my-chart --debug
```

## Probes

```yaml
livenessProbe:   # restart if unhealthy
  httpGet:
    path: /health
    port: 8080
  initialDelaySeconds: 15
  periodSeconds: 20

readinessProbe:  # remove from LB if not ready
  httpGet:
    path: /ready
    port: 8080
  initialDelaySeconds: 5
  periodSeconds: 10

startupProbe:    # for slow-starting containers
  httpGet:
    path: /health
    port: 8080
  failureThreshold: 30
  periodSeconds: 10
```
