# Kibana Service

Part of the **TechDeveloper** infrastructure stack.

## Overview

Kibana is the visualization and management UI for Elasticsearch. It provides search, observability, and security solutions built on the Elastic Stack. This repository contains the configuration to deploy Kibana on Kubernetes, optimized for production use.

## Configuration

| Property | Value |
|----------|-------|
| **Namespace** | `common` |
| **Port** | `5601` (HTTP) |
| **Service DNS** | `kibana.common.svc.cluster.local` |
| **Elasticsearch** | Connects to `http://elasticsearch:9200` |

## Credentials

Kibana uses the same Elasticsearch credentials managed via Kubernetes Secrets (`elasticsearch-secret`).

| Key | Value |
|-----|-------|
| **User** | `elastic` / password from secret |

## Deployment

### Kubernetes (Manual)

```bash
# Apply all manifests
kubectl apply -f k8s/

# Verify deployment
kubectl rollout status deployment/kibana -n common

# Check pods
kubectl get pods -n common -l app=kibana

# Check service
kubectl get svc kibana -n common
```

> Ensure the namespace `common` exists and Elasticsearch is running before deploying.

### Jenkins (Automated)

A `Jenkinsfile` is included for automated deployment pipelines. The pipeline will automatically:

1. **Pre-Deployment Validation** — Validates manifests and cluster connectivity
2. **Deploy to K8s** — Applies K8s manifests with error handling
3. **Verify Rollout Status** — Waits for rollout completion (300s timeout)
4. **Health Check** — Verifies pods are ready and services are available
5. **Deployment Summary** — Shows deployment details and status

### Rollback

```bash
# Manual rollback (if needed)
kubectl rollout undo deployment/kibana -n common

# Check rollback status
kubectl rollout status deployment/kibana -n common
```

## Jenkins Pipeline Features

- ✅ Pre-deployment validation (manifest validation, cluster connectivity)
- ✅ Rollout status verification with timeout (300s)
- ✅ Automatic rollback on deployment failure
- ✅ Comprehensive health checks (pod readiness, service availability)
- ✅ Deployment summary with detailed status
- ✅ Cross-platform support (Unix/Windows agents)
- ✅ Build parameters (`SKIP_DEPLOY`, `DEPLOYMENT_MODE`)
- ✅ Structured error handling with try-catch
- ✅ Post-build actions (success/failure/unstable notifications)
- ✅ Workspace cleanup

### Error Handling

- Deployment failure → Automatic rollback to previous version
- Health check failure → Build marked as UNSTABLE
- Manifest validation failure → Build fails before deployment

## K8s Deployment Features

- ✅ Enhanced labels and annotations (component, tier, managed-by)
- ✅ Prometheus monitoring annotations
- ✅ Pod disruption budget for high availability
- ✅ Security context (runAsUser, capabilities, read-only filesystem)
- ✅ Rolling update strategy (maxUnavailable: 0, maxSurge: 1)
- ✅ Revision history limit (3 revisions)
- ✅ Enhanced probes (exec commands with proper timeout)
- ✅ Session affinity for consistent connections
- ✅ ConfigMap for service-specific configurations
- ✅ Resource requests and limits

## Security

- Drop ALL capabilities, add only required ones
- Run as non-root user (where applicable)
- Read-only root filesystem (where applicable)
- fsGroup for proper volume permissions

## Monitoring

### Prometheus Metrics

Service exposes metrics at configured port (check annotations).

### Check Logs

```bash
kubectl logs -n common -l app=kibana --tail=100 -f
```

### Check Events

```bash
kubectl get events -n common --field-selector involvedObject.name=kibana
```

## High Availability

- Pod disruption budget (minAvailable: 0 for single replica)
- Rolling update with zero downtime
- Graceful termination (30s grace period)

## Troubleshooting

### Deployment Stuck

```bash
# Check rollout status
kubectl rollout status deployment/kibana -n common

# Check pod events
kubectl describe pod -n common -l app=kibana

# Restart deployment
kubectl rollout restart deployment/kibana -n common
```

### Health Check Failures

```bash
# Check pod logs
kubectl logs -n common -l app=kibana

# Exec into pod
kubectl exec -it -n common <pod-name> -- /bin/sh

# Check readiness/liveness probes
kubectl describe pod -n common <pod-name>
```

## Production Checklist

- [ ] Resource limits configured appropriately
- [ ] Persistent storage provisioned
- [ ] Secrets created and referenced
- [ ] Monitoring annotations added
- [ ] Pod disruption budget configured
- [ ] Security context applied
- [ ] Health probes tuned for service startup time
- [ ] Revision history limit set
- [ ] Namespace created
- [ ] Elasticsearch running and accessible
- [ ] RBAC permissions granted (if needed)
