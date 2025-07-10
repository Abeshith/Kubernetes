# NetworkPolicy

## What is a Network Policy?

A **Network Policy** is a Kubernetes resource that controls **ingress** and **egress** traffic to/from pods.

### Key Functions:
- **Secure communication** between apps inside a cluster
- **Prevents unauthorized access** to sensitive services (like databases)
- **Controls pod-to-pod communication** for security purposes

## Why is Network Policy Needed?

### Security Question:
**"How do you secure the DB from other namespaces?"**

### Problem Scenario:
- Pods from other namespaces shouldn't access your DB pod
- **Example**: If a hacker gains access to one pod, they shouldn't be able to reach your database
- **Without Network Policy**: Anyone in the same cluster can access the DB — huge security risk

### Purpose:
**"To restrict pod-to-pod communication inside the cluster for security."**

## Use Case Example

**Objective**: Only allow the `payments-app` to talk to the DB.

```
[ DB Pod ] ← only accessible by ← [ payments-app Pod ]
```

**Result**: 
- ✅ **With Network Policy**: Only explicitly allowed pods (like payments-app) can talk to DB
- ❌ **Without Network Policy**: Anyone in the same cluster can access the DB — huge risk

## How NetworkPolicy Works

### Two Main Types of Rules:

#### 1. **Ingress**
- **Controls**: Incoming traffic **TO** the pod (who can talk to it)
- **Example**: "Only payments pod can access DB"

#### 2. **Egress**
- **Controls**: Outgoing traffic **FROM** the pod (where it can go)
- **Example**: "Pod cannot access google.com or external websites"

### Traffic Flow Diagram:
```
           +----------------+
           |  DB Pod        |
           +----------------+
              ^         ^
              |         |
Ingress ----> |         | <---- Egress
       [Payments Pod]     [Other Pods (blocked)]
```

## Hands-On Implementation

### Step 1: Create Secure Environment

```bash
# Create a namespace for secure workloads
kubectl create namespace secure-ns
# Purpose: Isolate workloads in dedicated namespace for testing
```

### Step 2: Deploy Database in Secure Namespace

```yaml
# db.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: db-deployment
  namespace: secure-ns
spec:
  replicas: 1
  selector:
    matchLabels:
      role: db
  template:
    metadata:
      labels:
        role: db
    spec:
      containers:
      - name: db
        image: nginx:latest
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: db-service
  namespace: secure-ns
spec:
  selector:
    role: db
  ports:
  - port: 80
    targetPort: 80
```

```bash
# Apply database deployment in secure namespace
kubectl apply -f db.yaml -n secure-ns
# Purpose: Create database pod that needs protection
```

### Step 3: Create Hacker Pod (Different Namespace)

```yaml
# hacker.yaml
apiVersion: v1
kind: Pod
metadata:
  name: hacker-pod
  namespace: default  # Different namespace
spec:
  containers:
  - name: hacker
    image: curlimages/curl:latest
    command: ["sleep", "3600"]
```

```bash
# Create and apply hacker pod from different namespace
kubectl apply -f hacker.yaml
# Purpose: Simulate unauthorized access attempt
```

### Step 4: Test Access Before Network Policy

```bash
# Try accessing DB from hacker pod (should work without Network Policy)
kubectl exec -it hacker-pod -- curl db-service.secure-ns.svc.cluster.local
# Purpose: Demonstrate vulnerability without Network Policy
```

### Step 5: Create Network Policy

```yaml
# network-policy.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-payment-to-db
  namespace: secure-ns
spec:
  podSelector:
    matchLabels:
      role: db              # Apply to pods with role=db
  policyTypes:
  - Ingress                 # Control incoming traffic
  ingress:
  - from:
    - podSelector:
        matchLabels:
          role: payments-app  # Only allow pods with role=payments-app
```

```bash
# Apply Network Policy
kubectl apply -f network-policy.yaml
# Purpose: Restrict access to DB pod to only payments-app
```

### Step 6: Deploy Authorized Payments App

```yaml
# payments-app.yaml
apiVersion: v1
kind: Pod
metadata:
  name: payments-pod
  namespace: secure-ns
  labels:
    role: payments-app      # Required label for access
spec:
  containers:
  - name: payments
    image: curlimages/curl:latest
    command: ["sleep", "3600"]
```

```bash
# Deploy authorized payments app
kubectl apply -f payments-app.yaml
# Purpose: Create authorized pod that can access DB
```

### Step 7: Test Network Policy

```bash
# Test access from hacker pod (should FAIL)
kubectl exec -it hacker-pod -- curl db-service.secure-ns.svc.cluster.local
# Expected: Connection refused or timeout

# Test access from payments pod (should SUCCEED)
kubectl exec -it payments-pod -n secure-ns -- curl db-service
# Expected: Successful connection
```

## Step-by-Step Summary

### Hands-On Workflow:

1. **Create** DB deployment in a namespace (`secure-ns`)
2. **Create** hacker pod outside the namespace
3. **Try to curl** or access the DB service (should work initially)
4. **Apply** network policy allowing only `payments-app`
5. **Result**: Hacker pod will be blocked; payments pod gets access

## Verification Commands

```bash
# Check Network Policy status
kubectl get networkpolicy -n secure-ns
# Purpose: View applied network policies

# Describe Network Policy for details
kubectl describe networkpolicy allow-payment-to-db -n secure-ns
# Purpose: See policy rules and affected pods

# Check pods with labels
kubectl get pods -n secure-ns --show-labels
# Purpose: Verify pod labels match policy selectors

# Test connectivity from different pods
kubectl exec -it <pod-name> -n <namespace> -- curl <service-name>
# Purpose: Verify network policy enforcement
```

## Key Security Benefits

1. **Namespace Isolation**: Prevents cross-namespace unauthorized access
2. **Principle of Least Privilege**: Only explicitly allowed traffic passes
3. **Database Protection**: Secures sensitive data stores
4. **Attack Containment**: Limits damage if one pod is compromised
5. **Compliance**: Meets security requirements for regulated environments

## Important Notes

- **Default Behavior**: Without Network Policy, all pods can communicate
- **CNI Requirement**: Network Policy requires compatible CNI plugin (Calico, Weave, etc.)
- **Namespace Scope**: Policies apply within specific namespaces
- **Label-Based**: Uses pod labels for traffic control
- **Ingress/Egress**: Control both incoming and outgoing traffic
