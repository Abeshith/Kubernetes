# K8S Helm Charts - E-Commerce Application

This repository contains Helm charts for deploying an e-commerce application on Kubernetes.

## Overview

The e-commerce application consists of multiple microservices packaged as Helm charts:
- **Payments Service**: Handles payment processing
- **Shipping Service**: Manages shipping and logistics

## Architecture

### Kubernetes Components
- **Pods**: Basic deployable units containing application containers
- **Services**: Network abstraction for accessing pods
- **Deployments**: Manage pod replicas and rolling updates
- **Ingress**: HTTP/HTTPS routing to services
- **ConfigMaps**: Configuration data storage
- **Secrets**: Sensitive data storage
- **PersistentVolumes**: Storage abstraction

### Helm Chart Structure
Each service chart includes:
- `Chart.yaml`: Chart metadata and version information
- `values.yaml`: Default configuration values
- `templates/`: Kubernetes manifest templates
  - `deployment.yaml`: Application deployment configuration
  - `service.yaml`: Service exposure configuration
  - `ingress.yaml`: Ingress routing rules
  - `hpa.yaml`: Horizontal Pod Autoscaler configuration
  - `serviceaccount.yaml`: Service account for pods
  - `_helpers.tpl`: Template helper functions
  - `NOTES.txt`: Post-installation notes
  - `tests/`: Chart testing resources

## Services

### Payments Service
Handles payment processing with the following features:
- Payment validation
- Transaction processing
- Payment gateway integration
- Secure payment data handling

### Shipping Service
Manages shipping operations including:
- Order fulfillment
- Shipping calculations
- Tracking information
- Delivery notifications

## Deployment

### Prerequisites
- Kubernetes cluster (v1.20+)
- Helm 3.x installed
- kubectl configured for your cluster

### Installation

1. **Add the repository** (if using a Helm repository):
```bash
helm repo add ecommerce ./E-Commerce
helm repo update
```

2. **Install the payments service**:
```bash
helm install payments ./E-Commerce/payments
```

3. **Install the shipping service**:
```bash
helm install shipping ./E-Commerce/shipping
```

### Custom Configuration

You can customize the deployment by creating a custom values file:

```yaml
# custom-values.yaml
image:
  repository: myregistry/payments
  tag: "1.2.0"
  
resources:
  requests:
    memory: "256Mi"
    cpu: "100m"
  limits:
    memory: "512Mi"
    cpu: "200m"

ingress:
  enabled: true
  host: payments.example.com
```

Then deploy with custom values:
```bash
helm install payments ./E-Commerce/payments -f custom-values.yaml
```

## Configuration

### Common Configuration Options

#### Image Configuration
```yaml
image:
  repository: <container-registry>/service-name
  tag: latest
  pullPolicy: IfNotPresent
```

#### Resource Management
```yaml
resources:
  requests:
    memory: "128Mi"
    cpu: "50m"
  limits:
    memory: "256Mi"
    cpu: "100m"
```

#### Autoscaling
```yaml
autoscaling:
  enabled: true
  minReplicas: 2
  maxReplicas: 10
  targetCPUUtilizationPercentage: 80
```

#### Ingress Configuration
```yaml
ingress:
  enabled: true
  className: nginx
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt-prod
  hosts:
    - host: api.example.com
      paths:
        - path: /
          pathType: Prefix
  tls:
    - secretName: api-tls
      hosts:
        - api.example.com
```

## Monitoring and Observability

### Health Checks
Each service includes:
- Liveness probes
- Readiness probes
- Startup probes

### Metrics
Services expose metrics for monitoring:
- Prometheus-compatible metrics endpoint
- Custom business metrics
- Performance indicators

## Security

### Service Accounts
Each service runs with its own service account with minimal required permissions.

### Network Policies
Implement network policies to control traffic between services:
- Ingress traffic rules
- Egress traffic rules
- Service-to-service communication

### Secrets Management
- Database credentials
- API keys
- TLS certificates
- Payment gateway secrets

## Testing

### Chart Testing
Run chart tests to verify deployments:
```bash
helm test payments
helm test shipping
```

### Integration Testing
Test service connectivity and functionality after deployment.

## Troubleshooting

### Common Issues

1. **Pod not starting**:
   - Check resource limits
   - Verify image availability
   - Review configuration

2. **Service not accessible**:
   - Verify service configuration
   - Check ingress rules
   - Validate network policies

3. **Persistent storage issues**:
   - Check PVC status
   - Verify storage class
   - Review volume mounts

### Useful Commands

```bash
# Check pod status
kubectl get pods -l app=payments

# View pod logs
kubectl logs -f deployment/payments

# Describe service
kubectl describe svc payments

# Check ingress
kubectl get ingress

# View events
kubectl get events --sort-by=.metadata.creationTimestamp
```

## Maintenance

### Upgrades
```bash
# Upgrade a release
helm upgrade payments ./E-Commerce/payments

# Rollback if needed
helm rollback payments 1
```

### Backup
- Back up persistent volumes
- Export configurations
- Document custom modifications

## Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Test thoroughly
5. Submit a pull request

## License

This project is licensed under the MIT License - see the LICENSE file for details.

## Support

For support and questions:
- Create an issue in the repository
- Review the troubleshooting section
- Check the Helm documentation

---

**Note**: This is a development/demo setup. For production deployments, ensure proper security hardening, monitoring, and backup procedures are in place.