# Node Level TroubleShooting

## Overview

Node-level troubleshooting in Kubernetes involves controlling where pods are scheduled and ensuring they run on appropriate nodes. This includes using Node Selector, Node Affinity, and Taints & Tolerations to manage pod placement.

## 1. Node Selector

Node selector is the simplest form of assigning pods to specific nodes using labels.

### Purpose:
To schedule a pod on a node that matches a specific label.

### Steps:

#### Step 1: Label the Node
```bash
# Add label to a specific node
kubectl label node <node-name> <key>=<value>
# Purpose: Tag node with specific characteristics for pod scheduling

# Example: Label node for SSD storage
kubectl label node worker-01 disktype=ssd
```

#### Step 2: Edit Pod/Deployment YAML
```yaml
# node-selector.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ssd-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: ssd-app
  template:
    metadata:
      labels:
        app: ssd-app
    spec:
      nodeSelector:
        disktype: ssd  # Must match node label
      containers:
      - name: app
        image: nginx:latest
```

#### Step 3: Apply Configuration
```bash
# Apply the deployment
kubectl apply -f node-selector.yaml
# Purpose: Deploy pod with node selector requirements
```

#### Step 4: Verify Pod Placement
```bash
# Check where the pod is scheduled
kubectl describe pod <pod-name>
# Purpose: Verify pod is scheduled on correct node

# Check pod placement
kubectl get pods -o wide
# Purpose: See which node the pod is running on
```

### Common Error:
If nodes don't have the required label:
```
"0/4 nodes are available: 4 node(s) didn't match node selector"
```

**Explanation**: If you have 4 nodes and 3 don't have the label, only 1 node will be eligible for scheduling.

## 2. Node Affinity

More advanced and expressive than nodeSelector. It allows rules for scheduling based on labels with required or preferred constraints.

### Purpose:
Control where pods are scheduled using required or preferred rules with flexible label matching.

### Types of Affinity:

1. **requiredDuringSchedulingIgnoredDuringExecution**: Must match to be scheduled
2. **preferredDuringSchedulingIgnoredDuringExecution**: Best-effort preference

### Example Configuration:

```yaml
# affinity-required.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: affinity-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: affinity-app
  template:
    metadata:
      labels:
        app: affinity-app
    spec:
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: disktype
                operator: In
                values:
                - ssd
      containers:
      - name: app
        image: nginx:latest
```

**Explanation**: The pod **must** be scheduled on nodes with `disktype=ssd`.

### Preferred Affinity Example:

```yaml
# affinity-preferred.yaml
spec:
  affinity:
    nodeAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 100
        preference:
          matchExpressions:
          - key: disktype
            operator: In
            values:
            - ssd
```

### Commands:
```bash
# Apply affinity configuration
kubectl apply -f affinity-required.yaml
# Purpose: Deploy with node affinity requirements

# Check scheduling decisions
kubectl describe pod <pod-name>
# Purpose: See affinity rules and scheduling results
```

### Use Cases:
- Group workloads on specific hardware (GPU nodes, SSD storage, ARM architecture)
- Combine multiple expressions with AND/OR logic
- Flexible scheduling based on node characteristics

## 3. Taints and Tolerations

Taints control which pods should **NOT** be scheduled on nodes unless they explicitly tolerate the taint.

### Purpose:
Prevent pods from being scheduled on unsuitable nodes unless they tolerate the taint.

### Taint a Node:

```bash
# Add taint to prevent scheduling
kubectl taint nodes <node-name> <key>=<value>:<effect>
# Purpose: Mark node as unsuitable for general workloads

# Example: Taint node for high-priority workloads only
kubectl taint node worker-01 priority=high:NoSchedule
```

### Taint Effects:

1. **NoSchedule**: Don't schedule pods unless tolerated
2. **PreferNoSchedule**: Avoid scheduling if possible
3. **NoExecute**: Evict existing pods unless tolerated

### Add Toleration to Pod:

```yaml
# toleration.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: high-priority-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: high-priority-app
  template:
    metadata:
      labels:
        app: high-priority-app
    spec:
      tolerations:
      - key: "priority"
        operator: "Equal"
        value: "high"
        effect: "NoSchedule"
      containers:
      - name: app
        image: nginx:latest
```

### Commands:

```bash
# Apply toleration configuration
kubectl apply -f toleration.yaml
# Purpose: Deploy pod that can tolerate node taints

# View node taints
kubectl describe node <node-name>
# Purpose: See what taints are applied to a node

# Remove taint from node
kubectl taint nodes <node-name> <key>:<effect>-
# Purpose: Remove specific taint from node
```

### Key Concepts:

- **Taint**: Prevents scheduling on a node (like a label or attribute)
- **Toleration**: Allows pods to be placed even on tainted nodes
- **Use Case**: If 8 nodes are tainted, only high-priority pods with toleration should go there

### Common Example:
Control-plane nodes are often tainted to prevent regular workloads from being scheduled unless tolerated.

## Troubleshooting Commands

```bash
# Check node labels
kubectl get nodes --show-labels
# Purpose: View all labels on nodes

# Describe node for detailed information
kubectl describe node <node-name>
# Purpose: See node capacity, taints, and allocated resources

# Check pod scheduling events
kubectl get events --sort-by=.metadata.creationTimestamp
# Purpose: See scheduling decisions and failures

# View pods with node placement
kubectl get pods -o wide
# Purpose: See which node each pod is running on

# Check pending pods
kubectl get pods --field-selector=status.phase=Pending
# Purpose: Find pods that failed to schedule
```

## Summary Table

| **Feature** | **Purpose** | **Use Case** | **Complexity** |
|-------------|-------------|--------------|----------------|
| **Node Selector** | Basic filtering using labels | Assign pod to a specific node | Easy |
| **Node Affinity** | Advanced rules using label expressions | Require or prefer nodes based on hardware attributes | Medium |
| **Taints & Tolerations** | Prevent pods from landing on certain nodes | Isolate workloads like system or GPU-only tasks | Medium–High |

## Best Practices

1. **Label Strategy**: Use consistent labeling convention for nodes
2. **Combine Methods**: Use node affinity with tolerations for complex scenarios
3. **Resource Planning**: Consider node capacity when using selectors
4. **Documentation**: Document taint purposes and toleration requirements
5. **Testing**: Test scheduling rules in development before production deployment
