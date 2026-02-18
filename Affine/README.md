# AFFiNE Helm Chart

Helm chart for deploying [AFFiNE](https://affine.pro) to Kubernetes. PostgreSQL and Redis/Valkey are managed externally.

## Prerequisites

- Kubernetes 1.24+
- Helm 3+
- An external PostgreSQL database
- An external Redis/Valkey instance

## Setup

### 1. Create the database

```sql
CREATE USER affine WITH PASSWORD 'your-secure-password';
CREATE DATABASE affine OWNER affine;
GRANT ALL PRIVILEGES ON DATABASE affine TO affine;
```

### 2. Create the Kubernetes Secret

```bash
kubectl create secret generic affine-external-db \
  --from-literal=DATABASE_URL="postgresql://affine:your-secure-password@your-pg-host:5432/affine" \
  --from-literal=REDIS_SERVER_HOST="your-redis-host" \
  --from-literal=REDIS_SERVER_PORT="6379" \
  --from-literal=REDIS_SERVER_PASSWORD="your-redis-password"
```

Omit `REDIS_SERVER_PASSWORD` if your Redis instance doesn't require authentication.

### 3. Install the chart

```bash
helm install affine ./affine
```

Or with custom values:

```bash
helm install affine ./affine -f my-values.yaml
```

## Configuration

| Parameter | Description | Default |
|---|---|---|
| `image.repository` | Container image repository | `ghcr.io/toeverything/affine-graphql` |
| `image.tag` | Container image tag | `stable` |
| `image.pullPolicy` | Image pull policy | `IfNotPresent` |
| `externalSecret.name` | Name of the user-provided Secret | `affine-external-db` |
| `affine.privateKey` | App private key (auto-generated if empty) | `""` |
| `affine.serverHost` | Public-facing hostname | `""` |
| `affine.serverHttps` | Whether the public URL uses HTTPS | `false` |
| `affine.redisDatabase` | Redis database number (0-15) | `""` |
| `persistence.enabled` | Enable PVC for blob storage | `true` |
| `persistence.size` | PVC size | `10Gi` |
| `persistence.storageClass` | PVC storage class | `""` |
| `ingress.enabled` | Enable Ingress | `false` |
| `ingress.className` | Ingress class name | `""` |
| `ingress.hosts` | Ingress host rules | `[{host: affine.local, paths: [{path: /, pathType: Prefix}]}]` |
| `ingress.tls` | Ingress TLS configuration | `[]` |
| `service.type` | Service type | `ClusterIP` |
| `service.port` | Service port | `3010` |
| `resources` | CPU/memory requests and limits | `{}` |

## Database Migrations

A Kubernetes Job runs automatically before each install/upgrade (`helm.sh/hook: pre-install,pre-upgrade`) to execute database migrations via `node ./scripts/self-host-predeploy.js`.

## Uninstall

```bash
helm uninstall affine
```

Note: The PVC is not deleted automatically. To remove it:

```bash
kubectl delete pvc affine-storage
```
