# Kibana Docker

Kibana Docker setup for Elasticsearch data visualization, log analysis, and monitoring dashboards.

Part of the **TechDeveloper** infrastructure stack.

---

## 📋 Table of Contents

- [Overview](#overview)
- [Quick Start](#quick-start)
- [Docker Compose Deployment](#docker-compose-deployment)
- [Kubernetes Deployment](#kubernetes-deployment)
- [Configuration](#configuration)
- [Environment Variables](#environment-variables)
- [Volumes](#volumes)
- [Networking](#networking)
- [Web UI Features](#web-ui-features)
- [Index Pattern Creation](#index-pattern-creation)
- [Discover](#discover)
- [Visualize](#visualize)
- [Dashboard](#dashboard)
- [Dev Tools](#dev-tools)
- [Common Queries and Filters](#common-queries-and-filters)
- [Monitoring](#monitoring)
- [Security](#security)
- [Troubleshooting](#troubleshooting)
- [Production Checklist](#production-checklist)

---

## Overview

Kibana is the visualization and management UI for Elasticsearch. It provides powerful tools for exploring, visualizing, and managing data stored in Elasticsearch indices.

**Use Cases:**
- Log analysis and troubleshooting
- Real-time monitoring dashboards
- Application performance monitoring (APM)
- Security analytics and SIEM
- Business intelligence and analytics
- Data exploration and discovery

**Key Features:**
- Interactive data exploration (Discover)
- Powerful visualizations (Charts, Maps, Tables)
- Custom dashboards
- Dev Tools console for API testing
- Index management
- Query builder with KQL and Lucene syntax

---

## Quick Start

### Prerequisites

- Docker & Docker Compose installed
- Elasticsearch running and accessible
- Port 5601 available
- Minimum 2GB RAM allocated to Docker

### Run with Docker Compose

```bash
# Clone repository
git clone https://github.com/techdeveloper-org/kibana.git
cd kibana

# Ensure Elasticsearch is running
docker ps | grep elasticsearch

# Create network (if not exists)
docker network create microservices-network

# Start Kibana
docker-compose up -d

# Check status
docker-compose ps

# View logs
docker-compose logs -f

# Wait for Kibana to start (30-60 seconds)
# Access Web UI
# Open browser: http://localhost:5601
```

**Expected Response:**
- Browser opens Kibana home page
- Status: "Kibana is now available"
- Green status indicator

---

## Docker Compose Deployment

### Configuration

The `docker-compose.yml` includes:

```yaml
version: "3.8"

services:
  kibana:
    image: docker.elastic.co/kibana/kibana:9.2.3
    container_name: kibana
    restart: always
    environment:
      - ELASTICSEARCH_HOSTS=http://elasticsearch:9200
      - xpack.security.enabled=false
      - "NODE_OPTIONS=--max-old-space-size=512"
    ports:
      - "5601:5601"
    volumes:
      - kibana_data:/usr/share/kibana/data
    networks:
      - microservices-network
```

### Commands

```bash
# Start service
docker-compose up -d

# Stop service
docker-compose down

# Stop and remove volumes
docker-compose down -v

# Restart service
docker-compose restart

# View logs
docker-compose logs -f kibana

# Execute commands inside container
docker-compose exec kibana bash
```

### Data Persistence

Data is persisted in Docker volume `kibana_data`. To backup:

```bash
# Backup volume
docker run --rm -v kibana_data:/data -v $(pwd):/backup alpine \
  tar czf /backup/kibana-backup-$(date +%Y%m%d).tar.gz -C /data .

# Restore volume
docker run --rm -v kibana_data:/data -v $(pwd):/backup alpine \
  tar xzf /backup/kibana-backup-YYYYMMDD.tar.gz -C /data
```

---

## Kubernetes Deployment

### Configuration

| Property | Value |
|----------|-------|
| **Namespace** | `common` |
| **Port** | `5601` (HTTP) |
| **Service DNS** | `kibana.common.svc.cluster.local` |
| **Elasticsearch** | Connects to `http://elasticsearch:9200` |

### Credentials

Kibana uses the same Elasticsearch credentials managed via Kubernetes Secrets (`elasticsearch-secret`).

| Key | Value |
|-----|-------|
| **User** | `elastic` / password from secret |

### Manual Deployment

```bash
# Ensure namespace exists
kubectl create namespace common --dry-run=client -o yaml | kubectl apply -f -

# Ensure Elasticsearch is running
kubectl get pods -n common -l app=elasticsearch

# Apply all manifests
kubectl apply -f k8s/

# Verify deployment
kubectl rollout status deployment/kibana -n common

# Check pods
kubectl get pods -n common -l app=kibana

# Check service
kubectl get svc kibana -n common
```

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

### K8s Features

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

---

## Configuration

### Kibana Configuration

| Setting | Value | Description |
|---------|-------|-------------|
| `ELASTICSEARCH_HOSTS` | `http://elasticsearch:9200` | Elasticsearch connection URL |
| `xpack.security.enabled` | `false` | Security disabled for local dev |
| `NODE_OPTIONS` | `--max-old-space-size=512` | Node.js memory limit (512MB) |

### Production Recommendations

For production, consider:

```yaml
environment:
  - ELASTICSEARCH_HOSTS=https://elasticsearch:9200
  - ELASTICSEARCH_USERNAME=kibana_system
  - ELASTICSEARCH_PASSWORD=${KIBANA_PASSWORD}
  - xpack.security.enabled=true
  - xpack.encryptedSavedObjects.encryptionKey=${ENCRYPTION_KEY}
  - "NODE_OPTIONS=--max-old-space-size=2048"  # 2GB for production
  - SERVER_PUBLICBASEURL=https://kibana.yourdomain.com
```

---

## Environment Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `ELASTICSEARCH_HOSTS` | `http://elasticsearch:9200` | Elasticsearch connection URL |
| `ELASTICSEARCH_USERNAME` | - | Elasticsearch username |
| `ELASTICSEARCH_PASSWORD` | - | Elasticsearch password |
| `xpack.security.enabled` | `false` | Enable X-Pack security |
| `NODE_OPTIONS` | `--max-old-space-size=512` | Node.js heap size |
| `SERVER_HOST` | `0.0.0.0` | Server bind address |
| `SERVER_PORT` | `5601` | Server port |
| `SERVER_NAME` | `kibana` | Server name |

### Memory Configuration

**Important:** Adjust Node.js memory based on usage.

```bash
# For dev/test (512MB)
NODE_OPTIONS=--max-old-space-size=512

# For production (2GB)
NODE_OPTIONS=--max-old-space-size=2048

# For heavy usage (4GB)
NODE_OPTIONS=--max-old-space-size=4096
```

---

## Volumes

### Docker Compose

| Volume | Mount Point | Purpose |
|--------|-------------|---------|
| `kibana_data` | `/usr/share/kibana/data` | Saved objects, dashboards, visualizations |

### Volume Management

```bash
# List volumes
docker volume ls | grep kibana_data

# Inspect volume
docker volume inspect kibana_data

# Remove volume (⚠️ deletes all saved objects)
docker volume rm kibana_data
```

---

## Networking

### Docker Network

- **Network:** `microservices-network` (external)
- **Type:** Bridge
- **Create:** `docker network create microservices-network`

### Exposed Ports

| Port | Protocol | Purpose |
|------|----------|---------|
| `5601` | HTTP | Web UI |

### Access from Other Services

```yaml
# In other docker-compose.yml
services:
  my-service:
    networks:
      - microservices-network
    environment:
      KIBANA_URL: http://kibana:5601
```

---

## Web UI Features

### Accessing the Web UI

**URL:** http://localhost:5601

### Home Page

The Kibana home page provides quick access to:
- **Discover** - Explore your data
- **Visualize** - Create visualizations
- **Dashboard** - Build dashboards
- **Canvas** - Create pixel-perfect presentations
- **Maps** - Geographic data visualization
- **Machine Learning** - Anomaly detection
- **Stack Management** - Settings and configurations

### Navigation

- **Left sidebar:** Main navigation menu
- **Top bar:** Search, time picker, refresh
- **Breadcrumbs:** Current location
- **Help:** Documentation and shortcuts

---

## Index Pattern Creation

Before exploring data, create an index pattern:

### Steps

1. **Navigate to Stack Management**
   - Click hamburger menu (☰)
   - Select "Stack Management"
   - Click "Index Patterns"

2. **Create Index Pattern**
   - Click "Create index pattern"
   - Enter pattern: `logs-*` or `products-*`
   - Click "Next step"

3. **Configure Time Field**
   - Select time field: `@timestamp` (recommended)
   - Click "Create index pattern"

### Example Index Patterns

| Pattern | Description |
|---------|-------------|
| `logs-*` | All log indices |
| `metrics-*` | All metric indices |
| `products-*` | Product data |
| `orders-2026-*` | Orders for 2026 |
| `*` | All indices (not recommended) |

### API Method

```bash
# Create index pattern via API
curl -X POST "localhost:5601/api/saved_objects/index-pattern/logs-pattern" \
  -H "kbn-xsrf: true" \
  -H "Content-Type: application/json" \
  -d '{
    "attributes": {
      "title": "logs-*",
      "timeFieldName": "@timestamp"
    }
  }'
```

---

## Discover

**Purpose:** Explore and search your data in real-time.

### Features

- **Search bar:** KQL or Lucene query syntax
- **Time picker:** Select time range
- **Fields:** Available fields from index
- **Document table:** View matching documents
- **Histogram:** Document count over time
- **Filters:** Add field-based filters
- **Save:** Save searches for reuse

### Quick Start

1. Navigate to Discover
2. Select index pattern: `logs-*`
3. Set time range: "Last 15 minutes"
4. Enter search query: `level: ERROR`
5. View results

### Example Queries

```
# Simple text search
message: "failed"

# Field-based search
status: 500 AND method: "POST"

# Range query
response_time > 1000

# Wildcard search
user.name: john*

# Boolean operators
(level: ERROR OR level: WARN) AND service: "auth-service"

# Negation
NOT status: 200

# Exists query
_exists_: error.message
```

---

## Visualize

**Purpose:** Create visual representations of your data.

### Visualization Types

| Type | Use Case |
|------|----------|
| **Line chart** | Time-series data, trends |
| **Bar chart** | Categorical comparisons |
| **Pie chart** | Proportions, percentages |
| **Data table** | Tabular data display |
| **Metric** | Single value display |
| **Tag cloud** | Text frequency |
| **Heat map** | Matrix data |
| **Gauge** | Progress, thresholds |
| **Goal** | Target vs actual |
| **Markdown** | Custom text/HTML |

### Creating a Visualization

1. **Navigate to Visualize**
   - Click "Create visualization"
   - Select type (e.g., "Line")
   - Choose index pattern

2. **Configure Metrics**
   - Y-axis: Count, Sum, Average, etc.
   - Example: Average response_time

3. **Configure Buckets**
   - X-axis: Date Histogram, Terms, etc.
   - Example: @timestamp per hour

4. **Apply and Save**
   - Click "Update"
   - Click "Save"
   - Enter name: "Response Time Trend"

### Example: Error Count by Service

```
Visualization: Vertical Bar
Index Pattern: logs-*
Metrics: Count
Buckets:
  - X-axis: Terms
  - Field: service.keyword
  - Size: 10
  - Order: Descending
Filters: level: ERROR
```

---

## Dashboard

**Purpose:** Combine multiple visualizations into a single view.

### Creating a Dashboard

1. **Navigate to Dashboard**
   - Click "Create dashboard"

2. **Add Visualizations**
   - Click "Add"
   - Select saved visualizations
   - Arrange and resize panels

3. **Configure Dashboard**
   - Set time range
   - Add filters
   - Configure refresh interval

4. **Save Dashboard**
   - Click "Save"
   - Enter name: "Application Monitoring"

### Example Dashboard Layouts

**Application Monitoring:**
- Error count over time (Line chart)
- Error breakdown by service (Pie chart)
- Top 10 error messages (Data table)
- Response time trend (Area chart)
- Request count metric (Metric)

**Business Analytics:**
- Revenue over time (Line chart)
- Sales by category (Bar chart)
- Top products (Data table)
- Conversion rate (Gauge)
- Total orders (Metric)

### Dashboard Features

- **Auto-refresh:** 5s, 10s, 30s, 1m, 5m, 15m, 30m, 1h
- **Time picker:** Quick ranges or custom
- **Filters:** Global dashboard filters
- **Full-screen mode:** Presentation view
- **Export:** PDF or PNG
- **Share:** Generate shareable URL

---

## Dev Tools

**Purpose:** Execute Elasticsearch API requests directly.

### Console

**Location:** Dev Tools > Console

**Features:**
- Syntax highlighting
- Auto-completion
- Request history
- Multi-line requests
- JSON formatting

### Common Commands

**Check Elasticsearch Status:**
```json
GET /
```

**List All Indices:**
```json
GET /_cat/indices?v
```

**Create Index:**
```json
PUT /products
{
  "settings": {
    "number_of_shards": 1,
    "number_of_replicas": 0
  },
  "mappings": {
    "properties": {
      "name": { "type": "text" },
      "price": { "type": "float" },
      "category": { "type": "keyword" }
    }
  }
}
```

**Index Document:**
```json
POST /products/_doc/1
{
  "name": "Laptop",
  "price": 999.99,
  "category": "Electronics",
  "timestamp": "2026-02-16T12:00:00"
}
```

**Search Documents:**
```json
GET /products/_search
{
  "query": {
    "match": {
      "category": "Electronics"
    }
  }
}
```

**Aggregations:**
```json
GET /products/_search
{
  "size": 0,
  "aggs": {
    "avg_price": {
      "avg": {
        "field": "price"
      }
    },
    "categories": {
      "terms": {
        "field": "category",
        "size": 10
      }
    }
  }
}
```

**Delete Index:**
```json
DELETE /products
```

### Shortcuts

| Shortcut | Action |
|----------|--------|
| `Ctrl/Cmd + Enter` | Submit request |
| `Ctrl/Cmd + /` | Comment/uncomment |
| `Ctrl/Cmd + I` | Auto-format |
| `Ctrl/Cmd + L` | Clear console |

---

## Common Queries and Filters

### KQL (Kibana Query Language)

**Field Searches:**
```
status: 200
level: ERROR
service.name: "auth-service"
response_time > 1000
```

**Boolean Operators:**
```
status: 500 and method: "POST"
level: ERROR or level: WARN
not status: 200
```

**Wildcards:**
```
user.name: john*
message: *failed*
```

**Ranges:**
```
price > 100 and price < 500
@timestamp > "2026-02-15"
```

**Nested Fields:**
```
user.email: "user@example.com"
error.stack_trace: *NullPointerException*
```

### Lucene Syntax

**Field Searches:**
```
status:200
level:ERROR
service.name:"auth-service"
response_time:[1000 TO *]
```

**Boolean Operators:**
```
status:500 AND method:"POST"
level:ERROR OR level:WARN
NOT status:200
```

**Wildcards:**
```
user.name:john*
message:*failed*
```

**Ranges:**
```
price:[100 TO 500]
@timestamp:[2026-02-15 TO 2026-02-16]
```

**Fuzzy Search:**
```
message:elasticsearch~2
```

**Proximity Search:**
```
"quick brown fox"~5
```

### Common Filters

**Time Range Filters:**
- Last 15 minutes
- Last 1 hour
- Last 24 hours
- Last 7 days
- Today
- This week
- This month

**Field Filters:**
```
level is ERROR
status is one of 500, 502, 503
response_time is between 1000 and 5000
message contains "error"
user.name exists
```

---

## Monitoring

### Health Check

**Check Kibana Status:**
```bash
curl http://localhost:5601/api/status
```

**Expected Response:**
```json
{
  "status": {
    "overall": {
      "level": "available",
      "summary": "All services are available"
    }
  }
}
```

### Check Logs

**Docker Compose:**
```bash
docker-compose logs -f kibana
```

**Kubernetes:**
```bash
kubectl logs -n common -l app=kibana --tail=100 -f
```

### Prometheus Metrics

Kibana exposes metrics when monitoring is enabled.

**Enable Monitoring:**
```yaml
environment:
  - xpack.monitoring.enabled=true
  - xpack.monitoring.collection.enabled=true
```

---

## Security

### Development (Current Setup)

- ⚠️ Security disabled (`xpack.security.enabled=false`)
- ⚠️ No authentication required
- ⚠️ HTTP only (no SSL/TLS)

**Only for local development!**

### Production Security

Enable security features:

```yaml
environment:
  - xpack.security.enabled=true
  - ELASTICSEARCH_USERNAME=kibana_system
  - ELASTICSEARCH_PASSWORD=${KIBANA_PASSWORD}
  - ELASTICSEARCH_SSL_VERIFICATIONMODE=certificate
  - SERVER_SSL_ENABLED=true
  - SERVER_SSL_CERTIFICATE=/usr/share/kibana/config/kibana.crt
  - SERVER_SSL_KEY=/usr/share/kibana/config/kibana.key
```

### Authentication Methods

- **Basic authentication** (Elasticsearch credentials)
- **SAML** (SSO integration)
- **OIDC** (OpenID Connect)
- **Kerberos** (Active Directory)
- **PKI** (Certificate-based)

### Kubernetes Security

- Drop ALL capabilities, add only required ones
- Run as non-root user (where applicable)
- Read-only root filesystem (where applicable)
- fsGroup for proper volume permissions
- Network policies for traffic control

---

## Troubleshooting

### Container Won't Start

**Check logs:**
```bash
docker-compose logs kibana
```

**Common issues:**
- Elasticsearch not running
- Wrong Elasticsearch URL
- Port 5601 already in use
- Insufficient memory

**Fix Elasticsearch connection:**
```bash
# Check if Elasticsearch is running
docker ps | grep elasticsearch

# Test Elasticsearch connection
curl http://localhost:9200

# Update ELASTICSEARCH_HOSTS if needed
```

### "Kibana server is not ready yet"

**Symptoms:**
Browser shows "Kibana server is not ready yet"

**Causes:**
- Kibana still starting up (wait 30-60s)
- Cannot connect to Elasticsearch
- Memory issues

**Solutions:**
```bash
# Wait for startup
docker-compose logs -f kibana

# Check Elasticsearch connectivity
docker-compose exec kibana curl http://elasticsearch:9200

# Increase memory
# Edit docker-compose.yml: NODE_OPTIONS=--max-old-space-size=1024
```

### Out of Memory

**Symptoms:**
```
JavaScript heap out of memory
FATAL ERROR: CALL_AND_RETRY_LAST Allocation failed
```

**Solution:**
```yaml
environment:
  - "NODE_OPTIONS=--max-old-space-size=1024"  # Increase to 1GB
```

Or increase Docker memory limit in Docker Desktop settings.

### Index Pattern Not Found

**Symptoms:**
- "No index patterns found"
- Cannot access Discover

**Solution:**
1. Verify Elasticsearch has data: `curl http://localhost:9200/_cat/indices?v`
2. Create index pattern (see [Index Pattern Creation](#index-pattern-creation))
3. Ensure time field exists if using time-based pattern

### Connection Refused

**Check:**
```bash
docker-compose ps
curl http://localhost:5601
```

**Solutions:**
1. Wait for service to start (30-60s)
2. Check if container is running
3. Verify port mapping: `docker-compose port kibana 5601`
4. Check Elasticsearch connectivity

### Kubernetes Issues

**Deployment Stuck:**
```bash
kubectl rollout status deployment/kibana -n common
kubectl describe pod -n common -l app=kibana
```

**Health Check Failures:**
```bash
kubectl logs -n common -l app=kibana
kubectl exec -it -n common <pod-name> -- /bin/sh
curl http://localhost:5601/api/status
```

---

## Production Checklist

### Docker Compose
- [ ] Increase Node.js memory based on usage
- [ ] Enable X-Pack security
- [ ] Configure SSL/TLS
- [ ] Set up persistent volume backups
- [ ] Configure Elasticsearch authentication
- [ ] Enable monitoring
- [ ] Set proper restart policy
- [ ] Use production-grade image tag (not `latest`)
- [ ] Configure encryption keys for saved objects
- [ ] Set up reverse proxy (nginx/traefik)

### Kubernetes
- [ ] Resource limits configured appropriately
- [ ] Persistent storage provisioned (StorageClass)
- [ ] Secrets created and referenced
- [ ] Monitoring annotations added
- [ ] Pod disruption budget configured
- [ ] Security context applied
- [ ] Health probes tuned for service startup time
- [ ] Revision history limit set
- [ ] Namespace created
- [ ] Elasticsearch running and accessible
- [ ] RBAC permissions granted (if needed)
- [ ] Network policies configured
- [ ] Backup strategy implemented
- [ ] Ingress configured for external access

---

## Integration with Elasticsearch

### Spring Boot Logging to Elasticsearch

**Logback Configuration:**
```xml
<!-- logback-spring.xml -->
<appender name="LOGSTASH" class="net.logstash.logback.appender.LogstashTcpSocketAppender">
    <destination>logstash:5000</destination>
    <encoder class="net.logstash.logback.encoder.LogstashEncoder">
        <customFields>{"service":"auth-service"}</customFields>
    </encoder>
</appender>
```

### Logstash Integration

```conf
# logstash.conf
input {
  tcp {
    port => 5000
    codec => json
  }
}

filter {
  # Add custom fields
  mutate {
    add_field => { "environment" => "production" }
  }
}

output {
  elasticsearch {
    hosts => ["elasticsearch:9200"]
    index => "logs-%{+YYYY.MM.dd}"
  }
}
```

### Application Metrics

**Send metrics to Elasticsearch:**
```java
// Java example using Micrometer
@Configuration
public class MetricsConfig {
    @Bean
    public ElasticsearchMeterRegistry elasticsearchMeterRegistry() {
        return new ElasticsearchMeterRegistry(
            ElasticsearchConfig.DEFAULT,
            Clock.SYSTEM
        );
    }
}
```

---

## References

- [Kibana Documentation](https://www.elastic.co/guide/en/kibana/current/index.html)
- [Docker Hub - Kibana](https://hub.docker.com/_/kibana)
- [Kibana Query Language (KQL)](https://www.elastic.co/guide/en/kibana/current/kuery-query.html)
- [Lucene Query Syntax](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-query-string-query.html)
- [Kibana Visualizations](https://www.elastic.co/guide/en/kibana/current/visualize.html)

---

**Version:** 9.2.3
**Maintained by:** TechDeveloper Team
**Last Updated:** 2026-02-16
