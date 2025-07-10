# CrashLoopBackOff

## What is CrashLoopBackOff?

CrashLoopBackOff is an error state where your pod keeps crashing and restarting repeatedly. Kubernetes fails to start the pod and retries several times, leading to a "crash loop" with exponential backoff delays between restart attempts.

## Why Does CrashLoopBackOff Happen?

### Common Causes:

1. **Misconfiguration**: Wrong commands, environment variables, or configuration files
2. **Liveness Probe Failures**: Health check fails causing Kubelet to restart the pod
3. **Memory Limits Too Low**: Leads to OOMKilled (Out of Memory) errors
4. **Long Init Containers**: Timeout issues during initialization
5. **Application Bugs**: Code exceptions causing immediate crashes
6. **Resource Constraints**: Insufficient CPU or memory allocated

## Restart Cycle & Backoff Mechanism

When a pod crashes, Kubelet tries to restart it with increasing delays:

```
Waiting → Running (error) → Crashed → Waiting (longer) → Loop
```

**Backoff Timeline:**
- First restart: Immediate
- Second restart: 10 seconds wait
- Third restart: 20 seconds wait  
- Fourth restart: 40 seconds wait
- Continues with exponential backoff

This exponential waiting period is called **CrashLoopBackOff**.

## Hands-On Examples & Solutions

### 1. Wrong Command Error

**Problem**: Deploying an app with incorrect CMD or ENTRYPOINT

```yaml
# wrong-cmd.yaml - Example of wrong command
apiVersion: apps/v1
kind: Deployment
metadata:
  name: wrong-cmd-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: wrong-cmd-app
  template:
    metadata:
      labels:
        app: wrong-cmd-app
    spec:
      containers:
      - name: app
        image: nginx:latest
        command: ["/wrong/command"]  # This will cause crash
```

**Solution**: 
1. Create correct Dockerfile with proper CMD/ENTRYPOINT
2. Build and push image to registry
3. Use correct image in deploy.yaml

```yaml
# Corrected version
spec:
  containers:
  - name: app
    image: nginx:latest
    # Remove wrong command, use default nginx command
```

### 2. Liveness Probe Failures

**Theory**: Liveness probe checks if the app is healthy. If it fails, Kubelet restarts the pod.

```yaml
# liveness-prob.yaml - Example with liveness probe
apiVersion: apps/v1
kind: Deployment
metadata:
  name: liveness-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: liveness-app
  template:
    metadata:
      labels:
        app: liveness-app
    spec:
      containers:
      - name: app
        image: nginx:latest
        ports:
        - containerPort: 80
        livenessProbe:
          httpGet:
            path: /health  # This endpoint must exist
            port: 80
          initialDelaySeconds: 30
          periodSeconds: 10
```

**How it works**:
- Kubelet checks `/health` endpoint every 10 seconds
- If probe fails → Kubelet restarts the pod
- This is for app's internal health monitoring

### 3. Readiness Probe vs Liveness Probe

```yaml
# Complete probe configuration
spec:
  containers:
  - name: app
    image: nginx:latest
    livenessProbe:
      httpGet:
        path: /health
        port: 80
      initialDelaySeconds: 30
    readinessProbe:
      httpGet:
        path: /ready
        port: 80
      initialDelaySeconds: 5
```

**Key Differences**:
- **Liveness Probe**: Restarts pod if fails (for crash recovery)
- **Readiness Probe**: Removes pod from service if fails (doesn't restart)

### 4. OOMKilled (Out of Memory)

**Problem**: Pod consumes more memory than its limit

```yaml
# oom_crash.yaml - Example causing OOM
apiVersion: apps/v1
kind: Deployment
metadata:
  name: oom-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: oom-app
  template:
    metadata:
      labels:
        app: oom-app
    spec:
      containers:
      - name: memory-hog
        image: nginx:latest
        resources:
          requests:
            memory: "64Mi"    # Too low for the application
          limits:
            memory: "128Mi"   # Application needs more
```

**Solution**: Increase resource limits

```yaml
# Fixed version with proper resources
spec:
  containers:
  - name: app
    image: nginx:latest
    resources:
      requests:
        memory: "256Mi"
        cpu: "250m"
      limits:
        memory: "512Mi"
        cpu: "500m"
```

## Troubleshooting Commands

```bash
# Watch pod status in real-time
kubectl get pods -w
# Purpose: Monitor pod restart cycles and status changes

# Get detailed pod information
kubectl describe pod <pod-name>
# Purpose: See events, restart count, and failure reasons (including OOMKilled)

# Check pod logs from current container
kubectl logs <pod-name>
# Purpose: View application logs to identify crash causes

# Check logs from previous crashed container
kubectl logs <pod-name> --previous
# Purpose: See logs from the container that crashed

# Get pod events sorted by time
kubectl get events --sort-by=.metadata.creationTimestamp
# Purpose: View chronological events including crash reasons

# Check resource usage
kubectl top pod <pod-name>
# Purpose: Monitor CPU and memory consumption
```

## Resource Quotas & Namespaces (Advanced)

### Problem Scenario:
Kubernetes can apply resource quotas per namespace to prevent resource abuse.

**Example Issue**:
- Pods A, B, C in same namespace
- Pod A uses all available resources
- Pods B and C fail to start (no memory left)

### Solution: Define Resource Quotas

```yaml
# namespace-quota.yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: compute-quota
  namespace: development
spec:
  hard:
    requests.cpu: "4"
    requests.memory: 8Gi
    limits.cpu: "8"
    limits.memory: 16Gi
    pods: "10"
```

```bash
# Apply resource quota
kubectl apply -f namespace-quota.yaml
# Purpose: Limit resource usage per namespace

# Check quota usage
kubectl describe quota compute-quota -n development
# Purpose: Monitor resource consumption against limits
```

## Prevention Best Practices

1. **Test Locally**: Always test containers locally before deploying
2. **Set Resource Limits**: Define appropriate CPU and memory limits
3. **Health Checks**: Implement proper liveness and readiness probes
4. **Monitor Logs**: Regularly check application logs for errors
5. **Gradual Rollouts**: Use rolling updates to catch issues early
6. **Resource Monitoring**: Monitor cluster resource usage

## Quick Diagnosis Workflow

```bash
# Step 1: Check pod status
kubectl get pods

# Step 2: If CrashLoopBackOff, describe the pod
kubectl describe pod <pod-name>

# Step 3: Check current and previous logs
kubectl logs <pod-name>
kubectl logs <pod-name> --previous

# Step 4: Look for specific issues:
# - OOMKilled in describe output
# - Liveness probe failures in events
# - Wrong command in container spec
# - Resource limit issues

# Step 5: Fix the root cause and redeploy
kubectl apply -f fixed-deployment.yaml
```
