# StatefulSet & Persistent Volume Claim

## 1. What is a StatefulSet?

A **StatefulSet** is a Kubernetes controller used to manage stateful applications where pod identity and data persistence matter.

### Key Characteristics:
- **Stable DNS Names**: Each pod gets predictable names (pod-0, pod-1, pod-2)
- **Stable Storage**: Persistent volume per pod that survives pod restarts
- **Ordered Operations**: Pods are created, updated, and deleted in sequence
- **Suitable for**: Databases, message queues, and other stateful applications

**Key Point**: StatefulSets manage apps where the state matters. Unlike stateless apps, they need their identity and data to persist (like databases).

## 2. StatefulSet vs Stateless Deployment

### Stateless Deployment:
- **Pod Identity**: Doesn't matter - pods are interchangeable
- **Restart Behavior**: Can kill and recreate pods randomly
- **Storage**: Usually uses shared storage or no persistent storage
- **Use Cases**: Web servers, API services, microservices

### StatefulSet:
- **Pod Identity**: Matters - each pod has unique identity (pod-0, pod-1, etc.)
- **Restart Behavior**: Cannot simply restart pods in any order
- **Storage**: Each pod gets its own persistent volume
- **Use Cases**: Databases, distributed systems requiring stable identity

```
StatefulSet Architecture:
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│   pod-0     │    │   pod-1     │    │   pod-2     │
│             │    │             │    │             │
├─────────────┤    ├─────────────┤    ├─────────────┤
│ PVC: data-0 │    │ PVC: data-1 │    │ PVC: data-2 │
└─────────────┘    └─────────────┘    └─────────────┘
```

## 3. Volume Binding in StatefulSets

### Ordered Creation:
**"Even if we have 3 replicas, the 2nd is created only when the 1st is completed."**

- StatefulSet ensures **ordered creation** and deletion
- Volumes are bound in sequence, one per pod
- Each pod gets its own unique Persistent Volume Claim (PVC)

### Scaling Behavior:
When you scale up, each new pod gets:
1. A unique Persistent Volume Claim (PVC)
2. Based on the `volumeClaimTemplate`
3. Stable network identity
4. Persistent storage that survives pod restarts

## 4. How PVC Works (Storage Flow)

```
StatefulSet
    ↓
PVC (PersistentVolumeClaim)
    ↓  
SC (StorageClass)
    ↓
Provisioner (e.g., EBS, GCE, Azure Disk)
    ↓
PV (Persistent Volume)
```

### Flow Explanation:
1. **PVCs**: Request storage from StorageClass
2. **StorageClass**: Uses a provisioner (like EBS or CSI driver) to create volumes dynamically
3. **Provisioner**: Creates actual storage volumes in cloud provider
4. **PV**: Physical storage that gets bound to PVC

## 5. VolumeClaimTemplate

### Why VolumeClaimTemplate?
- **Cannot manually create PVCs** for each pod in a StatefulSet
- **Template-based creation**: Define once, Kubernetes handles unique PVC creation per pod
- **Automatic naming**: Creates PVCs like `data-pod-0`, `data-pod-1`, etc.

### Template Benefits:
- Consistent storage configuration across all pods
- Automatic PVC lifecycle management
- Simplified scaling operations

## 6. StorageClass & Provisioning

### StorageClass Purpose:
A **StorageClass** defines how storage is provisioned and what type of storage to use.

### Examples:
- `ebs` (AWS Elastic Block Store)
- `standard` (Default storage)
- `ssd` (SSD-based storage)

### Provisioning Flow:
```
volumeClaimTemplate → PVC → StorageClass → Provisioner → PV
```

### Important Note:
**"StorageClass charge — if I have 1 Disk for Read & Write + Another for Backup → Waste unless both pods are running."**

**Conclusion**: Use storage efficiently, avoid unused persistent volumes that incur costs.

## Hands-On Implementation

### Step 1: Define StorageClass (AWS EBS Example)

```yaml
# storageclass.yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: ebs
provisioner: kubernetes.io/aws-ebs
parameters:
  type: gp2
  fsType: ext4
allowVolumeExpansion: true
```

### Step 2: Create StatefulSet with VolumeClaimTemplate

```yaml
# statefulset.yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  selector:
    matchLabels:
      app: nginx
  serviceName: "nginx"
  replicas: 3
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
        volumeMounts:
        - name: www
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
  - metadata:
      name: www
    spec:
      accessModes: [ "ReadWriteOnce" ]
      storageClassName: "ebs"
      resources:
        requests:
          storage: 1Gi
```

### Step 3: Create Headless Service

```yaml
# service.yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  ports:
  - port: 80
    name: web
  clusterIP: None
  selector:
    app: nginx
```

### Step 4: Apply Configuration

```bash
# Apply StorageClass
kubectl apply -f storageclass.yaml
# Purpose: Define how storage should be provisioned

# Apply Service (must be created before StatefulSet)
kubectl apply -f service.yaml
# Purpose: Create headless service for stable network identity

# Apply StatefulSet
kubectl apply -f statefulset.yaml
# Purpose: Create stateful application with persistent storage
```

## What Happens When You Apply?

### Automatic Resource Creation:
1. **Pods created**: `web-0`, `web-1`, `web-2` (in order)
2. **PVCs auto-created**: `www-web-0`, `www-web-1`, `www-web-2`
3. **Each PVC binds** to a separate volume from StorageClass `ebs`
4. **Data persists** even if pods are restarted

### Pod Creation Sequence:
```
web-0 (created first) → www-web-0 (PVC bound)
     ↓ (only after web-0 is ready)
web-1 (created second) → www-web-1 (PVC bound)
     ↓ (only after web-1 is ready)
web-2 (created third) → www-web-2 (PVC bound)
```

## Monitoring Commands

```bash
# Check StatefulSet status
kubectl get statefulsets
# Purpose: View StatefulSet replicas and ready status

# Check pods in order
kubectl get pods -l app=nginx
# Purpose: See pod creation order and status

# Check PVCs created by StatefulSet
kubectl get pvc
# Purpose: View auto-created PVCs and their binding status

# Check PVs bound to PVCs
kubectl get pv
# Purpose: See actual persistent volumes created

# Describe StatefulSet for detailed info
kubectl describe statefulset web
# Purpose: View StatefulSet configuration and events

# Check pod storage mounts
kubectl describe pod web-0
# Purpose: Verify volume mounts and PVC binding
```

## Scaling Operations

```bash
# Scale up StatefulSet
kubectl scale statefulset web --replicas=5
# Purpose: Add more pods with persistent storage

# Scale down StatefulSet
kubectl scale statefulset web --replicas=1
# Purpose: Remove pods (PVCs remain for data safety)

# Watch scaling process
kubectl get pods -w
# Purpose: Monitor ordered scaling operations
```

## Key Benefits

1. **Data Persistence**: Data survives pod restarts and rescheduling
2. **Stable Identity**: Each pod maintains consistent DNS name
3. **Ordered Operations**: Predictable startup and shutdown sequence
4. **Storage Isolation**: Each pod gets dedicated storage
5. **Stateful Applications**: Perfect for databases and distributed systems

## Important Considerations

1. **Storage Costs**: Each pod gets its own volume - monitor usage
2. **Backup Strategy**: Plan for persistent volume backups
3. **Storage Class**: Choose appropriate storage type for workload
4. **Scaling**: Understand that scaling creates/retains storage
5. **Deletion**: PVCs persist even after StatefulSet deletion (by design)
