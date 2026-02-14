# Kibana Service

Part of the TechDeveloper infrastructure stack.

## Overview

Kibana is the visualization and management UI for Elasticsearch. It provides search, observability, and security solutions built on the Elastic Stack.

## Deployment

### Kubernetes

To deploy manually:

```bash
kubectl apply -f k8s/
```

Ensure the namespace `common` exists and Elasticsearch is running.

### Jenkins

A `Jenkinsfile` is included for automated deployment pipelines.

## Configuration

- **Namespace**: `common`
- **Port**: `5601` (HTTP)
- **Service DNS**: `kibana.common.svc.cluster.local`
- **Elasticsearch**: Connects to `http://elasticsearch:9200`

## Credentials

Kibana uses the same Elasticsearch credentials managed via Kubernetes Secrets (`elasticsearch-secret`).

- **User**: `elastic` / password from secret
