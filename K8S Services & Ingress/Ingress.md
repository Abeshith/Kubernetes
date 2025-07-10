# Kubernetes Ingress

## What is Ingress?

Ingress is a Kubernetes resource that manages external HTTP and HTTPS access to services within a cluster. It provides load balancing, SSL termination, and name-based virtual hosting, acting as a smart router that directs traffic to different services based on rules.

## Why Use Ingress?

- **Single Entry Point**: One external IP/domain for multiple services
- **Path-Based Routing**: Route traffic based on URL paths (e.g., `/api`, `/web`)
- **Host-Based Routing**: Route traffic based on hostnames (e.g., `api.example.com`, `web.example.com`)
- **SSL/TLS Termination**: Handle HTTPS certificates centrally
- **Cost Effective**: Avoid multiple LoadBalancer services (which can be expensive in cloud)
- **Advanced Features**: Rate limiting, authentication, redirects

## Ingress Controller

### What is an Ingress Controller?

An **Ingress Controller** is the actual implementation that fulfills the Ingress rules. It's a pod running in your cluster that:

- Watches for Ingress resources
- Configures the load balancer according to Ingress rules
- Handles the actual traffic routing

### Popular Ingress Controllers:

- **NGINX Ingress Controller**: Most popular, feature-rich
- **Traefik**: Easy to use, great for microservices
- **HAProxy**: High performance
- **AWS ALB**: For AWS Application Load Balancer
- **GCE**: For Google Cloud Load Balancer

### Key Point:
**Ingress resource alone does nothing** - you need an Ingress Controller to make it work!

## Complete Ingress Deployment Process

### Step 1: Deploy Application (Deployment)

```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
  labels:
    app: web-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web-app
  template:
    metadata:
      labels:
        app: web-app
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
```

```bash
# Apply deployment configuration
kubectl apply -f deployment.yaml
# Purpose: Create the application deployment that will receive traffic

# Verify deployment creation
kubectl get deployments
# Purpose: Confirm deployment is created and pods are running
```

### Step 2: Expose Deployment (Service)

```yaml
# service.yaml
apiVersion: v1
kind: Service
metadata:
  name: web-app
  labels:
    app: web-app
spec:
  selector:
    app: web-app
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
  type: ClusterIP
```

```bash
# Apply service configuration
kubectl apply -f service.yaml
# Purpose: Create internal service to route traffic to pods

# Verify service creation
kubectl get services
# Purpose: Confirm service is created and has ClusterIP
```

### Step 3: Install Ingress Controller (if not installed)

```bash
# For Minikube - Enable NGINX Ingress Controller
minikube addons enable ingress
# Purpose: Install and enable NGINX Ingress Controller in Minikube

```

### Step 4: Create Ingress Resource (ingress.yaml)

```yaml
# ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: web-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: foo.bar.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: web-app
            port:
              number: 80
```

### Step 5: Apply Ingress Configuration

```bash
# Create the ingress resource
kubectl apply -f ingress.yaml
# Purpose: Deploy ingress rules to the cluster

# Verify ingress creation
kubectl get ingress
# Purpose: Check ingress status and get external IP/address
```

## Essential Ingress Commands

### Ingress Monitoring & Management

```bash
# Get all ingress resources
kubectl get ingress
# Purpose: View all ingress resources and their addresses

# Get detailed ingress information
kubectl describe ingress <ingress-name>
# Purpose: See ingress rules, events, and backend service details

# Get ingress with wide output
kubectl get ingress -o wide
# Purpose: View ingress with additional details like class and address

# Watch ingress for address updates
kubectl get ingress --watch
# Purpose: Monitor ingress until external address is assigned

# Get ingress in YAML format
kubectl get ingress <ingress-name> -o yaml
# Purpose: Export ingress configuration for backup or modification

# Edit ingress configuration
kubectl edit ingress <ingress-name>
# Purpose: Modify ingress rules in real-time
```

### Troubleshooting Commands

```bash
# Check Ingress Controller pods
kubectl get pods -n ingress-nginx
# Purpose: Verify Ingress Controller is running properly

```

## Local Development Setup

### For Minikube Users

```bash
# Step 1: Get ingress IP address
kubectl get ingress
# Look for ADDRESS column - usually shows Minikube IP

# Step 2: Get Minikube IP (alternative method)
minikube ip
# Purpose: Get the IP address to add to hosts file

# Step 3: Edit hosts file (Windows)
# Open PowerShell as Administrator
notepad C:\Windows\System32\drivers\etc\hosts
# Add line: <INGRESS-IP> foo.bar.com

# Step 3: Edit hosts file (Linux/Mac)
sudo vim /etc/hosts
# Add line: <INGRESS-IP> foo.bar.com

# Step 4: Test domain resolution
ping foo.bar.com
# Purpose: Verify that foo.bar.com resolves to Minikube IP

# Step 5: Access application via browser or curl
curl http://foo.bar.com
# Purpose: Test ingress routing through custom domain
```

## Quick Reference Commands

```bash
# Complete deployment to ingress workflow using YAML files
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
kubectl apply -f ingress.yaml
kubectl get ingress

# Check ingress address assignment
kubectl get ingress --watch

# Test local ingress setup
ping foo.bar.com
curl http://foo.bar.com

# Cleanup
kubectl delete -f ingress.yaml
kubectl delete -f service.yaml
kubectl delete -f deployment.yaml
```

## Key Points to Remember

1. **Ingress Controller Required**: Install before creating Ingress resources
2. **Service Dependency**: Ingress routes to Services, not directly to Pods
3. **DNS Setup**: For local development, update `/etc/hosts` file
4. **Address Assignment**: Wait for ingress to get an address before testing
5. **Path Types**: Use `Prefix` for directory-style routing, `Exact` for exact matches