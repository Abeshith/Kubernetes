# K8S Pods

## What is a Pod?

A Pod is the smallest deployable unit in Kubernetes. It represents a single instance of a running process in your cluster and can contain one or more containers that share the same network and storage.

## Why Use Pods?

- **Atomic Unit**: Pods ensure containers are deployed, scaled, and managed together
- **Shared Resources**: Containers in a pod share network (IP address) and storage volumes
- **Co-location**: Tightly coupled containers can communicate via localhost
- **Lifecycle Management**: All containers in a pod are started and stopped together

## Key Characteristics

- Each pod gets its own IP address
- Containers within a pod can communicate using `localhost`
- Pods are ephemeral - they can be created, destroyed, and recreated
- Usually managed by higher-level controllers (Deployments, ReplicaSets)

## Essential Pod Commands

### Basic Pod Operations

```bash
# Create a pod from YAML file
kubectl create -f pod.yaml
# Purpose: Deploy a pod using configuration file

# Get list of all pods
kubectl get pods
# Purpose: View all pods in current namespace

# Get detailed pod information with IP and node details
kubectl get pods -o wide
# Purpose: View pods with additional details like IP addresses and nodes

# Get pod details in YAML format
kubectl get pods <pod-name> -o yaml
# Purpose: Export pod configuration for inspection or backup
```

### Pod Monitoring & Debugging

```bash
# View detailed pod information and events
kubectl describe pod <pod-name>
# Purpose: Get complete details about pod including events and troubleshooting info

# View pod logs
kubectl logs <pod-name>
# Purpose: Check application logs for debugging

# Follow pod logs in real-time
kubectl logs -f <pod-name>
# Purpose: Monitor live log output from the pod

# Access pod shell for debugging
kubectl exec -it <pod-name> -- /bin/bash
# Purpose: Get interactive shell access inside the pod for troubleshooting
```

### Pod Management

```bash
# Delete a specific pod
kubectl delete pod <pod-name>
# Purpose: Remove a pod from the cluster

# Delete pod using configuration file
kubectl delete -f pod.yaml
# Purpose: Remove pod using the same file used to create it

# Force delete a stuck pod
kubectl delete pod <pod-name> --force --grace-period=0
# Purpose: Forcefully remove a pod that's not responding to normal deletion
```
