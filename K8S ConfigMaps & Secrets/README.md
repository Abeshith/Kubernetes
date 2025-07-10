# K8S ConfigMaps & Secrets

## What is ConfigMap?

ConfigMap is used to pass **non-sensitive configuration data** like database ports, application settings, or environment-specific configs into pods. It separates configuration from application code, making applications more portable and easier to manage.

## Why Use ConfigMaps?

- **Configuration Management**: Store non-sensitive config data separately from application code
- **Environment Flexibility**: Different configs for dev, staging, production
- **Dynamic Updates**: Update configuration without rebuilding container images
- **Decoupling**: Keep configuration separate from application logic

## What are Secrets?

Secrets are used to store and manage **sensitive information** like passwords, API keys, certificates, and tokens. They are base64-encoded and provide a more secure way to handle sensitive data compared to plain text.

## Why Use Secrets?

- **Security**: Store sensitive data like passwords and API keys securely
- **Base64 Encoding**: Data is encoded (not encrypted) for basic obfuscation
- **Access Control**: Can be restricted using RBAC policies
- **Separate Sensitive Data**: Keep secrets separate from application code

## Step 1: Create & Apply ConfigMap

```yaml
# config.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: demo-cm
data:
  db-port: "3306"
```

```bash
# Create ConfigMap from YAML file
kubectl apply -f config.yaml
# Purpose: Deploy ConfigMap to store non-sensitive configuration

# List all ConfigMaps
kubectl get cm
# Purpose: View all ConfigMaps in the current namespace

# Get detailed ConfigMap information
kubectl describe cm demo-cm
# Purpose: Show details and data stored in the ConfigMap
```

## Step 2: Create Deployment using ConfigMap (Environment Variables)

```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: demo-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: demo-app
  template:
    metadata:
      labels:
        app: demo-app
    spec:
      containers:
      - name: demo-container
        image: nginx:latest
        env:
        - name: DB_PORT
          valueFrom:
            configMapKeyRef:
              name: demo-cm
              key: db-port
```

```bash
# Apply deployment configuration
kubectl apply -f deployment.yaml
# Purpose: Create deployment that uses ConfigMap as environment variables

# Check pods with custom columns
kubectl get pods -o custom-columns
# Purpose: View pod details in custom format

# Access pod shell to check environment variables
kubectl exec -it <pod-name> -- /bin/bash
# Purpose: Get interactive shell access to verify ConfigMap data

# Check for database environment variables
env | grep db
# Purpose: Verify if ConfigMap data is available as environment variables
```

## Step 3: Use ConfigMap with VolumeMount (Alternative Method)

```yaml
# deployment.yaml (Updated for VolumeMount)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: demo-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: demo-app
  template:
    metadata:
      labels:
        app: demo-app
    spec:
      containers:
      - name: demo-container
        image: nginx:latest
        volumeMounts:
        - name: db-connection
          mountPath: /opt
      volumes:
      - name: db-connection
        configMap:
          name: demo-cm
```

```bash
# Apply updated deployment
kubectl apply -f deployment.yaml
# Purpose: Use ConfigMap as mounted files instead of environment variables
```

## Step 4: Verify ConfigMap from VolumeMount

```bash
# List files in mounted directory
kubectl exec -it <pod-name> -- ls /opt
# Purpose: Check if ConfigMap data is mounted as files

# Read ConfigMap data from mounted file
kubectl exec -it <pod-name> -- cat /opt/db-port
# Purpose: Verify ConfigMap value is accessible as file content
```

## Step 5: Modify ConfigMap

```bash
# Edit ConfigMap file (change db-port to "3307")
vim config.yaml

# Apply updated ConfigMap
kubectl apply -f config.yaml
# Purpose: Update ConfigMap with new configuration values

# Verify updated ConfigMap in pod
kubectl exec -it <pod-name> -- cat /opt/db-port
# Purpose: Confirm updated ConfigMap values are reflected in the pod
```

## Step 6: Create & Use Secrets

### Method 1: Imperative Secret Creation

```bash
# Create secret from command line
kubectl create secret generic test-secret --from-literal=db-port="3306"
# Purpose: Create secret with sensitive data using imperative command
```

### Method 2: Declarative Secret Creation

```yaml
# secrets.yaml
apiVersion: v1
kind: Secret
metadata:
  name: test-secret
type: Opaque
data:
  db-port: MzMwNg==  # base64 encoded value of "3306"
```

```bash
# Apply secret configuration
kubectl apply -f secrets.yaml
# Purpose: Create secret using YAML configuration file

# List all secrets
kubectl get secrets
# Purpose: View all secrets in the current namespace

# Get detailed secret information
kubectl describe secrets test-secret
# Purpose: Show secret metadata (data values are hidden for security)

# Verify secret data in pod (using same VolumeMount approach)
kubectl exec -it <pod-name> -- cat /opt/db-port
# Purpose: Access secret data mounted as file in the pod
```

## Key Differences

| **ConfigMap** | **Secret** |
|---------------|------------|
| Non-sensitive data | Sensitive data |
| Plain text storage | Base64 encoded |
| Application config | Passwords, API keys |
| Visible in describe | Hidden in describe |

## Important Notes

1. **VolumeMount vs Environment**: Use VolumeMount when config files change frequently
2. **Base64 Encoding**: Secrets are base64-encoded, not encrypted
3. **Dynamic Updates**: Changes to mounted ConfigMaps/Secrets reflect in pods automatically
4. **Security**: Always use Secrets for sensitive data, never ConfigMaps
