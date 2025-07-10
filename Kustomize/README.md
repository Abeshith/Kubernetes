# Kustomize

## What is Kustomize?

**Kustomize** is a Kubernetes-native configuration management tool that lets you customize raw, template-free YAML files for different environments like dev, stage, or prod, without duplication.

**Key Concept**: "Kustomize modifies YAMLs without using templates or copying files."

## Use Case

You have one base YAML for your app, but need different values for:

- **Dev environment**: 1 replica
- **Stage environment**: 3 replicas  
- **Prod environment**: 5 replicas

Instead of duplicating YAML files, Kustomize patches the base configuration with environment-specific changes.

## Kustomize Structure

```
kustomization.yaml (base)
├── deployment.yaml
├── service.yaml

overlays/
├── dev/
│   ├── kustomization.yaml
│   └── replica.yaml
├── stage/
│   ├── kustomization.yaml
│   └── replica.yaml
```

## Base Configuration (Common YAML)

### Base Kustomization File

```yaml
# base/kustomization.yaml
resources:
  - deployment.yaml
  - service.yaml
```

### Base Deployment

```yaml
# base/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-name
  labels:
    app: my-app
spec:
  replicas: 1  # Default replica count
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: app
        image: nginx:latest
        ports:
        - containerPort: 80
```

### Base Service

```yaml
# base/service.yaml
apiVersion: v1
kind: Service
metadata:
  name: app-service
  labels:
    app: my-app
spec:
  selector:
    app: my-app
  ports:
  - port: 80
    targetPort: 80
  type: ClusterIP
```

## Overlays (Environment-Specific)

### Dev Overlay

#### Dev Kustomization
```yaml
# overlays/dev/kustomization.yaml
resources:
  - ../../base
patchesStrategicMerge:
  - replica.yaml
```

#### Dev Replica Patch
```yaml
# overlays/dev/replica.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-name
spec:
  replicas: 1  # Dev environment: single replica
```

### Stage Overlay

#### Stage Kustomization
```yaml
# overlays/stage/kustomization.yaml
resources:
  - ../../base
patchesStrategicMerge:
  - replica.yaml
```

#### Stage Replica Patch
```yaml
# overlays/stage/replica.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-name
spec:
  replicas: 3  # Stage environment: three replicas
```

## Benefits

### Key Advantages:
- **Avoids Duplication**: No copying YAML across environments
- **Template-Free Approach**: No complex templating like Helm
- **Built-in to kubectl**: Available as `kubectl kustomize`
- **Selective Patching**: Only modify what changes between environments

**Core Philosophy**: "Use same base, only patch overlays with changes — like replicas, env, labels etc."

## Hands-On Implementation

### Apply Dev Environment

```bash
# Apply development configuration
kubectl apply -k overlays/dev
# Purpose: Deploy application with dev-specific settings (1 replica)
```

### Apply Stage Environment

```bash
# Apply staging configuration
kubectl apply -k overlays/stage
# Purpose: Deploy application with stage-specific settings (3 replicas)
```

### Preview Changes (Dry Run)

```bash
# Preview dev configuration without applying
kubectl kustomize overlays/dev
# Purpose: See the merged YAML before deployment

# Preview stage configuration
kubectl kustomize overlays/stage
# Purpose: Verify stage-specific patches
```

## Internal Working

### How Kustomize Works:
1. **Base**: Contains raw Kubernetes definitions
2. **Overlays**: Patch/merge the base with environment-specific changes
3. **Build Process**: Combines base + patches into final YAML
4. **Apply**: Deploys the merged configuration

### Supported Operations:
- **Patches**: Modify existing fields
- **Images**: Update container images
- **Namespace**: Set target namespace
- **NamePrefix**: Add prefixes to resource names
- **Labels**: Add/modify labels and selectors

## Advanced Examples

### Image Updates

```yaml
# overlays/prod/kustomization.yaml
resources:
  - ../../base
images:
- name: nginx
  newTag: 1.21.0
patchesStrategicMerge:
  - replica.yaml
```

### Namespace and Name Prefix

```yaml
# overlays/prod/kustomization.yaml
namespace: production
namePrefix: prod-
resources:
  - ../../base
patchesStrategicMerge:
  - replica.yaml
```

### Multiple Patches

```yaml
# overlays/prod/kustomization.yaml
resources:
  - ../../base
patchesStrategicMerge:
  - replica.yaml
  - resources.yaml
  - env-vars.yaml
```

## Complete Directory Structure Example

```
kustomize-demo/
├── base/
│   ├── kustomization.yaml
│   ├── deployment.yaml
│   └── service.yaml
└── overlays/
    ├── dev/
    │   ├── kustomization.yaml
    │   └── replica.yaml
    ├── stage/
    │   ├── kustomization.yaml
    │   └── replica.yaml
    └── prod/
        ├── kustomization.yaml
        ├── replica.yaml
        └── resources.yaml
```

## Kustomize Commands

```bash
# Build and view configuration
kubectl kustomize overlays/dev
# Purpose: Generate merged YAML without applying

# Apply specific overlay
kubectl apply -k overlays/stage
# Purpose: Deploy with environment-specific configuration

# Delete resources
kubectl delete -k overlays/dev
# Purpose: Remove resources deployed with specific overlay

# Validate kustomization
kubectl kustomize overlays/prod --dry-run=client
# Purpose: Check for configuration errors
```

## Comparison with Other Tools

| Feature | Kustomize | Helm | Plain YAML |
|---------|-----------|------|------------|
| **Templates** | No | Yes | No |
| **Learning Curve** | Low | Medium | Minimal |
| **File Duplication** | No | No | Yes |
| **Built-in kubectl** | Yes | No | Yes |
| **Complex Logic** | Limited | Full | None |

## Best Practices

1. **Keep Base Simple**: Base should contain common, unchanging configuration
2. **Small Patches**: Only patch what actually changes between environments
3. **Consistent Structure**: Use same overlay structure across environments
4. **Version Control**: Store all configurations in Git
5. **Validation**: Always preview with `kubectl kustomize` before applying

## Troubleshooting

```bash
# Check kustomization syntax
kubectl kustomize overlays/dev

# Validate patches apply correctly
kubectl apply -k overlays/dev --dry-run=client

# Debug patch issues
kubectl kustomize overlays/dev | kubectl apply -f - --dry-run=client -o yaml
```

## Common Use Cases

1. **Environment Promotion**: Dev → Stage → Prod with different configs
2. **Multi-tenant Deployments**: Same app, different namespaces/prefixes
3. **Feature Flags**: Enable/disable features per environment
4. **Resource Scaling**: Different replica counts per environment
5. **Image Versioning**: Different image tags per environment
