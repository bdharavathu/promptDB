# PromptDB Helm Chart

This Helm chart deploys PromptDB on Kubernetes with version management and easy configuration.

## Prerequisites

- Kubernetes 1.19+
- Helm 3.0+
- PV provisioner support in the underlying infrastructure (or use hostPath for local development)

## Installing the Chart

### Quick Install (Development)

```bash
# From the webapp directory
helm install promptdb ./helm/promptdb

# Or specify namespace
helm install promptdb ./helm/promptdb -n promptdb --create-namespace
```

### Production Install with Custom Values

```bash
# Create a custom values file
cat > my-values.yaml <<EOF
replicaCount: 3

image:
  repository: your-registry/promptdb
  tag: "1.0.0"

secret:
  secretKey: "your-production-secret-key-here"

persistence:
  data:
    hostPath:
      enabled: false
      path: ""
    storageClass: "your-storage-class"

ingress:
  enabled: true
  className: "nginx"
  hosts:
    - host: promptdb.yourdomain.com
      paths:
        - path: /
          pathType: Prefix
  tls:
    - secretName: promptdb-tls
      hosts:
        - promptdb.yourdomain.com
EOF

# Install with custom values
helm install promptdb ./helm/promptdb -f my-values.yaml -n promptdb --create-namespace
```

## Configuration

The following table lists the configurable parameters of the PromptDB chart and their default values.

### Application Settings

| Parameter | Description | Default |
|-----------|-------------|---------|
| `replicaCount` | Number of replicas | `2` |
| `image.repository` | Image repository | `promptdb` |
| `image.tag` | Image tag | `latest` |
| `image.pullPolicy` | Image pull policy | `IfNotPresent` |
| `config.flaskEnv` | Flask environment | `production` |
| `config.port` | Application port | `5000` |
| `secret.secretKey` | Secret key for Flask | `change-me-in-production` |

### Service Settings

| Parameter | Description | Default |
|-----------|-------------|---------|
| `service.type` | Kubernetes service type | `ClusterIP` |
| `service.port` | Service port | `80` |
| `service.targetPort` | Container port | `5000` |

### Ingress Settings

| Parameter | Description | Default |
|-----------|-------------|---------|
| `ingress.enabled` | Enable ingress | `false` |
| `ingress.className` | Ingress class name | `nginx` |
| `ingress.hosts` | Ingress hosts | `[{host: promptdb.local, paths: [{path: /, pathType: Prefix}]}]` |
| `ingress.tls` | TLS configuration | `[]` |

### Persistence Settings

| Parameter | Description | Default |
|-----------|-------------|---------|
| `persistence.prompts.enabled` | Enable prompts PVC | `true` |
| `persistence.prompts.size` | Prompts PVC size | `500Mi` |
| `persistence.prompts.storageClass` | Storage class | `""` |
| `persistence.prompts.hostPath.enabled` | Use hostPath | `false` |
| `persistence.data.enabled` | Enable data PVC | `true` |
| `persistence.data.size` | Data PVC size | `1Gi` |
| `persistence.data.hostPath.enabled` | Use hostPath for data | `true` |
| `persistence.data.hostPath.path` | HostPath location | `/Users/bdharavathu/codespace/pvdata/promptdbdata` |

### Resource Settings

| Parameter | Description | Default |
|-----------|-------------|---------|
| `resources.limits.cpu` | CPU limit | `500m` |
| `resources.limits.memory` | Memory limit | `512Mi` |
| `resources.requests.cpu` | CPU request | `100m` |
| `resources.requests.memory` | Memory request | `128Mi` |

### Autoscaling Settings

| Parameter | Description | Default |
|-----------|-------------|---------|
| `autoscaling.enabled` | Enable HPA | `false` |
| `autoscaling.minReplicas` | Minimum replicas | `2` |
| `autoscaling.maxReplicas` | Maximum replicas | `10` |
| `autoscaling.targetCPUUtilizationPercentage` | Target CPU % | `80` |

## Upgrading the Chart

```bash
# Upgrade with default values
helm upgrade promptdb ./helm/promptdb -n promptdb

# Upgrade with custom values
helm upgrade promptdb ./helm/promptdb -f my-values.yaml -n promptdb

# Upgrade with specific version
helm upgrade promptdb ./helm/promptdb --set image.tag=1.0.1 -n promptdb
```

## Uninstalling the Chart

```bash
helm uninstall promptdb -n promptdb
```

## Examples

### Example 1: Local Development with HostPath

```yaml
# dev-values.yaml
replicaCount: 1

persistence:
  prompts:
    hostPath:
      enabled: true
      path: /path/to/your/prompts
  data:
    hostPath:
      enabled: true
      path: /path/to/data

resources:
  limits:
    cpu: 200m
    memory: 256Mi
```

```bash
helm install promptdb ./helm/promptdb -f dev-values.yaml
```

### Example 2: Production with LoadBalancer

```yaml
# prod-values.yaml
replicaCount: 3

image:
  repository: myregistry.io/promptdb
  tag: "1.0.0"

secret:
  secretKey: "super-secret-production-key"

service:
  type: LoadBalancer

persistence:
  prompts:
    hostPath:
      enabled: false
    storageClass: "fast-ssd"
  data:
    hostPath:
      enabled: false
    storageClass: "standard"

autoscaling:
  enabled: true
  minReplicas: 3
  maxReplicas: 10

resources:
  limits:
    cpu: 1000m
    memory: 1Gi
  requests:
    cpu: 500m
    memory: 512Mi
```

```bash
helm install promptdb ./helm/promptdb -f prod-values.yaml -n production --create-namespace
```

### Example 3: With Ingress and TLS

```yaml
# ingress-values.yaml
ingress:
  enabled: true
  className: "nginx"
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt-prod
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
  hosts:
    - host: promptdb.example.com
      paths:
        - path: /
          pathType: Prefix
  tls:
    - secretName: promptdb-tls
      hosts:
        - promptdb.example.com
```

```bash
helm install promptdb ./helm/promptdb -f ingress-values.yaml -n promptdb --create-namespace
```

## Helm Commands Cheat Sheet

```bash
# List all releases
helm list -n promptdb

# Get release status
helm status promptdb -n promptdb

# Get release values
helm get values promptdb -n promptdb

# Get all release information
helm get all promptdb -n promptdb

# Rollback to previous version
helm rollback promptdb -n promptdb

# Rollback to specific revision
helm rollback promptdb 1 -n promptdb

# Show history
helm history promptdb -n promptdb

# Dry run (test without installing)
helm install promptdb ./helm/promptdb --dry-run --debug -n promptdb

# Template (render templates locally)
helm template promptdb ./helm/promptdb

# Package the chart
helm package ./helm/promptdb

# Lint the chart
helm lint ./helm/promptdb
```

## Version Management

Helm provides built-in version management:

```bash
# Install version 1.0.0
helm install promptdb ./helm/promptdb --set image.tag=1.0.0 -n promptdb

# Upgrade to 1.0.1
helm upgrade promptdb ./helm/promptdb --set image.tag=1.0.1 -n promptdb

# Check history
helm history promptdb -n promptdb

# Rollback if needed
helm rollback promptdb 1 -n promptdb
```

## Troubleshooting

### Check Pod Status
```bash
kubectl get pods -n promptdb
kubectl describe pod <pod-name> -n promptdb
kubectl logs <pod-name> -n promptdb
```

### Check Service
```bash
kubectl get svc -n promptdb
kubectl describe svc promptdb -n promptdb
```

### Check PVC
```bash
kubectl get pvc -n promptdb
kubectl describe pvc promptdb-data-pvc -n promptdb
```

### Test Health Endpoint
```bash
kubectl port-forward svc/promptdb 8080:80 -n promptdb
curl http://localhost:8080/health
```

## Support

For issues or questions, please refer to the main documentation or open an issue in the repository.
