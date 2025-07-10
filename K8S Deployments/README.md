# K8S Deployments

## What is a Deployment?

A Deployment is a higher-level Kubernetes resource that manages ReplicaSets and provides declarative updates for pods. It ensures that a specified number of pod replicas are running at all times and handles rolling updates and rollbacks automatically.

## Why Use Deployments?

- **Declarative Management**: Define desired state and Kubernetes maintains it
- **Rolling Updates**: Update applications without downtime
- **Rollback Capability**: Easily revert to previous versions if issues occur
- **Scaling**: Automatically scale pods up or down based on requirements
- **Self-Healing**: Automatically replace failed pods

## Key Characteristics

- Creates and manages ReplicaSets automatically
- Provides rolling update strategy for zero-downtime deployments
- Maintains revision history for easy rollbacks
- Ensures desired number of replicas are always running
- Supports multiple deployment strategies (RollingUpdate, Recreate)
- **Self-Healing**: If you delete a replica pod, it automatically creates a new one to maintain the desired state.

## Essential Deployment Commands

### Basic Deployment Operations

```bash
# Create a deployment from YAML file
kubectl create -f deployment.yaml
# Purpose: Deploy application using configuration file

# Create a deployment imperatively
kubectl create deployment my-app --image=nginx:1.20
# Purpose: Quickly create deployment with specified image

# Get list of all deployments
kubectl get deployments
# Purpose: View all deployments in current namespace

# Get detailed deployment information
kubectl get deployments -o wide
# Purpose: View deployments with additional details like images and selectors
```

### Deployment Monitoring & Debugging

```bash
# View detailed deployment information and events
kubectl describe deployment <deployment-name>
# Purpose: Get complete details about deployment including events and pod template

# Check deployment status and rollout progress
kubectl rollout status deployment/<deployment-name>
# Purpose: Monitor the progress of deployment updates

# View deployment logs from all pods
kubectl logs deployment/<deployment-name>
# Purpose: Check application logs from all pods in the deployment

# Get deployment details in YAML format
kubectl get deployment <deployment-name> -o yaml
# Purpose: Export deployment configuration for inspection or backup
```

### Deployment Management

```bash
# Delete a deployment
kubectl delete deployment <deployment-name>
# Purpose: Remove deployment and all its managed pods

# Delete deployment using configuration file
kubectl delete -f deployment.yaml
# Purpose: Remove deployment using the same file used to create it

# Edit deployment configuration
kubectl edit deployment <deployment-name>
# Purpose: Modify deployment settings in real-time
```
