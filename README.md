# Kubernetes Learning Guide: Blog Application Deployment

## 📚 Learning Scope & Todo List

This comprehensive guide covers essential Kubernetes concepts through hands-on deployment of a multi-tier blog application. Below is the learning progression:

### 🎯 Training Roadmap

- [x] **Step 1: Environment Setup** - Verify minikube and kubectl configuration
- [x] **Step 2: Core Concepts** - Understand Pods, Deployments, Services, ConfigMaps, Secrets, PVCs
- [x] **Step 3: Database Layer** - Deploy PostgreSQL with persistent storage and secrets
- [x] **Step 4: Cache Layer** - Deploy Redis for stateless service concepts
- [x] **Step 5: Application Layer** - Deploy Flask backend with scaling and configuration
- [x] **Step 6: Frontend Layer** - Deploy Node.js frontend with LoadBalancer
- [x] **Step 7: Networking** - Test connectivity and service discovery
- [x] **Step 8: Monitoring** - Learn debugging and troubleshooting techniques
- [x] **Step 9: Operations** - Practice scaling, rolling updates, and rollbacks
- [x] **Step 10: Advanced Features** - Explore namespaces, quotas, and resource management

---

## 🏗️ Application Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    KUBERNETES CLUSTER                       │
│                                                             │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐     │
│  │   Internet  │────│  Frontend   │────│   Backend   │     │
│  │             │    │ (Node.js)   │    │  (Flask)    │     │
│  │             │    │LoadBalancer │    │ ClusterIP   │     │
│  └─────────────┘    │  2 replicas │    │ 3 replicas  │     │
│                     └─────────────┘    └─────────────┘     │
│                                               │             │
│                     ┌─────────────┐    ┌─────────────┐     │
│                     │   Redis     │    │ PostgreSQL  │     │
│                     │  (Cache)    │    │ (Database)  │     │
│                     │ ClusterIP   │    │ ClusterIP   │     │
│                     │ 1 replica   │    │ 1 replica   │     │
│                     └─────────────┘    └─────────────┘     │
│                                               │             │
│                                        ┌─────────────┐     │
│                                        │Persistent   │     │
│                                        │Volume       │     │
│                                        └─────────────┘     │
└─────────────────────────────────────────────────────────────┘
```

---

## 🔑 Core Kubernetes Concepts

### 1. **Pods**
The smallest deployable unit in Kubernetes.
- **Purpose**: Run one or more containers together
- **Characteristics**: Share network, storage, and lifecycle
- **Example**: Each database, backend, frontend instance runs in separate pods

```bash
# View pods
kubectl get pods
kubectl get pods -o wide  # With more details
```

### 2. **Deployments**
Manage Pod replicas and updates.
- **Purpose**: Ensure desired number of pods are running
- **Features**: Rolling updates, rollbacks, scaling
- **Example**: Backend deployment with 3 replicas for high availability

```bash
# Scale deployment
kubectl scale deployment backend --replicas=5

# Rolling update
kubectl set image deployment/backend backend=python:3.12-slim

# Rollback
kubectl rollout undo deployment/backend
```

### 3. **Services**
Enable network communication between pods.

#### Service Types:
- **ClusterIP**: Internal cluster communication only
- **LoadBalancer**: External access with cloud provider integration
- **NodePort**: External access via node IP and port

```bash
# View services
kubectl get services
kubectl get endpoints  # See which pods services point to
```

### 4. **ConfigMaps**
Store non-sensitive configuration data.
- **Purpose**: Separate config from container images
- **Usage**: Environment variables, config files
- **Example**: Database URLs, API endpoints

```bash
# View ConfigMap
kubectl get configmap backend-config -o yaml
```

### 5. **Secrets**
Store sensitive information securely.
- **Purpose**: Passwords, tokens, keys
- **Features**: Base64 encoded, encrypted at rest
- **Example**: Database credentials

```bash
# View Secret (values are encoded)
kubectl get secret postgres-secret -o yaml
```

### 6. **Persistent Volumes (PV) & Claims (PVC)**
Provide durable storage for stateful applications.
- **PV**: Storage resource in cluster
- **PVC**: Request for storage by pod
- **Example**: PostgreSQL data persistence

```bash
# View storage
kubectl get pv,pvc
```

---

## 🚀 Deployment Guide

### Prerequisites

1. **Install minikube**
2. **Install kubectl**
3. **Start Docker Desktop**

### Step 1: Start Minikube

```bash
# Start cluster
minikube start --driver=docker

# Verify cluster
kubectl cluster-info
kubectl get nodes
```

### Step 2: Deploy Database Layer

```bash
# Deploy PostgreSQL with persistence
kubectl apply -f postgres-deployment.yml

# Verify deployment
kubectl get pods -l app=postgres
kubectl logs deployment/postgres
```

**What this creates:**
- Secret for database credentials
- PersistentVolumeClaim for data storage
- Deployment with 1 replica
- ClusterIP service for internal access

### Step 3: Deploy Cache Layer

```bash
# Deploy Redis
kubectl apply -f redis-deployment.yml

# Test connectivity
kubectl run redis-test --image=redis:7-alpine --rm -it --restart=Never -- redis-cli -h redis-service ping
```

### Step 4: Deploy Application Backend

```bash
# Deploy Flask backend
kubectl apply -f backend-deployment.yml

# Test API
kubectl run curl-test --image=curlimages/curl --rm -it --restart=Never -- curl -s http://backend-service:5000/api/health
```

**Key features:**
- ConfigMap for environment variables
- 3 replicas for high availability
- Resource requests and limits
- Health check endpoints

### Step 5: Deploy Frontend

```bash
# Deploy Node.js frontend
kubectl apply -f frontend-deployment.yml

# Access application
minikube service frontend-service
```

### Step 6: Enable Monitoring

```bash
# Enable metrics server for resource monitoring if not already enabled.(check addons)
minikube addons enable metrics-server

# Check resource usage
kubectl top nodes
kubectl top pods
```

---

## 🔧 Essential kubectl Commands

### Pod Management
```bash
# List pods
kubectl get pods
kubectl get pods -o wide
kubectl get pods -l app=backend  # Filter by label

# Pod details
kubectl describe pod <pod-name>
kubectl logs <pod-name>
kubectl logs -f <pod-name>  # Follow logs
kubectl logs deployment/backend  # All pods in deployment

# Execute commands in pod
kubectl exec <pod-name> -- <command>
kubectl exec -it <pod-name> -- /bin/bash  # Interactive shell
```

### Deployment Operations
```bash
# Apply configuration
kubectl apply -f <file.yml>

# Scale deployment
kubectl scale deployment <name> --replicas=<count>

# Rolling update
kubectl set image deployment/<name> <container>=<new-image>

# Check rollout status
kubectl rollout status deployment/<name>

# View rollout history
kubectl rollout history deployment/<name>

# Rollback
kubectl rollout undo deployment/<name>
```

### Service & Networking
```bash
# List services
kubectl get services
kubectl get endpoints

# Test connectivity
kubectl run test-pod --image=curlimages/curl --rm -it --restart=Never -- curl <service-url>

# DNS testing
kubectl run dns-test --image=busybox --rm -it --restart=Never -- nslookup <service-name>
```

### Monitoring & Debugging
```bash
# Resource usage
kubectl top nodes
kubectl top pods

# Events
kubectl get events --sort-by='.lastTimestamp'

# Cluster info
kubectl cluster-info
kubectl get all  # All resources
```

### Configuration Management
```bash
# View ConfigMaps
kubectl get configmaps
kubectl describe configmap <name>

# View Secrets
kubectl get secrets
kubectl describe secret <name>
```

---

## 📊 Scaling and Updates

### Horizontal Scaling

```bash
# Scale up
kubectl scale deployment backend --replicas=5

# Scale down  
kubectl scale deployment backend --replicas=2

# Check scaling
kubectl get pods -l app=backend
```

### Rolling Updates

```bash
# Update image
kubectl set image deployment/backend backend=python:3.12-slim

# Monitor update
kubectl rollout status deployment/backend

# Verify update
kubectl exec <pod-name> -- python --version
```

### Rollback Strategy

```bash
# Rollback to previous version
kubectl rollout undo deployment/backend

# Rollback to specific revision
kubectl rollout undo deployment/backend --to-revision=1

# Check history
kubectl rollout history deployment/backend
```

---

## 🏢 Advanced Features

### Namespaces for Environment Separation

```bash
# Create namespaces
kubectl create namespace staging
kubectl create namespace production

# Deploy to specific namespace
kubectl apply -f redis-deployment.yml -n staging

# List resources in namespace
kubectl get pods -n staging

# Cross-namespace service access
<service-name>.<namespace>.svc.cluster.local
```

### Resource Quotas and Limits

```yaml
# Resource Quota
apiVersion: v1
kind: ResourceQuota
metadata:
  name: staging-quota
  namespace: staging
spec:
  hard:
    requests.cpu: "1000m"
    requests.memory: 2Gi
    limits.cpu: "2000m" 
    limits.memory: 4Gi
    pods: "10"
```

```bash
# Check quota usage
kubectl describe quota <quota-name> -n <namespace>
```

---

## 🔍 Troubleshooting Guide

### Common Issues and Solutions

#### 1. Pod Stuck in Pending
```bash
# Check events
kubectl describe pod <pod-name>

# Common causes:
# - Insufficient resources
# - PVC not bound
# - Image pull issues
```

#### 2. Service Not Accessible
```bash
# Check service endpoints
kubectl get endpoints <service-name>

# Verify pod labels match service selector
kubectl get pods --show-labels
```

#### 3. Image Pull Errors
```bash
# Check pod events
kubectl describe pod <pod-name>

# Verify image name and tag
# Check image registry accessibility
```

#### 4. Resource Constraints
```bash
# Check resource usage
kubectl top pods
kubectl describe node

# Verify resource requests/limits
kubectl describe deployment <name>
```

---

## 🔒 Security Best Practices

### 1. **Use Secrets for Sensitive Data**
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-secret
type: Opaque
stringData:
  username: "admin"
  password: "secretpassword"
```

### 2. **Set Resource Limits**
```yaml
resources:
  requests:
    memory: "128Mi"
    cpu: "100m"
  limits:
    memory: "256Mi"
    cpu: "200m"
```

### 3. **Use Non-Root Containers**
```yaml
securityContext:
  runAsNonRoot: true
  runAsUser: 1000
```

---

## 📈 Production Considerations

### 1. **High Availability**
- Run multiple replicas
- Use anti-affinity rules
- Spread across availability zones

### 2. **Monitoring**
- Implement health checks
- Use Prometheus + Grafana
- Set up alerting

### 3. **Backup Strategy**
- Regular database backups
- Persistent volume snapshots
- Configuration backup

### 4. **Network Security**
- Network policies
- Service mesh (Istio)
- TLS encryption

---

## 🎓 Learning Outcomes

After completing this guide, you will understand:

✅ **Container Orchestration**
- Pod lifecycle and management
- Deployment strategies
- Service discovery

✅ **Storage Management**
- Persistent volumes
- StatefulSets concepts
- Data persistence patterns

✅ **Networking**
- Service types and use cases
- DNS-based service discovery
- Load balancing

✅ **Configuration Management**
- ConfigMaps and Secrets
- Environment-specific configurations
- Security best practices

✅ **Operations**
- Scaling applications
- Rolling updates and rollbacks
- Monitoring and debugging

✅ **Advanced Topics**
- Namespace isolation
- Resource quotas
- Multi-environment deployments

---

## 🔗 Next Steps

1. **Helm**: Package manager for Kubernetes
2. **Ingress Controllers**: Advanced routing and SSL
3. **CI/CD Integration**: GitOps and automated deployments  
4. **Service Mesh**: Istio for microservices communication
5. **Observability**: Prometheus, Grafana, Jaeger
6. **Security**: RBAC, Pod Security Standards

---

## 📚 Additional Resources

- [Kubernetes Official Documentation](https://kubernetes.io/docs/)
- [kubectl Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)
- [Kubernetes Best Practices](https://kubernetes.io/docs/concepts/configuration/overview/)
- [minikube Documentation](https://minikube.sigs.k8s.io/docs/)

---

*This guide provides hands-on experience with real-world Kubernetes concepts and prepares you for production container orchestration scenarios.*