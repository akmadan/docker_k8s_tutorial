# Kubernetes Tutorial - Complete Reference Guide

## Table of Contents
1. [Introduction to Kubernetes](#introduction-to-kubernetes)
2. [Kubernetes Architecture](#kubernetes-architecture)
3. [Installation & Setup](#installation--setup)
4. [Kubernetes Objects](#kubernetes-objects)
5. [Pods](#pods)
6. [Deployments](#deployments)
7. [Services](#services)
8. [ConfigMaps and Secrets](#configmaps-and-secrets)
9. [Namespaces](#namespaces)
10. [ReplicaSets](#replicasets)
11. [StatefulSets](#statefulsets)
12. [DaemonSets](#daemonsets)
13. [Jobs and CronJobs](#jobs-and-cronjobs)
14. [Ingress](#ingress)
15. [Volumes and Persistent Volumes](#volumes-and-persistent-volumes)
16. [Resource Management](#resource-management)
17. [Health Checks](#health-checks)
18. [Scaling](#scaling)
19. [Rolling Updates and Rollbacks](#rolling-updates-and-rollbacks)
20. [Networking](#networking)
21. [Service Discovery](#service-discovery)
22. [Troubleshooting](#troubleshooting)
23. [Best Practices](#best-practices)

---

## Introduction to Kubernetes

### What is Kubernetes?
- **Container orchestration platform** for automating deployment, scaling, and management
- Originally developed by Google (based on Borg)
- Open-source project maintained by CNCF
- Also known as **K8s** (K-eight-S)

### Why Kubernetes?
- **Automated scaling**: Scale applications up/down automatically
- **Self-healing**: Restart failed containers, replace nodes
- **Service discovery**: Automatic load balancing
- **Rolling updates**: Zero-downtime deployments
- **Resource management**: CPU and memory allocation
- **Portability**: Run on any cloud or on-premises

### Key Concepts
- **Cluster**: Set of nodes (machines) running Kubernetes
- **Node**: Worker machine (VM or physical)
- **Pod**: Smallest deployable unit (one or more containers)
- **Service**: Stable network endpoint for pods
- **Namespace**: Virtual cluster within a cluster

### Kubernetes vs Docker
- **Docker**: Container runtime and platform
- **Kubernetes**: Orchestrates containers (can use Docker, containerd, etc.)
- Kubernetes manages Docker containers at scale

---

## Kubernetes Architecture

### Master Node (Control Plane)
Components:
- **API Server**: Front-end for Kubernetes control plane
- **etcd**: Distributed key-value store (cluster state)
- **Scheduler**: Assigns pods to nodes
- **Controller Manager**: Runs controllers (replication, endpoints, etc.)

### Worker Nodes
Components:
- **kubelet**: Agent that communicates with master
- **kube-proxy**: Network proxy maintaining network rules
- **Container Runtime**: Docker, containerd, CRI-O, etc.

### How It Works
1. User submits desired state via `kubectl` or API
2. API Server validates and stores in etcd
3. Scheduler assigns pods to nodes
4. kubelet on nodes creates containers
5. Controllers ensure desired state is maintained

---

## Installation & Setup

### kubectl Installation
```bash
# macOS
brew install kubectl

# Linux
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin/

# Verify
kubectl version --client
```

### Local Kubernetes Options
- **Minikube**: Single-node local cluster
- **Docker Desktop**: Built-in Kubernetes
- **kind**: Kubernetes in Docker
- **k3d**: Lightweight Kubernetes in Docker

### Minikube Setup
```bash
# Install Minikube
brew install minikube  # macOS
# or download from minikube.sigs.k8s.io

# Start cluster
minikube start
minikube start --driver=docker

# Check status
minikube status

# Stop cluster
minikube stop

# Delete cluster
minikube delete

# Access dashboard
minikube dashboard
```

### Verify Installation
```bash
# Check cluster connection
kubectl cluster-info

# Get nodes
kubectl get nodes

# Get all resources
kubectl get all
```

---

## Kubernetes Objects

### Object Spec and Status
- **Spec**: Desired state (what you want)
- **Status**: Current state (what exists)
- Kubernetes tries to make status match spec

### Common Objects
- Pods
- Deployments
- Services
- ConfigMaps
- Secrets
- Namespaces
- ReplicaSets
- StatefulSets
- DaemonSets
- Jobs
- CronJobs
- Ingress

### YAML Structure
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
  labels:
    app: my-app
spec:
  containers:
  - name: my-container
    image: nginx:latest
```

---

## Pods

### What is a Pod?
- **Smallest deployable unit** in Kubernetes
- Contains one or more containers
- Containers in a pod share:
  - Network namespace (same IP)
  - Storage volumes
  - IPC namespace

### Pod YAML Example
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: nginx
    tier: frontend
spec:
  containers:
  - name: nginx-container
    image: nginx:1.21
    ports:
    - containerPort: 80
    env:
    - name: ENV_VAR
      value: "value"
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
      limits:
        memory: "128Mi"
        cpu: "500m"
```

### Pod Commands
```bash
# Create pod from YAML
kubectl apply -f pod.yaml
kubectl create -f pod.yaml

# Create pod imperatively
kubectl run nginx-pod --image=nginx:1.21

# Get pods
kubectl get pods
kubectl get pods -o wide  # More details
kubectl get pods -A  # All namespaces

# Describe pod
kubectl describe pod nginx-pod

# Get pod logs
kubectl logs nginx-pod
kubectl logs -f nginx-pod  # Follow logs
kubectl logs nginx-pod -c container-name  # Multi-container pod

# Execute command in pod
kubectl exec -it nginx-pod -- bash
kubectl exec nginx-pod -- ls -la

# Delete pod
kubectl delete pod nginx-pod
kubectl delete -f pod.yaml
kubectl delete pods --all
```

### Pod Lifecycle
- **Pending**: Pod accepted, containers not created
- **Running**: Pod bound to node, containers running
- **Succeeded**: All containers terminated successfully
- **Failed**: At least one container terminated in failure
- **Unknown**: Pod state cannot be determined

---

## Deployments

### What is a Deployment?
- **Manages ReplicaSets** and provides declarative updates
- Ensures desired number of pods are running
- Handles **rolling updates** and **rollbacks**
- Most common way to deploy applications

### Deployment YAML Example
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.21
        ports:
        - containerPort: 80
        resources:
          requests:
            memory: "64Mi"
            cpu: "250m"
          limits:
            memory: "128Mi"
            cpu: "500m"
```

### Deployment Commands
```bash
# Create deployment
kubectl apply -f deployment.yaml
kubectl create deployment nginx --image=nginx:1.21

# Get deployments
kubectl get deployments
kubectl get deploy  # Short form

# Scale deployment
kubectl scale deployment nginx-deployment --replicas=5
kubectl scale --replicas=3 deployment/nginx-deployment

# Update deployment
kubectl set image deployment/nginx-deployment nginx=nginx:1.22
kubectl edit deployment nginx-deployment

# Rollout status
kubectl rollout status deployment/nginx-deployment

# Rollout history
kubectl rollout history deployment/nginx-deployment

# Rollback
kubectl rollout undo deployment/nginx-deployment
kubectl rollout undo deployment/nginx-deployment --to-revision=2

# Pause rollout
kubectl rollout pause deployment/nginx-deployment

# Resume rollout
kubectl rollout resume deployment/nginx-deployment

# Delete deployment
kubectl delete deployment nginx-deployment
```

### Deployment Strategies
- **Rolling Update** (default): Gradually replace old pods
- **Recreate**: Kill all old pods, then create new ones

```yaml
spec:
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
```

---

## Services

### What is a Service?
- **Abstract way to expose pods** as network service
- Provides stable IP address and DNS name
- Load balances traffic to pods
- Enables service discovery

### Service Types
1. **ClusterIP** (default): Internal cluster access
2. **NodePort**: Expose on each node's IP
3. **LoadBalancer**: Cloud provider load balancer
4. **ExternalName**: Maps to external DNS name

### ClusterIP Service
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  type: ClusterIP
  selector:
    app: nginx
  ports:
  - port: 80
    targetPort: 80
    protocol: TCP
```

### NodePort Service
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-nodeport
spec:
  type: NodePort
  selector:
    app: nginx
  ports:
  - port: 80
    targetPort: 80
    nodePort: 30080
    protocol: TCP
```

### LoadBalancer Service
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-lb
spec:
  type: LoadBalancer
  selector:
    app: nginx
  ports:
  - port: 80
    targetPort: 80
```

### Service Commands
```bash
# Create service
kubectl apply -f service.yaml
kubectl expose deployment nginx-deployment --port=80 --type=NodePort

# Get services
kubectl get services
kubectl get svc  # Short form

# Describe service
kubectl describe service nginx-service

# Get service endpoints
kubectl get endpoints nginx-service

# Delete service
kubectl delete service nginx-service
```

### Service Discovery
- Services get DNS name: `<service-name>.<namespace>.svc.cluster.local`
- Pods in same namespace can access via service name
- Example: `http://nginx-service:80`

---

## ConfigMaps and Secrets

### ConfigMaps

#### What is a ConfigMap?
- Store **non-confidential configuration data**
- Key-value pairs
- Can be mounted as files or environment variables

#### Create ConfigMap
```bash
# From literal values
kubectl create configmap my-config --from-literal=key1=value1 --from-literal=key2=value2

# From file
kubectl create configmap my-config --from-file=config.properties

# From YAML
kubectl apply -f configmap.yaml
```

#### ConfigMap YAML
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  database_url: "postgresql://localhost:5432/mydb"
  log_level: "info"
  config.properties: |
    server.port=8080
    server.host=0.0.0.0
```

#### Use ConfigMap in Pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
  - name: my-container
    image: nginx
    envFrom:
    - configMapRef:
        name: app-config
    # OR
    env:
    - name: DATABASE_URL
      valueFrom:
        configMapKeyRef:
          name: app-config
          key: database_url
    # OR as volume
    volumeMounts:
    - name: config-volume
      mountPath: /etc/config
  volumes:
  - name: config-volume
    configMap:
      name: app-config
```

#### ConfigMap Commands
```bash
# Get configmaps
kubectl get configmaps
kubectl get cm  # Short form

# Describe configmap
kubectl describe configmap app-config

# Edit configmap
kubectl edit configmap app-config

# Delete configmap
kubectl delete configmap app-config
```

### Secrets

#### What is a Secret?
- Store **sensitive data** (passwords, tokens, keys)
- Base64 encoded (not encrypted by default)
- Similar to ConfigMap but for sensitive data

#### Create Secret
```bash
# From literal
kubectl create secret generic my-secret --from-literal=username=admin --from-literal=password=secret123

# From file
kubectl create secret generic my-secret --from-file=./username.txt --from-file=./password.txt

# From YAML (base64 encoded)
kubectl apply -f secret.yaml
```

#### Secret YAML
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
type: Opaque
data:
  username: YWRtaW4=  # base64 encoded
  password: c2VjcmV0MTIz  # base64 encoded
```

#### Encode/Decode
```bash
# Encode
echo -n 'admin' | base64
echo -n 'secret123' | base64

# Decode
echo 'YWRtaW4=' | base64 -d
```

#### Use Secret in Pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
  - name: my-container
    image: nginx
    envFrom:
    - secretRef:
        name: app-secret
    # OR
    env:
    - name: PASSWORD
      valueFrom:
        secretKeyRef:
          name: app-secret
          key: password
    # OR as volume
    volumeMounts:
    - name: secret-volume
      mountPath: /etc/secret
      readOnly: true
  volumes:
  - name: secret-volume
    secret:
      secretName: app-secret
```

#### Secret Commands
```bash
# Get secrets
kubectl get secrets
kubectl get secret  # Short form

# Describe secret
kubectl describe secret app-secret

# View secret (decoded)
kubectl get secret app-secret -o jsonpath='{.data.password}' | base64 -d

# Delete secret
kubectl delete secret app-secret
```

---

## Namespaces

### What is a Namespace?
- **Virtual cluster** within a physical cluster
- Isolate resources
- Resource quotas and limits
- Default namespaces: `default`, `kube-system`, `kube-public`

### Namespace Commands
```bash
# Create namespace
kubectl create namespace production
kubectl create ns staging  # Short form

# Get namespaces
kubectl get namespaces
kubectl get ns

# Switch namespace context
kubectl config set-context --current --namespace=production

# Create resource in namespace
kubectl apply -f pod.yaml -n production

# Get resources in namespace
kubectl get pods -n production
kubectl get all -n production

# Delete namespace (deletes all resources)
kubectl delete namespace production
```

### Namespace YAML
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: production
  labels:
    name: production
```

---

## ReplicaSets

### What is a ReplicaSet?
- Ensures specified number of pod replicas are running
- **Deployments use ReplicaSets** internally
- Rarely created directly (use Deployments instead)

### ReplicaSet YAML
```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: nginx-replicaset
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.21
```

### ReplicaSet Commands
```bash
# Get replicasets
kubectl get replicasets
kubectl get rs

# Scale replicaset
kubectl scale replicaset nginx-replicaset --replicas=5

# Delete replicaset
kubectl delete replicaset nginx-replicaset
```

---

## StatefulSets

### What is a StatefulSet?
- Manages **stateful applications**
- Provides stable network identity
- Ordered deployment and scaling
- Stable persistent storage
- Use cases: Databases, message queues

### StatefulSet YAML
```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql-statefulset
spec:
  serviceName: mysql
  replicas: 3
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - name: mysql
        image: mysql:8.0
        env:
        - name: MYSQL_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-secret
              key: password
        volumeMounts:
        - name: mysql-storage
          mountPath: /var/lib/mysql
  volumeClaimTemplates:
  - metadata:
      name: mysql-storage
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 10Gi
```

### StatefulSet Characteristics
- Pods have stable names: `<statefulset-name>-<ordinal>`
- Ordered creation/deletion
- Stable network identity
- Persistent storage per pod

### StatefulSet Commands
```bash
# Get statefulsets
kubectl get statefulsets
kubectl get sts

# Scale statefulset
kubectl scale statefulset mysql-statefulset --replicas=5

# Delete statefulset
kubectl delete statefulset mysql-statefulset
```

---

## DaemonSets

### What is a DaemonSet?
- Ensures **all nodes run a copy** of a pod
- Use cases: Log collection, monitoring, networking
- Automatically adds pods to new nodes

### DaemonSet YAML
```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd-logging
spec:
  selector:
    matchLabels:
      name: fluentd-logging
  template:
    metadata:
      labels:
        name: fluentd-logging
    spec:
      containers:
      - name: fluentd
        image: fluent/fluentd:latest
        volumeMounts:
        - name: varlog
          mountPath: /var/log
        - name: varlibdockercontainers
          mountPath: /var/lib/docker/containers
          readOnly: true
      volumes:
      - name: varlog
        hostPath:
          path: /var/log
      - name: varlibdockercontainers
        hostPath:
          path: /var/lib/docker/containers
```

### DaemonSet Commands
```bash
# Get daemonsets
kubectl get daemonsets
kubectl get ds

# Delete daemonset
kubectl delete daemonset fluentd-logging
```

---

## Jobs and CronJobs

### Jobs

#### What is a Job?
- Runs **one or more pods** until completion
- Use cases: Batch processing, one-time tasks

#### Job YAML
```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: pi-job
spec:
  template:
    spec:
      containers:
      - name: pi
        image: perl:5.34
        command: ["perl", "-Mbignum=bpi", "-wle", "print bpi(2000)"]
      restartPolicy: Never
  backoffLimit: 4
```

#### Job Commands
```bash
# Get jobs
kubectl get jobs

# Describe job
kubectl describe job pi-job

# Get job logs
kubectl logs job/pi-job

# Delete job
kubectl delete job pi-job
```

### CronJobs

#### What is a CronJob?
- Runs **Jobs on a schedule** (cron syntax)
- Use cases: Scheduled backups, reports, cleanup

#### CronJob YAML
```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: backup-cronjob
spec:
  schedule: "0 2 * * *"  # 2 AM daily
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: backup
            image: backup-image:latest
            command: ["/bin/sh", "-c", "backup.sh"]
          restartPolicy: OnFailure
  successfulJobsHistoryLimit: 3
  failedJobsHistoryLimit: 1
```

#### Cron Syntax
```
# ┌───────────── minute (0 - 59)
# │ ┌───────────── hour (0 - 23)
# │ │ ┌───────────── day of month (1 - 31)
# │ │ │ ┌───────────── month (1 - 12)
# │ │ │ │ ┌───────────── day of week (0 - 6)
# │ │ │ │ │
# * * * * *
```

#### CronJob Commands
```bash
# Get cronjobs
kubectl get cronjobs
kubectl get cj

# Describe cronjob
kubectl describe cronjob backup-cronjob

# Suspend cronjob
kubectl patch cronjob backup-cronjob -p '{"spec":{"suspend":true}}'

# Resume cronjob
kubectl patch cronjob backup-cronjob -p '{"spec":{"suspend":false}}'

# Delete cronjob
kubectl delete cronjob backup-cronjob
```

---

## Ingress

### What is Ingress?
- **Manages external HTTP/HTTPS access** to services
- Provides load balancing, SSL termination, routing
- Requires Ingress Controller (nginx, traefik, etc.)

### Ingress YAML
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: myapp.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: nginx-service
            port:
              number: 80
  - host: api.example.com
    http:
      paths:
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: api-service
            port:
              number: 8080
```

### TLS Ingress
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: tls-ingress
spec:
  tls:
  - hosts:
    - myapp.example.com
    secretName: tls-secret
  rules:
  - host: myapp.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: nginx-service
            port:
              number: 80
```

### Ingress Commands
```bash
# Get ingress
kubectl get ingress
kubectl get ing

# Describe ingress
kubectl describe ingress app-ingress

# Delete ingress
kubectl delete ingress app-ingress
```

### Install Ingress Controller
```bash
# Nginx Ingress Controller
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.8.1/deploy/static/provider/cloud/deploy.yaml

# For Minikube
minikube addons enable ingress
```

---

## Volumes and Persistent Volumes

### Volumes

#### What is a Volume?
- **Directory accessible to containers** in a pod
- Data persists beyond container lifecycle
- Types: emptyDir, hostPath, PersistentVolumeClaim, etc.

#### Volume Types in Pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: volume-pod
spec:
  containers:
  - name: app
    image: nginx
    volumeMounts:
    - name: app-storage
      mountPath: /app/data
    - name: config-volume
      mountPath: /etc/config
  volumes:
  - name: app-storage
    emptyDir: {}  # Temporary storage
  - name: config-volume
    configMap:
      name: app-config
```

### Persistent Volumes (PV) and Persistent Volume Claims (PVC)

#### Persistent Volume
- **Cluster-wide storage resource**
- Provisioned by administrator
- Independent of pod lifecycle

#### Persistent Volume YAML
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-volume
spec:
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  hostPath:
    path: /data/pv
```

#### Persistent Volume Claim
- **Request for storage** by user
- Binds to available PV
- Used in pod via volumeMount

#### Persistent Volume Claim YAML
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-claim
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
```

#### Use PVC in Pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pvc-pod
spec:
  containers:
  - name: app
    image: nginx
    volumeMounts:
    - name: storage
      mountPath: /data
  volumes:
  - name: storage
    persistentVolumeClaim:
      claimName: pvc-claim
```

#### Storage Classes
```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast-ssd
provisioner: kubernetes.io/aws-ebs
parameters:
  type: gp3
  fsType: ext4
```

#### Dynamic Provisioning with StorageClass
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: dynamic-pvc
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: fast-ssd
  resources:
    requests:
      storage: 10Gi
```

### Volume Commands
```bash
# Get persistent volumes
kubectl get pv

# Get persistent volume claims
kubectl get pvc

# Describe PV/PVC
kubectl describe pv pv-volume
kubectl describe pvc pvc-claim

# Delete PVC (and bound PV if reclaim policy allows)
kubectl delete pvc pvc-claim
```

---

## Resource Management

### Resource Requests and Limits

#### Requests
- **Minimum resources** guaranteed to container
- Used for scheduling decisions

#### Limits
- **Maximum resources** container can use
- Prevents resource exhaustion

### Resource YAML
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: resource-pod
spec:
  containers:
  - name: app
    image: nginx
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"  # 0.25 CPU
      limits:
        memory: "128Mi"
        cpu: "500m"  # 0.5 CPU
```

### Resource Units
- **CPU**: Cores (1, 0.5) or millicores (1000m, 500m)
- **Memory**: Bytes (128974848), KiB, Mi, Gi, etc.

### Resource Quotas
```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: compute-quota
  namespace: production
spec:
  hard:
    requests.cpu: "4"
    requests.memory: 8Gi
    limits.cpu: "8"
    limits.memory: 16Gi
    persistentvolumeclaims: "4"
    pods: "10"
```

### Limit Ranges
```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: mem-limit-range
  namespace: production
spec:
  limits:
  - default:
      memory: "512Mi"
      cpu: "500m"
    defaultRequest:
      memory: "256Mi"
      cpu: "250m"
    type: Container
```

### Resource Commands
```bash
# Get resource usage
kubectl top nodes
kubectl top pods
kubectl top pod <pod-name>

# Get resource quotas
kubectl get resourcequotas
kubectl get quota

# Get limit ranges
kubectl get limitranges
kubectl get limits
```

---

## Health Checks

### Liveness Probe
- **Detects if container is running**
- Restarts container if probe fails
- Use when container can recover from deadlock

### Readiness Probe
- **Detects if container is ready** to serve traffic
- Removes pod from service endpoints if fails
- Use when container needs time to initialize

### Startup Probe
- **Detects if application has started**
- Disables liveness/readiness until succeeds
- Useful for slow-starting applications

### Probe Types
- **HTTP**: HTTP GET request
- **TCP**: TCP connection check
- **Exec**: Execute command

### Health Check YAML
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: healthcheck-pod
spec:
  containers:
  - name: app
    image: nginx
    livenessProbe:
      httpGet:
        path: /health
        port: 8080
      initialDelaySeconds: 30
      periodSeconds: 10
      timeoutSeconds: 5
      failureThreshold: 3
    readinessProbe:
      httpGet:
        path: /ready
        port: 8080
      initialDelaySeconds: 5
      periodSeconds: 5
    startupProbe:
      httpGet:
        path: /startup
        port: 8080
      failureThreshold: 30
      periodSeconds: 10
```

### Probe Parameters
- **initialDelaySeconds**: Wait before first probe
- **periodSeconds**: How often to probe
- **timeoutSeconds**: Probe timeout
- **successThreshold**: Consecutive successes needed
- **failureThreshold**: Consecutive failures before action

---

## Scaling

### Manual Scaling
```bash
# Scale deployment
kubectl scale deployment nginx-deployment --replicas=5

# Scale replicaset
kubectl scale replicaset nginx-replicaset --replicas=3
```

### Horizontal Pod Autoscaler (HPA)

#### What is HPA?
- **Automatically scales** number of pods
- Based on CPU, memory, or custom metrics
- Requires metrics-server

#### HPA YAML
```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: nginx-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: nginx-deployment
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
```

#### HPA Commands
```bash
# Create HPA
kubectl apply -f hpa.yaml
kubectl autoscale deployment nginx-deployment --cpu-percent=70 --min=2 --max=10

# Get HPA
kubectl get hpa

# Describe HPA
kubectl describe hpa nginx-hpa

# Delete HPA
kubectl delete hpa nginx-hpa
```

### Vertical Pod Autoscaler (VPA)
- Adjusts CPU/memory requests and limits
- Requires VPA controller installation

---

## Rolling Updates and Rollbacks

### Rolling Update Strategy
```yaml
spec:
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1          # Can create 1 extra pod
      maxUnavailable: 0  # No pods unavailable during update
```

### Update Process
1. Create new ReplicaSet with new image
2. Gradually scale up new ReplicaSet
3. Gradually scale down old ReplicaSet
4. Zero downtime if configured correctly

### Update Commands
```bash
# Update image
kubectl set image deployment/nginx-deployment nginx=nginx:1.22

# Update with edit
kubectl edit deployment nginx-deployment

# Check rollout status
kubectl rollout status deployment/nginx-deployment

# Pause rollout
kubectl rollout pause deployment/nginx-deployment

# Resume rollout
kubectl rollout resume deployment/nginx-deployment
```

### Rollback
```bash
# View rollout history
kubectl rollout history deployment/nginx-deployment

# View specific revision
kubectl rollout history deployment/nginx-deployment --revision=2

# Rollback to previous version
kubectl rollout undo deployment/nginx-deployment

# Rollback to specific revision
kubectl rollout undo deployment/nginx-deployment --to-revision=2
```

---

## Networking

### Cluster Networking
- **Pod Network**: All pods can communicate
- **Service Network**: Virtual IPs for services
- **CNI Plugins**: Calico, Flannel, Weave, etc.

### Network Policies
- **Control traffic** between pods
- Default: all pods can communicate
- With NetworkPolicy: only allowed traffic

### Network Policy YAML
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: app-network-policy
spec:
  podSelector:
    matchLabels:
      app: nginx
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend
    ports:
    - protocol: TCP
      port: 80
  egress:
  - to:
    - podSelector:
        matchLabels:
          app: database
    ports:
    - protocol: TCP
      port: 5432
```

### Network Policy Commands
```bash
# Get network policies
kubectl get networkpolicies
kubectl get netpol

# Describe network policy
kubectl describe networkpolicy app-network-policy
```

---

## Service Discovery

### DNS-Based Discovery
- Services get DNS name: `<service-name>.<namespace>.svc.cluster.local`
- Short name: `<service-name>` (same namespace)
- Cross-namespace: `<service-name>.<namespace>`

### Environment Variables
- Kubernetes injects service environment variables
- Format: `<SERVICE_NAME>_SERVICE_HOST`, `<SERVICE_NAME>_SERVICE_PORT`

### Example Service Discovery
```yaml
# Service
apiVersion: v1
kind: Service
metadata:
  name: database
spec:
  selector:
    app: postgres
  ports:
  - port: 5432

# Pod can access via:
# - http://database:5432 (same namespace)
# - http://database.default.svc.cluster.local:5432 (full DNS)
```

---

## Troubleshooting

### Common Issues

#### Pod Not Starting
```bash
# Check pod status
kubectl get pods
kubectl describe pod <pod-name>

# Check events
kubectl get events --sort-by=.metadata.creationTimestamp

# Check logs
kubectl logs <pod-name>
kubectl logs <pod-name> -c <container-name>  # Multi-container
```

#### Pod CrashLoopBackOff
```bash
# Check logs
kubectl logs <pod-name> --previous

# Describe pod
kubectl describe pod <pod-name>

# Check resource limits
kubectl top pod <pod-name>
```

#### Service Not Working
```bash
# Check service endpoints
kubectl get endpoints <service-name>

# Check service selector matches pod labels
kubectl get pods --show-labels
kubectl describe service <service-name>

# Test service from pod
kubectl run -it --rm debug --image=busybox --restart=Never -- wget -O- http://<service-name>:<port>
```

#### Image Pull Errors
```bash
# Check image pull secrets
kubectl get secrets

# Describe pod for pull errors
kubectl describe pod <pod-name>
```

### Debugging Commands
```bash
# Get all resources
kubectl get all
kubectl get all -n <namespace>

# Describe resource
kubectl describe <resource-type> <resource-name>

# Get resource YAML
kubectl get <resource-type> <resource-name> -o yaml

# Get resource JSON
kubectl get <resource-type> <resource-name> -o json

# Watch resources
kubectl get pods -w
kubectl get deployments -w

# Execute command in pod
kubectl exec -it <pod-name> -- bash
kubectl exec <pod-name> -- env

# Port forward
kubectl port-forward <pod-name> 8080:80
kubectl port-forward service/<service-name> 8080:80

# Copy files
kubectl cp <pod-name>:/path/to/file ./local-file
kubectl cp ./local-file <pod-name>:/path/to/file
```

### Logs and Events
```bash
# Pod logs
kubectl logs <pod-name>
kubectl logs -f <pod-name>  # Follow
kubectl logs --tail=100 <pod-name>  # Last 100 lines
kubectl logs --since=1h <pod-name>  # Last hour

# Previous container logs
kubectl logs <pod-name> --previous

# All pod logs in deployment
kubectl logs -l app=nginx

# Events
kubectl get events
kubectl get events -n <namespace>
kubectl get events --field-selector involvedObject.name=<pod-name>
```

---

## Best Practices

### Security
1. **Don't run as root**: Use securityContext
2. **Use secrets** for sensitive data
3. **Network policies** for pod isolation
4. **RBAC** for access control
5. **Image scanning** for vulnerabilities
6. **Resource limits** to prevent DoS

### Resource Management
1. **Set requests and limits** for all containers
2. **Use resource quotas** per namespace
3. **Monitor resource usage** regularly
4. **Use HPA** for automatic scaling

### Configuration
1. **Use ConfigMaps** for configuration
2. **Use Secrets** for sensitive data
3. **Avoid hardcoding** values in images
4. **Use environment variables** or mounted files

### Deployment
1. **Use Deployments** instead of Pods directly
2. **Set resource requests/limits**
3. **Configure health checks** (liveness, readiness)
4. **Use rolling updates** for zero downtime
5. **Tag images** with versions (avoid `latest`)

### Monitoring
1. **Monitor pod status**
2. **Check logs regularly**
3. **Set up alerts** for failures
4. **Use monitoring tools** (Prometheus, Grafana)

### Example Production-Ready Deployment
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: production-app
  labels:
    app: production-app
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  selector:
    matchLabels:
      app: production-app
  template:
    metadata:
      labels:
        app: production-app
    spec:
      securityContext:
        runAsNonRoot: true
        runAsUser: 1000
      containers:
      - name: app
        image: myapp:v1.0.0
        ports:
        - containerPort: 3000
        env:
        - name: NODE_ENV
          value: "production"
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: app-secret
              key: database-url
        resources:
          requests:
            memory: "256Mi"
            cpu: "250m"
          limits:
            memory: "512Mi"
            cpu: "500m"
        livenessProbe:
          httpGet:
            path: /health
            port: 3000
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /ready
            port: 3000
          initialDelaySeconds: 5
          periodSeconds: 5
        volumeMounts:
        - name: config
          mountPath: /app/config
      volumes:
      - name: config
        configMap:
          name: app-config
---
apiVersion: v1
kind: Service
metadata:
  name: production-app-service
spec:
  type: LoadBalancer
  selector:
    app: production-app
  ports:
  - port: 80
    targetPort: 3000
---
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: production-app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: production-app
  minReplicas: 3
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
```

---

## Essential kubectl Commands Cheat Sheet

```bash
# Cluster info
kubectl cluster-info
kubectl get nodes

# Namespaces
kubectl get ns
kubectl create ns <name>
kubectl config set-context --current --namespace=<name>

# Pods
kubectl get pods
kubectl get pods -o wide
kubectl describe pod <name>
kubectl logs <pod-name>
kubectl exec -it <pod-name> -- bash
kubectl delete pod <name>

# Deployments
kubectl get deployments
kubectl scale deployment <name> --replicas=5
kubectl rollout status deployment/<name>
kubectl rollout history deployment/<name>
kubectl rollout undo deployment/<name>

# Services
kubectl get services
kubectl expose deployment <name> --port=80 --type=NodePort
kubectl get endpoints <service-name>

# ConfigMaps & Secrets
kubectl get configmaps
kubectl get secrets
kubectl create configmap <name> --from-literal=key=value
kubectl create secret generic <name> --from-literal=key=value

# Apply/Delete
kubectl apply -f <file.yaml>
kubectl delete -f <file.yaml>
kubectl get all

# Debugging
kubectl get events
kubectl top nodes
kubectl top pods
kubectl port-forward <pod-name> 8080:80
```

---

## Summary

### Key Takeaways
- **Pods** are the smallest deployable unit
- **Deployments** manage ReplicaSets and provide updates
- **Services** provide stable network endpoints
- **ConfigMaps/Secrets** manage configuration
- **Namespaces** provide isolation
- **Health checks** ensure reliability
- **HPA** enables automatic scaling
- **Rolling updates** provide zero downtime

### Learning Path
1. Start with Pods and Deployments
2. Learn Services for networking
3. Understand ConfigMaps and Secrets
4. Explore advanced resources (StatefulSets, Jobs)
5. Master scaling and updates
6. Practice troubleshooting

---

## Next Steps
- Practice deploying real applications
- Learn about Helm for package management
- Explore service meshes (Istio, Linkerd)
- Study advanced topics (operators, custom resources)
- Set up monitoring and logging
- Learn about GitOps (ArgoCD, Flux)

