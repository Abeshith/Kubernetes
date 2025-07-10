# Kind Cluster Demo

## What is KIND?

**KIND** stands for **Kubernetes IN Docker** - a tool for running local Kubernetes clusters using Docker containers.

### Primary Use Cases:
- **Testing**: Local Kubernetes testing environment
- **CI/CD**: Automated testing pipelines
- **Local Development**: Developer workstation clusters
- **Education**: Learning Kubernetes without cloud costs

**Key Concept**: KIND uses Docker containers to run Kubernetes control planes and nodes, allowing you to run multiple K8s clusters locally for different teams or testing scenarios.

## Architecture

### Single Cluster (Basic):
```
+---------+
| KIND    |
| Cluster |
+---------+
   |
[ Docker container with master/control-plane + nodes ]
```

### Multi-node Cluster:
```
+--------+     +--------+     +--------+
| Master | --> | Node 1 | --> | Node 2 |
+--------+     +--------+     +--------+
```

**Important**: These nodes are Docker containers, created by KIND, running Kubernetes components inside.

## Why Use KIND?

### Benefits:
- **Lightweight and Fast**: Runs inside Docker containers
- **No Cloud Dependency**: No need for cloud provisioning or VMs
- **Multi-cluster Management**: Easy to manage multiple clusters locally
- **Production Simulation**: Good for simulating production environments locally
- **Resource Efficient**: Uses fewer resources than full VMs

## Practical Use Cases

### Team Isolation:
```
[ Developer A ] → [ Cluster A ]
[ Developer B ] → [ Cluster B ]
```

**Scenario**: Each developer/team can spin up their own local KIND cluster for:
- Isolated development environments
- Different feature testing
- Version compatibility testing
- Team-specific configurations

## Hands-On Implementation

### Create Single Node Cluster

```bash
# Create a basic single-node cluster
kind create cluster --name abhi-demo
# Purpose: Create simple cluster for basic testing and development
```

**This creates**:
- 1 master node
- Cluster context: `kind-abhi-demo`
- Ready-to-use Kubernetes environment

### Create Multi-node Cluster

#### Step 1: Create Configuration File

```yaml
# multi-node-cluster.yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
  - role: control-plane
  - role: worker
  - role: worker
```

#### Step 2: Create Multi-node Cluster

```bash
# Create multi-node cluster using configuration
kind create cluster --name multi-cluster --config multi-node-cluster.yaml
# Purpose: Create production-like environment with multiple worker nodes
```

**This configures**:
- 1 master (control-plane)
- 2 worker nodes
- Load balancing across workers

## kubeconfig Management

### View Current Configuration

```bash
# View current kubeconfig settings
kubectl config view
# Purpose: See all available clusters and contexts
```

### Switch Between Clusters

```bash
# Switch to single-node cluster
kubectl config use-context kind-abhi-demo
# Purpose: Set kubectl to work with specific KIND cluster

# Switch to multi-node cluster
kubectl config use-context kind-multi-cluster
# Purpose: Change kubectl context to different cluster
```

### Add New Context (if not auto-added)

```bash
# Manually add new context
kubectl config set-context kind-new --cluster=kind-new --user=kind-user
# Purpose: Create custom context for KIND cluster
```

## Verification Commands

### Check Cluster Nodes

```bash
# View all nodes in current cluster
kubectl get nodes
# Purpose: Verify cluster setup and node status
```

**Expected Output** (Multi-node):
- 1 master node
- 2 worker nodes
- All in "Ready" status

### Check Docker Containers

```bash
# View KIND containers
docker ps
# Purpose: See all KIND containers representing cluster nodes
```

**Important Note**: "Docker ps → shows all KIND containers = nodes of the cluster."

## Complete Workflow Example

```bash
# Step 1: Create multi-node cluster
kind create cluster --name demo-cluster --config multi-node-cluster.yaml

# Step 2: Verify cluster creation
kubectl get nodes

# Step 3: Check Docker containers
docker ps

# Step 4: Deploy test application
kubectl create deployment nginx --image=nginx:latest

# Step 5: Check deployment across nodes
kubectl get pods -o wide

# Step 6: Clean up when done
kind delete cluster --name demo-cluster
```

## Cluster Management Commands

```bash
# List all KIND clusters
kind get clusters
# Purpose: See all created KIND clusters

# Delete specific cluster
kind delete cluster --name <cluster-name>
# Purpose: Remove cluster and free resources

# Load Docker image into cluster
kind load docker-image <image-name> --name <cluster-name>
# Purpose: Use local Docker images in KIND cluster

# Get cluster info
kubectl cluster-info --context kind-<cluster-name>
# Purpose: View cluster endpoints and services
```

## Advanced Configuration Options

### Custom Port Mappings

```yaml
# kind-config-with-ports.yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
  extraPortMappings:
  - containerPort: 30000
    hostPort: 30000
    protocol: TCP
- role: worker
- role: worker
```

### Resource Limits

```yaml
# kind-config-resources.yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
  image: kindest/node:v1.28.0
- role: worker
  image: kindest/node:v1.28.0
```

## Best Practices

1. **Cluster Naming**: Use descriptive names for easy identification
2. **Resource Monitoring**: Monitor Docker resource usage
3. **Cleanup**: Delete unused clusters to free resources
4. **Configuration Files**: Store cluster configs in version control
5. **Testing**: Use separate clusters for different test scenarios

## Troubleshooting

```bash
# Check KIND cluster status
kind get clusters

# View cluster logs
docker logs <kind-container-name>

# Reset cluster if issues occur
kind delete cluster --name <cluster-name>
kind create cluster --name <cluster-name>

# Check Kubernetes version
kubectl version --short
```
