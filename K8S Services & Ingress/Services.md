# K8S Services & Ingress

## What is a Service?

A Service in Kubernetes is an abstraction that defines a logical set of pods and provides a stable network endpoint to access them. Services enable communication between different components of your application and provide load balancing across multiple pod replicas.

## How Services Work - Labels and Selectors

Services use **labels and selectors** to identify which pods they should route traffic to:

- **Labels**: Key-value pairs attached to pods (e.g., `app: nginx`, `version: v1.0`)
- **Selectors**: Service configuration that matches pods with specific labels
- **Dynamic Discovery**: When pods are created/destroyed, services automatically update their endpoint list based on label matching
- **Loose Coupling**: Services don't need to know specific pod IPs, they just target labels

**Example**: A service with selector `app: web` will route traffic to all pods labeled with `app: web`, regardless of their IP addresses.

## Why Use Services?

- **Stable Endpoint**: Provides consistent IP and DNS name even when pods are recreated
- **Load Balancing**: Distributes traffic across multiple pod replicas
- **Service Discovery**: Enables pods to find and communicate with each other
- **Decoupling**: Separates frontend from backend components

## Service Types Explained

### ClusterIP (Default)
- **What it is**: Creates an internal IP address accessible only within the cluster
- **Use case**: Internal communication between services (e.g., frontend to backend)
- **Access**: Only accessible from within the cluster nodes and pods

### NodePort
- **What it is**: Exposes service on each node's IP at a static port (30000-32767)
- **Use case**: External access to applications for development/testing
- **Access**: Accessible from outside cluster using `<NodeIP>:<NodePort>`

### LoadBalancer
- **What it is**: Creates an external load balancer (requires cloud provider support)
- **Use case**: Production external access with proper load balancing
- **Access**: Provides external IP address for accessing the service

### ExternalName
- **What it is**: Maps service to external DNS name
- **Use case**: Accessing external services through Kubernetes service abstraction

## Essential Service Commands

### Basic Service Operations

```bash
# Create a service from YAML file
kubectl create -f service.yaml
# Purpose: Deploy service using configuration file

# Create a service imperatively (ClusterIP)
kubectl expose deployment my-app --port=80 --target-port=8080
# Purpose: Quickly expose a deployment with ClusterIP service

# Create NodePort service imperatively
kubectl expose deployment my-app --type=NodePort --port=80 --target-port=8080
# Purpose: Expose deployment externally using NodePort

# Create LoadBalancer service imperatively
kubectl expose deployment my-app --type=LoadBalancer --port=80 --target-port=8080
# Purpose: Expose deployment externally using cloud load balancer

# Get list of all services
kubectl get services
# Purpose: View all services in current namespace

# Get detailed service information
kubectl get services -o wide
# Purpose: View services with additional details like endpoints and external IPs
```

### Service Monitoring & Debugging

```bash
# View detailed service information
kubectl describe service <service-name>
# Purpose: Get complete details about service including endpoints and events

# Get service endpoints
kubectl get endpoints <service-name>
# Purpose: View which pods are backing the service

# Get service details in YAML format
kubectl get service <service-name> -o yaml
# Purpose: Export service configuration for inspection or backup
```

## Accessing Applications

### ClusterIP Access (Internal Only)

```bash
# Access from within a pod in the cluster
kubectl exec -it <pod-name> -- curl http://<service-name>:<port>
# Purpose: Test service connectivity from within the cluster

# Port forward to access locally
kubectl port-forward service/<service-name> 8080:80
# Purpose: Access ClusterIP service from local machine for testing
```

### NodePort Access (External)

```bash
# Get node IP addresses
kubectl get nodes -o wide
# Purpose: Find external IP addresses of nodes

# Get Minikube IP (for local development)
minikube ip
# Purpose: Get the IP address of Minikube cluster for NodePort access

# Access application using NodePort
# http://<NodeIP>:<NodePort>
# Example: http://192.168.1.100:30080
# Purpose: Access application from outside the cluster

# Access application using Minikube IP and NodePort
# Example: http://$(minikube ip):30080
curl http://$(minikube ip):30080
# Purpose: Access application using Minikube's IP address

# Get NodePort details
kubectl get service <service-name> -o jsonpath='{.spec.ports[0].nodePort}'
# Purpose: Find the assigned NodePort number

# Test NodePort connectivity
curl http://<node-ip>:<nodeport>
# Purpose: Verify external access to the application

```

### LoadBalancer Access (External with Cloud Provider)

```bash
# For Minikube: Start tunnel in separate terminal (REQUIRED for LoadBalancer)
minikube tunnel
# Purpose: Enable LoadBalancer services in Minikube by creating network tunnel
# Note: Keep this running in a separate terminal window

# Get external IP assigned by load balancer
kubectl get service <service-name> -o jsonpath='{.status.loadBalancer.ingress[0].ip}'
# Purpose: Find the external IP assigned by cloud provider or Minikube tunnel

# Get external IP with hostname (for some cloud providers)
kubectl get service <service-name> -o jsonpath='{.status.loadBalancer.ingress[0].hostname}'
# Purpose: Get external hostname when IP is not directly assigned

# Access application using LoadBalancer IP
# http://<LoadBalancer-IP>:<Port>
# Purpose: Access application through cloud load balancer

# Check load balancer status and wait for external IP
kubectl get service <service-name> --watch
# Purpose: Monitor load balancer provisioning status (wait for EXTERNAL-IP)

# Test LoadBalancer connectivity with Minikube tunnel
curl http://127.0.0.1:80
# Purpose: Access LoadBalancer service through Minikube tunnel (usually localhost)

# Test LoadBalancer connectivity with specific IP
curl http://203.0.113.10:80
# Purpose: Verify external access through load balancer using specific IP

# Test LoadBalancer with HTTPS (if configured)
curl https://203.0.113.10:443
# Purpose: Test secure connection through load balancer

# Get complete LoadBalancer service details
kubectl get service <service-name> -o wide
# Purpose: View all LoadBalancer details including external IP and ports

# Complete Minikube LoadBalancer access example
# Terminal 1: Start tunnel
minikube tunnel

# Edit LoadBalancer service configuration
kubectl edit service <service-name>
# Purpose: Modify LoadBalancer settings like ports, selectors, or type
# Example changes: Change port, modify selectors, or convert to NodePort
```

### Service Management

```bash
# Delete a service
kubectl delete service <service-name>
# Purpose: Remove service from the cluster

# Delete service using configuration file
kubectl delete -f service.yaml
# Purpose: Remove service using the same file used to create it

# Edit service configuration in real-time
kubectl edit service <service-name>
# Purpose: Modify service settings like type, ports, selectors, or labels
# Common edits: Change service type, modify port mappings, update selectors

# Patch service with specific changes
kubectl patch service <service-name> -p '{"spec":{"type":"LoadBalancer"}}'
# Purpose: Quick modification of specific service properties

```

