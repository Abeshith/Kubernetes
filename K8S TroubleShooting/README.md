# K8S TroubleShooting

## Overview

This section covers common Kubernetes troubleshooting scenarios and their solutions. Each topic includes detailed explanations, practical examples, and step-by-step resolution procedures.

## Troubleshooting Topics

### 1. [CrashLoopBackOff](./CrashLoopBackOff/)
- **Problem**: Pods keep crashing and restarting repeatedly
- **Common Causes**: Wrong commands, liveness probe failures, OOM errors, application bugs
- **Solutions**: Fix configurations, adjust resource limits, implement proper health checks

### 2. [ImagePullBackOff](./ImagePullBackOff/)
- **Problem**: Kubernetes cannot pull container images from registry
- **Common Causes**: Wrong image names, private registry authentication issues
- **Solutions**: Correct image names, configure docker registry secrets

### 3. [NetworkPolicy](./NetworkPolicy/)
- **Problem**: Securing pod-to-pod communication within clusters
- **Use Case**: Restrict database access to authorized applications only
- **Solutions**: Implement ingress/egress traffic controls using NetworkPolicy

### 4. [Node Level TroubleShooting](./Node%20Level%20TroubleShooting/)
- **Problem**: Controlling pod placement and scheduling on specific nodes
- **Topics Covered**: Node Selector, Node Affinity, Taints & Tolerations
- **Solutions**: Label-based scheduling, hardware-specific placement, workload isolation

### 5. [StatefulSet & Persistent Volume Claim](./StateFullSet%20&%20Persistent%20Volume%20Claim/)
- **Problem**: Managing stateful applications requiring persistent storage
- **Use Case**: Databases and applications needing stable identity and storage
- **Solutions**: StatefulSet configuration, VolumeClaimTemplates, StorageClass setup

## Quick Reference

| Issue Type | Key Symptoms | Primary Solution |
|------------|--------------|------------------|
| **CrashLoopBackOff** | Pod restart loops, exponential backoff | Fix app configuration, resource limits |
| **ImagePullBackOff** | ErrImagePull → ImagePullBackOff | Correct image name, add registry secrets |
| **Network Security** | Unauthorized pod access | Apply NetworkPolicy rules |
| **Pod Scheduling** | Pods on wrong nodes | Use selectors, affinity, taints |
| **Stateful Apps** | Data loss on restart | Implement StatefulSet with PVCs |

## General Troubleshooting Commands

```bash
# Check pod status and events
kubectl get pods
kubectl describe pod <pod-name>

# View cluster events
kubectl get events --sort-by=.metadata.creationTimestamp

# Check logs
kubectl logs <pod-name>
kubectl logs <pod-name> --previous

# Resource monitoring
kubectl top pods
kubectl top nodes
```

## Best Practices

1. **Proactive Monitoring**: Regularly check pod and node status
2. **Resource Planning**: Set appropriate CPU/memory limits
3. **Security First**: Implement NetworkPolicies for sensitive workloads
4. **Documentation**: Keep track of node labels and taints
5. **Testing**: Validate configurations in development before production
