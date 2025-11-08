# GUIDE POST TEMPLATE
# Tags: #guide, #elasticsearch, #kubernetes, #easy, #homelab

---

## Guide Overview

**Guide:** Building and Optimizing Elasticsearch on Kubernetes: A Logging Stack Homelab Journey
**Category:** Monitoring / Infrastructure Optimization
**Difficulty:** Easy
**Estimated Time:** 1-2 hours for full deployment and optimization
**Cost:** Free (open-source tools)
**Note:** This guide documents a complete Elasticsearch deployment from initial setup through optimization, with all configurations tested and validated in a live homelab environment.

---

## What You'll Build

This guide walks through deploying a complete Elasticsearch logging stack in Kubernetes, then documents the real-world problems that emerged during operation and how they were systematically resolved.

By the end of this guide, you'll have:
- A working Elasticsearch, Kibana, and Filebeat deployment on Kubernetes
- Ready-to-go configurations for single-node homelab environments
- Solutions for common operational problems (crash loops, resource sizing, index management)
- A methodology for using AI tools to accelerate troubleshooting
- Tested configurations handling up to 40+ million documents across multiple namespaces

---

## Brief Background on the ELK Stack

The ELK Stack (Elasticsearch, Logstash, Kibana) is a popular open-source solution for centralized logging and log analysis. In modern implementations, Filebeat often replaces Logstash as a more lightweight log shipper, making it the "EFK" stack.

**Components:**
- **Elasticsearch**: A distributed search and analytics engine that stores and indexes log data
- **Filebeat**: A lightweight log shipper that collects logs from various sources and forwards them to Elasticsearch
- **Kibana**: A visualization and exploration tool that provides a web interface for querying and visualizing data stored in Elasticsearch

This stack is particularly valuable in Kubernetes environments where logs from hundreds of pods across multiple nodes need to be aggregated, searched, and analyzed from a single location.

---

## Use Cases for the ELK Stack

### When to Use ELK/EFK

**Ideal for:**
- **Complex log analysis** - Full-text search across all log fields with powerful query language
- **Multi-source aggregation** - Collecting logs from diverse sources (Kubernetes, applications, infrastructure)
- **Historical analysis** - Investigating issues that occurred hours or days ago with fast search
- **Security monitoring** - Correlating security events across multiple systems
- **Compliance** - Retaining logs for audit requirements with search capabilities
- **Dashboarding** - Creating visualizations and dashboards in Kibana for team visibility

**Homelab use cases:**
- Learning Kubernetes logging patterns
- Troubleshooting application deployments
- Monitoring security tools and experiments
- Centralizing logs from multiple namespaces/projects
- Building skills relevant to enterprise environments

**Production use cases:**
- Application performance monitoring (APM)
- Security Information and Event Management (SIEM)
- Infrastructure monitoring and alerting
- Customer support troubleshooting
- Business analytics from application logs

### When NOT to Use ELK

**Consider alternatives if:**

**Resource-constrained environments:**
- ELK requires significant resources (4GB+ RAM for Elasticsearch alone)
- Alternative: **Grafana Loki** uses ~80% fewer resources for simple log aggregation
- Trade-off: Loki is slower for full-text search but much lighter

**Simple log viewing:**
- If you just need to "tail" recent logs without complex queries
- Alternative: **kubectl logs** or simple file-based logging
- Trade-off: No centralization or historical search

**Managed solutions preferred:**
- If you don't want to maintain infrastructure
- Alternative: **CloudWatch**, **Datadog**, **New Relic**, **Splunk Cloud**
- Trade-off: Ongoing costs vs. self-hosted maintenance burden

**Metric-focused monitoring:**
- ELK is optimized for logs, not time-series metrics
- Alternative: **Prometheus + Grafana** for metrics
- Note: Many deployments use both (ELK for logs, Prometheus for metrics)

### ELK vs Alternatives Comparison

| Feature | ELK/EFK | Loki | Splunk | CloudWatch |
|---------|---------|------|--------|------------|
| **Cost** | Free (self-hosted) | Free (self-hosted) | Paid | Pay-per-use |
| **Resource Usage** | High (4GB+) | Low (~500MB) | High | N/A (managed) |
| **Full-Text Search** | Excellent | Slower | Excellent | Good |
| **Complexity** | Moderate | Low | Moderate | Low |
| **Best For** | Complex queries | Simple aggregation | Enterprise | AWS workloads |

---

## Prerequisites

**Required:**
- Running Kubernetes cluster (RKE, k3s, or any distribution)
- At least one node with 4GB+ RAM available for Elasticsearch
- Basic Kubectl and Helm knowledge
- Understanding of Kubernetes concepts (StatefulSets, DaemonSets, Deployments, ConfigMaps)

**Helpful but Optional:**
- Familiarity with Elasticsearch concepts
- Experience with log aggregation systems
- Access to AI tools (Claude, ChatGPT) for troubleshooting assistance

---

## Architecture Overview

This deployment creates a centralized logging pipeline for Kubernetes:

```
┌─────────────────────────────────────────────────────────────┐
│             Kubernetes Cluster                              │
│                                                             │
│  ┌──────────┐  ┌──────────┐   ┌──────────┐                  │
│  │   Pod    │  │   Pod    │   │   Pod    │                  │
│  │  Logs    │  │  Logs    │   │  Logs    │                  │
│  └────┬─────┘  └────┬─────┘   └────┬─────┘                  │
│       │             │              │                        │
│       └─────────────┴──────────────┘                        │
│                     │                                       │
│              ┌──────▼───────┐                               │
│              │   Filebeat   │   (DaemonSet on each node)    │
│              │  Log Shipper │                               │
│              └──────┬───────┘                               │
│                     │                                       │
│              ┌──────▼────────┐                              │
│              │ Elasticsearch │  (StatefulSet, single node)  │
│              │   Storage &   │                              │
│              │    Indexing   │                              │
│              └──────┬────────┘                              │
│                     │                                       │
│              ┌──────▼────────┐                              │
│              │    Kibana     │  (Deployment)                │
│              │ Visualization │                              │
│              └───────────────┘                              │
└─────────────────────────────────────────────────────────────┘
```

**Data Flow:**
1. Filebeat runs on each node, tailing container logs
2. Logs are shipped to Elasticsearch for indexing and storage
3. Kibana provides a web UI for searching and visualizing logs

---

## Hardware/Software Requirements

### Minimum Requirements
- **Cluster:** Kubernetes cluster with at least one node
- **Node Resources:** 4GB+ RAM available for Elasticsearch
- **Storage:** Calculate based on your log volume (see formula below)

### Software Versions
- Kubernetes: 1.24+ (any distribution)
- Elasticsearch: 8.5.1
- Kibana: 8.5.1
- Filebeat: 8.5.1
- Helm: 3.x

### Storage Sizing Calculator

**Formula:**
```
Required Storage = (Daily Log Volume in GB × Retention Days) × 1.5
```

The 1.5 multiplier accounts for:
- Elasticsearch overhead (indices, mappings, shards): ~20%
- Growth buffer: ~30%

**Example Calculation (this deployment):**
- Daily log volume: ~300MB
- Retention period: 30 days
- Calculation: (0.3 GB × 30) × 1.5 = 13.5GB
- Allocated: 50Gi (provides 4x headroom for growth)

**How to measure your daily log volume:**
```bash
# After running 24 hours, check index sizes:
kubectl exec elasticsearch-master-0 -n elastic-stack -- \
  curl -s -k -u "elastic:PASSWORD" \
  "https://localhost:9200/_cat/indices?v&s=store.size:desc"

# Sum the sizes of indices from the last 24 hours
```

**Quick Reference:**
- Small homelab (1-5 namespaces, low traffic): 20-30Gi
- Medium homelab (5-15 namespaces): 50-100Gi
- Large homelab (15+ namespaces, high traffic): 100-200Gi

**Final Resource Usage (this deployment):**
- Elasticsearch: 4GB RAM, 1 CPU
- Kibana: 2GB RAM, 1 CPU
- Filebeat: 100-200MB RAM per node
- Storage: 12GB used of 50Gi allocated (300MB/day × 30 days with overhead)

---

## Quick Reference

This section provides essential commands and information for managing your Elasticsearch deployment.

### Important Endpoints and Ports

- **Elasticsearch:** `https://elasticsearch-master.elastic-stack.svc:9200`
- **Kibana:** `http://kibana-kibana.elastic-stack.svc:5601`
- **Namespace:** `elastic-stack`

### Default Credentials

```bash
# Get Elasticsearch password
kubectl get secret elasticsearch-master-credentials \
  -n elastic-stack -o jsonpath='{.data.password}' | base64 -d

# Default username: elastic
```

### Common kubectl Commands

**Check pod status:**
```bash
# All elastic-stack pods
kubectl get pods -n elastic-stack

# Watch for changes
kubectl get pods -n elastic-stack -w

# Check specific component
kubectl get pods -n elastic-stack -l app=elasticsearch-master
kubectl get pods -n elastic-stack -l app=kibana-kibana
kubectl get pods -n elastic-stack -l app=filebeat-filebeat
```

**Check resource usage:**
```bash
# Pod resource consumption
kubectl top pods -n elastic-stack

# Node resource consumption
kubectl top nodes
```

**View logs:**
```bash
# Elasticsearch logs
kubectl logs elasticsearch-master-0 -n elastic-stack

# Kibana logs
kubectl logs deployment/kibana-kibana -n elastic-stack

# Filebeat logs (specific pod)
kubectl logs <filebeat-pod-name> -n elastic-stack

# Follow logs in real-time
kubectl logs -f elasticsearch-master-0 -n elastic-stack
```

**Access services locally:**
```bash
# Port-forward Elasticsearch
kubectl port-forward svc/elasticsearch-master 9200:9200 -n elastic-stack

# Port-forward Kibana
kubectl port-forward svc/kibana-kibana 5601:5601 -n elastic-stack

# Then access:
# Elasticsearch: https://localhost:9200
# Kibana: http://localhost:5601
```

### Elasticsearch API Commands

**Cluster health:**
```bash
# Get password first
ELASTIC_PASSWORD=$(kubectl get secret elasticsearch-master-credentials \
  -n elastic-stack -o jsonpath='{.data.password}' | base64 -d)

# Check cluster health
curl -k -u "elastic:${ELASTIC_PASSWORD}" \
  "https://localhost:9200/_cluster/health?pretty"

# Cluster stats
curl -k -u "elastic:${ELASTIC_PASSWORD}" \
  "https://localhost:9200/_cluster/stats?pretty"
```

**Index management:**
```bash
# List all indices
curl -k -u "elastic:${ELASTIC_PASSWORD}" \
  "https://localhost:9200/_cat/indices?v"

# List indices sorted by size
curl -k -u "elastic:${ELASTIC_PASSWORD}" \
  "https://localhost:9200/_cat/indices?v&s=store.size:desc"

# List indices sorted by creation date
curl -k -u "elastic:${ELASTIC_PASSWORD}" \
  "https://localhost:9200/_cat/indices?v&s=creation.date:desc"

# Delete specific index
curl -k -u "elastic:${ELASTIC_PASSWORD}" \
  -X DELETE "https://localhost:9200/index-name"
```

**Node information:**
```bash
# Node stats
curl -k -u "elastic:${ELASTIC_PASSWORD}" \
  "https://localhost:9200/_cat/nodes?v"

# JVM memory usage
curl -k -u "elastic:${ELASTIC_PASSWORD}" \
  "https://localhost:9200/_nodes/stats/jvm?pretty"
```

### Helm Commands

**Check current deployments:**
```bash
# List Helm releases in namespace
helm list -n elastic-stack

# Get values for a release
helm get values elasticsearch -n elastic-stack
helm get values kibana -n elastic-stack
helm get values filebeat -n elastic-stack
```

**Upgrade deployments:**
```bash
# Upgrade Elasticsearch
helm upgrade elasticsearch elastic/elasticsearch \
  -f elasticsearch-values.yaml \
  -n elastic-stack

# Upgrade Kibana
helm upgrade kibana elastic/kibana \
  -f kibana-values.yaml \
  -n elastic-stack

# Upgrade Filebeat
helm upgrade filebeat elastic/filebeat \
  -f filebeat-values.yaml \
  -n elastic-stack
```

**Rollback if needed:**
```bash
# See release history
helm history elasticsearch -n elastic-stack

# Rollback to previous version
helm rollback elasticsearch -n elastic-stack

# Rollback to specific revision
helm rollback elasticsearch 2 -n elastic-stack
```

### Quick Troubleshooting Commands

**Pod won't start:**
```bash
# Describe pod to see events
kubectl describe pod <pod-name> -n elastic-stack

# Check PVC status
kubectl get pvc -n elastic-stack

# Check node resources
kubectl top nodes
```

**Performance issues:**
```bash
# Check resource usage
kubectl top pods -n elastic-stack

# Check Elasticsearch heap usage
kubectl exec elasticsearch-master-0 -n elastic-stack -- \
  curl -k -u "elastic:${ELASTIC_PASSWORD}" \
  "https://localhost:9200/_nodes/stats/jvm?pretty"

# Check index count (too many can slow things down)
curl -k -u "elastic:${ELASTIC_PASSWORD}" \
  "https://localhost:9200/_cat/indices?v" | wc -l
```

**Restart components:**
```bash
# Restart Elasticsearch (StatefulSet - restarts in order)
kubectl rollout restart statefulset/elasticsearch-master -n elastic-stack

# Restart Kibana
kubectl rollout restart deployment/kibana-kibana -n elastic-stack

# Restart Filebeat
kubectl rollout restart daemonset/filebeat-filebeat -n elastic-stack
```

### Configuration File Locations

- Local values files: `elasticsearch-values.yaml`, `kibana-values.yaml`, `filebeat-values.yaml`
- See the **References and Additional Resources** section at the end of this guide for GitHub repository links

---

## Elastic Deployment

### Overview
Deploy Elasticsearch as a StatefulSet with persistent storage. This initial deployment uses conservative resource estimates that we'll adjust later based on actual usage. The yaml will deploy Elasticsearch in single-node mode.

### Instructions

**1. Create namespace**
```bash
kubectl create namespace elastic-stack
```

**2. Add Elastic Helm repository**
```bash
helm repo add elastic https://helm.elastic.co
helm repo update
```

**3. Create Elasticsearch values file**

Create `elasticsearch-values.yaml`:

```yaml
---
clusterName: "elasticsearch"
nodeGroup: "master"

# Elasticsearch roles for single-node deployment
roles:
  - master
  - data
  - data_content
  - data_hot
  - data_warm
  - data_cold
  - ingest
  - ml
  - remote_cluster_client
  - transform

replicas: 1
minimumMasterNodes: 1

# Initial resource allocation (conservative estimate)
# NOTE: This will be increased later based on actual usage
resources:
  requests:
    cpu: "1000m"
    memory: "2Gi"  # INITIAL - will increase to 4Gi after monitoring
  limits:
    cpu: "1000m"
    memory: "2Gi"  # INITIAL - will increase to 4Gi after monitoring

esJavaOpts: "-Xmx1g -Xms1g"  # INITIAL - 50% of 2Gi, will adjust later

# Persistent storage
# Use the storage calculator in the requirements section to determine your needs
# Formula: (Daily Log Volume GB × Retention Days) × 1.5
volumeClaimTemplate:
  accessModes: ["ReadWriteOnce"]
  resources:
    requests:
      storage: 50Gi  # Adjust based on your log volume

# Security settings
secret:
  enabled: true
  password: "elastic"  # Change in production

# Single-node optimization
antiAffinity: "hard"

protocol: https
httpPort: 9200
transportPort: 9300

# Security hardening
podSecurityContext:
  fsGroup: 1000
  runAsUser: 1000

securityContext:
  capabilities:
    drop:
      - ALL
  runAsNonRoot: true
  runAsUser: 1000

sysctlInitContainer:
  enabled: true

sysctlVmMaxMapCount: 262144

readinessProbe:
  failureThreshold: 3
  initialDelaySeconds: 10
  periodSeconds: 10
  successThreshold: 3
  timeoutSeconds: 5

clusterHealthCheckParams: "wait_for_status=yellow&timeout=1s"
```

**4. Deploy Elasticsearch**
```bash
helm install elasticsearh elastic/elasticsearch \
  -f elasticsearch-values.yaml \
  -n elastic-stack
```

**5. Wait for Elasticsearch to be ready**
```bash
kubectl get pods -n elastic-stack -w
```

Expected output:
```
NAME                     READY   STATUS    RESTARTS   AGE
elasticsearch-master-0   1/1     Running   0          3m
```

### Verification

**Check Elasticsearch health:**
```bash
# Get the password
ELASTIC_PASSWORD=$(kubectl get secret elasticsearch-master-credentials \
  -n elastic-stack -o jsonpath='{.data.password}' | base64 -d)

# Port-forward to access Elasticsearch
kubectl port-forward svc/elasticsearch-master 9200:9200 -n elastic-stack &

# Check cluster health
curl -k -u "elastic:${ELASTIC_PASSWORD}" https://localhost:9200/_cluster/health?pretty
```

Expected response:
```json
{
  "cluster_name" : "elasticsearch",
  "status" : "yellow",
  "number_of_nodes" : 1,
  "number_of_data_nodes" : 1
}
```

**Note:** Yellow status is expected for single-node clusters (no replicas possible).

---

## Kibana Deployment

### Overview
Kibana provides the web interface for querying and visualizing logs stored in Elasticsearch.

### Instructions

**1. Create Kibana values file**

Create `kibana-values.yaml`:

```yaml
---
image: "docker.elastic.co/kibana/kibana"
imageTag: "8.5.1"

replicas: 1

# Resource allocation
resources:
  requests:
    memory: "1Gi"
    cpu: "1000m"
  limits:
    memory: "2Gi"
    cpu: "1000m"

# Elasticsearch connection
elasticsearchHosts: "https://elasticsearch-master:9200"

# Environment variables
extraEnvs:
  - name: "ELASTICSEARCH_SSL_CERTIFICATEAUTHORITIES"
    value: "/usr/share/kibana/config/certs/ca.crt"
  - name: "SERVER_HOST"
    value: "0.0.0.0"
  - name: "NODE_OPTIONS"
    value: "--max-old-space-size=1800"

secretMounts:
  - name: elasticsearch-certs
    secretName: elasticsearch-master-certs
    path: /usr/share/kibana/config/certs
    readOnly: true

# Security context
securityContext:
  capabilities:
    drop:
      - ALL
  runAsNonRoot: true
  runAsUser: 1000

podSecurityContext:
  fsGroup: 1000

# Service configuration
service:
  type: ClusterIP
  port: 5601
```

**2. Create Kibana service account token**

Create a service account token for Kibana to authenticate with Elasticsearch:

```bash
# Create a service account token in Elasticsearch
kubectl exec elasticsearch-master-0 -n elastic-stack -- \
  /usr/share/elasticsearch/bin/elasticsearch-service-tokens create \
  elastic/kibana kibana-token > /tmp/kibana-token.txt

# Extract the token value (4th field in output)
TOKEN=$(cat /tmp/kibana-token.txt | awk '{print $4}')

# Create Kubernetes secret with the token
kubectl create secret generic kibana-kibana-es-token \
  -n elastic-stack \
  --from-literal=token="${TOKEN}"

# Clean up temp file
rm /tmp/kibana-token.txt
```

**3. Deploy Kibana**

```bash
helm install kibana elastic/kibana \
  -f kibana-values.yaml \
  -n elastic-stack
```

**4. Wait for Kibana to be ready**
```bash
kubectl get pods -n elastic-stack -w
```

### Verification

**Access Kibana:**
```bash
kubectl port-forward svc/kibana-kibana 5601:5601 -n elastic-stack
```

Navigate to `http://localhost:5601` in your browser. Login with:
- Username: `elastic`
- Password: (value from `ELASTIC_PASSWORD` variable)

---

## Filebeat Deployment

### Overview
Filebeat runs as a DaemonSet on each node, collecting container logs and shipping them to Elasticsearch.

### Instructions

**1. Create Filebeat values file**

Create `filebeat-values.yaml`:

```yaml
---
image: "docker.elastic.co/beats/filebeat"
imageTag: "8.5.1"
imagePullPolicy: "IfNotPresent"

# DaemonSet configuration
daemonset:
  enabled: true

deployment:
  enabled: false

# Cluster role for reading pod metadata
clusterRoleRules:
- apiGroups:
  - ""
  resources:
  - namespaces
  - pods
  - nodes
  verbs:
  - get
  - list
  - watch
- apiGroups:
  - apps
  resources:
  - replicasets
  verbs:
  - get
  - list
  - watch

# Resource limits
resources:
  requests:
    cpu: "100m"
    memory: "100Mi"
  limits:
    cpu: "200m"
    memory: "200Mi"

# Environment variables
extraEnvs:
  - name: ELASTICSEARCH_PASSWORD
    valueFrom:
      secretKeyRef:
        name: elasticsearch-master-credentials
        key: password

# Volume mounts for certificates
secretMounts:
  - name: elasticsearch-master-certs
    secretName: elasticsearch-master-certs
    path: /usr/share/filebeat/certs/
  - name: elasticsearch-credentials
    secretName: elasticsearch-master-credentials
    path: /usr/share/filebeat/secrets/

# Filebeat configuration
filebeatConfig:
  filebeat.yml: |
    filebeat.inputs:
    - type: container
      paths:
        - /var/log/containers/*.log
      processors:
      - add_kubernetes_metadata:
          host: ${NODE_NAME}
          matchers:
          - logs_path:
              logs_path: "/var/log/containers/"
      - drop_event:
          when:
            or:
              # Drop filebeat's own logs
              - equals:
                  kubernetes.namespace: "kube-system"
                  kubernetes.container.name: "filebeat"
              # Drop metrics-server logs (noisy)
              - equals:
                  kubernetes.namespace: "kube-system"
                  kubernetes.container.name: "metrics-server"

    # Send to Elasticsearch
    output.elasticsearch:
      hosts: ['https://elasticsearch-master.elastic-stack.svc:9200']
      protocol: "https"
      username: "elastic"
      password: "${ELASTICSEARCH_PASSWORD}"
      ssl.certificate_authorities:
        - /usr/share/filebeat/certs/ca.crt
      indices:
        - index: "logs-kubernetes-%{+yyyy.MM.dd}"
          when.or:
            - not:
                has_fields: ['kubernetes.namespace']
        - index: "logs-security-%{[kubernetes.namespace]}-%{+yyyy.MM.dd}"
          when.or:
            - equals:
                kubernetes.namespace: "your-security-namespace-1"
            - equals:
                kubernetes.namespace: "your-security-namespace-2"
            - equals:
                kubernetes.namespace: "your-security-namespace-3"
        - index: "logs-app-%{[kubernetes.namespace]}-%{+yyyy.MM.dd}"

    # ILM and index template settings
    setup.ilm.enabled: true
    setup.ilm.rollover_alias: "filebeat"
    setup.ilm.pattern: "{now/d}-000001"
    setup.template.name: "filebeat"
    setup.template.pattern: "logs-*"
    setup.template.settings:
      index.number_of_shards: 1
      index.number_of_replicas: 0

readinessProbe:
  failureThreshold: 3
  initialDelaySeconds: 10
  periodSeconds: 10
  successThreshold: 3
  timeoutSeconds: 15
```

**2. Deploy Filebeat**
```bash
helm install filebeat elastic/filebeat \
  -f filebeat-values.yaml \
  -n elastic-stack
```

**3. Verify Filebeat pods**
```bash
kubectl get pods -n elastic-stack -l app=filebeat-filebeat
```

Expected output (one pod per node):
```
NAME                      READY   STATUS    RESTARTS   AGE
filebeat-filebeat-abc12   1/1     Running   0          2m
filebeat-filebeat-def34   1/1     Running   0          2m
```

### Verification

**Check logs are flowing to Elasticsearch:**
```bash
# Port-forward to Elasticsearch
kubectl port-forward svc/elasticsearch-master 9200:9200 -n elastic-stack &

# Check indices
curl -k -u "elastic:${ELASTIC_PASSWORD}" \
  "https://localhost:9200/_cat/indices/logs-*?v"
```

Expected output:
```
health status index                      pri rep docs.count
yellow open   logs-kubernetes-2025.10.29  1   0      12345
yellow open   logs-app-default-2025.10.29 1   0       5678
```

**View logs in Kibana:**
1. Navigate to Kibana (http://localhost:5601)
2. Go to "Discover"
3. Create index pattern: `logs-*`
4. You should see logs flowing in

---

## Baking Period and Monitoring

### Overview
After initial deployment, I let the system run for several weeks to understand actual usage patterns and identify issues.

### Monitoring Commands

**Check resource usage:**
```bash
# Node-level resource usage
kubectl top nodes

# Pod-level resource usage
kubectl top pods -n elastic-stack

# Elasticsearch memory usage specifically
kubectl exec elasticsearch-master-0 -n elastic-stack -- \
  curl -k -u "elastic:${ELASTIC_PASSWORD}" \
  "https://localhost:9200/_nodes/stats/jvm?pretty"
```

**Check pod status and restarts:**
```bash
# Watch for crash loops
kubectl get pods -n elastic-stack -w

# Check pod restart counts
kubectl get pods -n elastic-stack
```

**Monitor Elasticsearch health:**
```bash
# Cluster stats
curl -k -u "elastic:${ELASTIC_PASSWORD}" \
  "https://localhost:9200/_cluster/stats?pretty"

# Index count and sizes
curl -k -u "elastic:${ELASTIC_PASSWORD}" \
  "https://localhost:9200/_cat/indices?v&s=store.size:desc"
```

### What I Observed (After 2-3 Weeks)

**Resource Usage Reality:**
```bash
$ kubectl top pods -n elastic-stack

NAME                       CPU    MEMORY
elasticsearch-master-0     823m   2760Mi  # Exceeding 2Gi limit!
kibana-kibana-xxx          200m   400Mi
filebeat-filebeat-xxx      50m    100Mi
filebeat-filebeat-yyy      45m    95Mi
```

**Problems Identified:**

1. **Elasticsearch Memory Issues**
   - Allocated: 2Gi
   - Actual usage: 2.76Gi
   - Risk: OOM kills, performance degradation

2. **Filebeat Crash Loops**
   ```bash
   $ kubectl get pods -n elastic-stack

   NAME                       READY   STATUS    RESTARTS   AGE
   filebeat-filebeat-abc12    1/1     Running   100+       3d
   ```

   Error in logs:
   ```
   cannot obtain lockfile: cannot start, data directory belongs to process with pid 8
   ```

3. **Index Sprawl**
   ```bash
   $ curl -k -u "elastic:${ELASTIC_PASSWORD}" \
     "https://localhost:9200/_cat/indices?v" | wc -l

   500+ indices  # No lifecycle management!
   ```

4. **No Backup Strategy**
   - Millions of documents stored
   - Growing data with no disaster recovery plan
   - Need automated backup solution

---

## Utilizing AI to Enhance Deployment

### Observing Findings with AI

At this point, I engaged Claude AI to help diagnose and fix these issues. This section documents that workflow and what we discovered.

### The AI-Assisted Workflow

**1. Gathered diagnostic data:**
```bash
# Filebeat crash loop investigation
kubectl describe pod filebeat-filebeat-abc12 -n elastic-stack > filebeat-crash.txt
kubectl logs filebeat-filebeat-abc12 -n elastic-stack --tail=100 > filebeat-logs.txt

# Elasticsearch resource analysis
kubectl describe pod elasticsearch-master-0 -n elastic-stack > es-describe.txt
kubectl top pod elasticsearch-master-0 -n elastic-stack > es-usage.txt

# Index analysis
curl -k -u "elastic:${ELASTIC_PASSWORD}" \
  "https://localhost:9200/_cat/indices?v&s=creation.date:desc" > indices.txt
```

**2. Presented problems to Claude AI:**
- Shared error messages and logs (sanitized of credentials)
- Described environment constraints (homelab, single-node)
- Asked for multiple solution options with trade-offs

**3. Evaluated solutions:**
- Discussed pros/cons of each approach
- Considered homelab vs. production best practices
- Made decisions based on my specific context

**4. Implemented fixes incrementally:**
- One change at a time
- Validated each fix before moving to next
- Documented decisions and findings

### Mitigating Findings Suggested by AI

The following sections detail each fix that was implemented based on AI-assisted troubleshooting.

#### Fix #1 - Backup Strategy (Velero + MinIO)

**Overview:**
Before making any risky configuration changes, I established a disaster recovery strategy using Velero to back up the Elasticsearch data to MinIO running on my NAS.

**Implementation Note:**
This guide focuses on Elasticsearch optimization, so I won't detail the full Velero/MinIO setup here. That will be covered in a dedicated backup guide. However, the key points:

- **Velero** installed in the cluster for Kubernetes backup/restore
- **MinIO** running on NAS as S3-compatible storage target
- **Scheduled backups** of the elastic-stack namespace and persistent volumes
- **Tested restores** to validate the backup chain works

**Why this was critical:**
With backups in place, I could confidently make configuration changes knowing I could recover from any mistakes. This enabled the emptyDir fix in the next step.

**Verification:**
```bash
# Check Velero backups exist
velero backup get

# Validate latest backup
velero backup describe <backup-name> --details
```

#### Fix #2 - Filebeat Crash Loop (emptyDir)

**Root Cause:**
Filebeat was using a `hostPath` volume for data persistence:

```yaml
# BEFORE (problematic configuration)
volumes:
  - name: data
    hostPath:
      path: /var/lib/filebeat-filebeat-elastic-stack-data
      type: DirectoryOrCreate
```

When Filebeat containers crashed, lock files remained on the host filesystem, preventing new containers from starting.

**The Fix:**

Changed to `emptyDir` volume type:

```bash
# Apply the patch
kubectl patch daemonset filebeat-filebeat -n elastic-stack \
  --type='json' \
  -p='[{"op": "replace", "path": "/spec/template/spec/volumes/3", "value": {"name": "data", "emptyDir": {}}}]'
```

**Current deployed configuration** (in DaemonSet spec):
```yaml
# AFTER (fixed configuration)
# File reference: kubectl get daemonset filebeat-filebeat -n elastic-stack -o yaml
volumes:
  - name: data
    emptyDir: {}  # ← The fix!
```

**Trade-offs:**

Pro:
- Clean restarts, no more crash loops
- Lock files cleared on pod restart

Con:
- Registry state (log file positions) lost on restart
- Potential duplicate logs after restart

Why acceptable:
- Logs are backed up to NAS
- Occasional duplicates preferable to non-functional system
- Homelab context (not financial transactions)

**Result:**

```bash
$ kubectl get pods -n elastic-stack -l app=filebeat-filebeat

NAME                      READY   STATUS    RESTARTS   AGE
filebeat-filebeat-abc12   1/1     Running   0          2d
filebeat-filebeat-def34   1/1     Running   0          2d
```

Zero restarts since October 29, 2025!

#### Fix #3 - Elasticsearch Resource Sizing

**The Problem:**

Initial allocation was conservative guesswork:
- Allocated: 2Gi RAM
- Actual usage: 2.76Gi RAM
- Java heap: 1g (50% of 2Gi)

This caused performance issues and risk of OOM kills.

**The Analysis:**

Elasticsearch needs memory for:
1. **Java heap** - Core operations (indexing, search)
2. **OS file cache** - Lucene segments (critical for performance)

Best practice: 50/50 split between heap and file cache.

**The Fix:**

Updated resource allocation to match reality:

```yaml
# BEFORE (initial guess)
resources:
  requests:
    cpu: "1000m"
    memory: "2Gi"
  limits:
    cpu: "1000m"
    memory: "2Gi"

esJavaOpts: "-Xmx1g -Xms1g"

# AFTER (data-driven sizing)
# See: https://github.com/npecka/peckacyber/blob/main/guides/elasticsearch-k8s-optimization/elasticsearch-values.yaml
resources:
  requests:
    cpu: "1000m"
    memory: "4Gi"  # Doubled to accommodate actual usage
  limits:
    cpu: "1000m"
    memory: "4Gi"

esJavaOpts: "-Xmx2g -Xms2g"  # 2GB heap, 2GB for OS/Lucene cache
```

**Applied via Helm upgrade:**
```bash
helm upgrade elasticsearh elastic/elasticsearch \
  -f elasticsearch-values.yaml \
  -n elastic-stack
```

**Result:**

```bash
$ kubectl top pod elasticsearch-master-0 -n elastic-stack

NAME                     CPU    MEMORY
elasticsearch-master-0   823m   2760Mi  # Now within limits!
```

Stable operation with no OOM events.

#### Fix #4 - Index Lifecycle Management (ILM)

**The Problem:**

500+ indices accumulated with no cleanup strategy:
- Daily indices never deleted
- Storage growing unbounded
- Performance degradation from too many indices

**The Solution:**

Enabled ILM in Filebeat configuration:

```yaml
# See: https://github.com/npecka/peckacyber/blob/main/guides/elasticsearch-k8s-optimization/filebeat-values.yaml

# ILM and index template settings
setup.ilm.enabled: true
setup.ilm.rollover_alias: "filebeat"
setup.ilm.pattern: "{now/d}-000001"
setup.template.name: "filebeat"
setup.template.pattern: "logs-*"
setup.template.settings:
  index.number_of_shards: 1
  index.number_of_replicas: 0  # Single node, no replicas needed
```

**Namespace-based index routing:**

```yaml
# See: https://github.com/npecka/peckacyber/blob/main/guides/elasticsearch-k8s-optimization/filebeat-values.yaml

indices:
  - index: "logs-kubernetes-%{+yyyy.MM.dd}"
    when.or:
      - not:
          has_fields: ['kubernetes.namespace']
  - index: "logs-security-%{[kubernetes.namespace]}-%{+yyyy.MM.dd}"
    when.or:
      - equals:
          kubernetes.namespace: "your-security-namespace-1"
      - equals:
          kubernetes.namespace: "your-security-namespace-2"
      - equals:
          kubernetes.namespace: "your-security-namespace-3"
  - index: "logs-app-%{[kubernetes.namespace]}-%{+yyyy.MM.dd}"
```

**Benefits:**
- **Organized indices** - Security logs separate from app logs
- **Automated rollover** - Daily indices managed automatically
- **Custom retention** - Can set different retention per namespace
- **Better performance** - Fewer indices to search across

**Applied the fix:**

```bash
# Update Filebeat configuration
helm upgrade filebeat elastic/filebeat \
  -f filebeat-values.yaml \
  -n elastic-stack

# Restart Filebeat to apply
kubectl rollout restart daemonset filebeat-filebeat -n elastic-stack
```

---

## Security Hardening

### Overview
Applied least-privilege security settings to all components.

### Pod Security Context

**Elasticsearch:**
```yaml
# File reference: elasticsearch StatefulSet spec
podSecurityContext:
  fsGroup: 1000
  runAsUser: 1000

securityContext:
  capabilities:
    drop:
      - ALL
  runAsNonRoot: true
  runAsUser: 1000
```

**Kibana:**
```yaml
# File reference: kibana Deployment spec
securityContext:
  capabilities:
    drop:
      - ALL
  runAsNonRoot: true
  runAsUser: 1000

podSecurityContext:
  fsGroup: 1000
```

### Understanding Least Privilege Security

**What we configured and why:**

1. **`runAsUser: 1000` and `runAsNonRoot: true`**
   - Forces containers to run as UID 1000 (elasticsearch user), not root (UID 0)
   - Prevents attackers from gaining root access if they compromise the container
   - Default behavior: Many containers run as root by default, which is dangerous

2. **`capabilities.drop: ALL`**
   - Linux capabilities are special privileges (like CAP_NET_ADMIN, CAP_SYS_ADMIN)
   - By default, containers get ~14 capabilities even when running as non-root
   - `drop: ALL` removes every capability, leaving only basic user permissions
   - Example: Without this, a compromised container might manipulate network interfaces or kill processes

3. **`fsGroup: 1000`**
   - Sets file system group ownership to GID 1000
   - Ensures Elasticsearch can read/write to mounted volumes without needing root
   - Volumes mounted into the pod will have group ownership set to 1000

**Real-world impact:**
If an attacker exploits a vulnerability in Elasticsearch, Kibana, or Filebeat:
- ❌ **Without these settings:** Attacker runs as root with capabilities → can escape container, access host filesystem, network manipulation
- ✅ **With these settings:** Attacker runs as unprivileged user 1000 with no capabilities → limited to application-level damage only

This is "defense in depth" - even if the application is compromised, the blast radius is minimal.

---

## Results and Key Takeaways

### Measurable Results

**Before Optimization:**
- **Filebeat:** 100+ restarts, crash loop
- **Elasticsearch:** 2Gi RAM (insufficient), OOM risk
- **Indices:** 500+ unmanaged indices, no lifecycle
- **Backup:** No strategy
- **Operations:** Manual, reactive troubleshooting

**After Implementation:**
- **Filebeat:** 0 restarts, stable
- **Elasticsearch:** 4Gi RAM, stable operation
- **Indices:** Organized by namespace with ILM
- **Backup:** Velero + MinIO automated backups to NAS
- **Operations:** Documented, reproducible

**Resource Impact:**
- **Eliminated:** 100+ crashes per week
- **Prevented:** OOM kills via proper memory allocation
- **Improved:** Query performance through ILM
- **Saved:** ~4 hours/week troubleshooting time

**System Stats (Example Deployment):**
- **Daily log volume:** ~300MB/day
- **Retention period:** 30 days
- **Storage used:** ~12GB (with Elasticsearch overhead)
- **Uptime:** Stable since fixes applied
- **Log sources:** Multiple namespaces (kube-system, security monitoring, CI/CD, application workloads, etc.)

### Key Takeaways

**1. Monitor Before Optimizing**
Running the system for 2-3 weeks revealed actual usage patterns. Initial resource guesses were 50% off - real data is essential.

**2. One Change at a Time**
Each fix was applied individually and validated. This made it easy to identify what worked and simplified rollback if needed.

**3. Context Matters More Than "Best Practices"**
The emptyDir solution isn't what production guides recommend, but it was perfect for my homelab where occasional duplicate logs are acceptable. Understand your constraints.

**4. Document Everything**
Creating detailed documentation of changes - what was changed, why, and what trade-offs were accepted - is invaluable for future troubleshooting and knowledge sharing.

**5. AI as a Force Multiplier**
Using Claude AI accelerated troubleshooting by correlating error messages across Kubernetes, Elasticsearch, and Filebeat domains simultaneously. However, I still made all decisions and validated everything in my environment.

**AI helped with:**
- Root cause analysis (correlating errors with known issues)
- Solution options (providing multiple approaches with trade-offs)
- Configuration examples (YAML snippets adapted to my context)
- Verification steps (commands to validate fixes)

**I was responsible for:**
- Gathering diagnostic data
- Evaluating trade-offs for my specific context
- Making architectural decisions
- Testing and validating all changes
- Documenting what actually worked

---

## AI Tips for Operations

### How to Use AI Tools Effectively

**1. Gather comprehensive diagnostics first:**
```bash
kubectl get pods -n <namespace>
kubectl describe pod <pod-name> -n <namespace>
kubectl logs <pod-name> -n <namespace> --tail=100
kubectl top nodes
kubectl top pods -n <namespace>
```

**2. Present specific problems, not vague questions:**
- ❌ "How do I configure Elasticsearch?"
- ✅ "My Filebeat pod is crash-looping with this error: [paste error]"

**3. Ask for trade-off analysis:**
- Request multiple solution options
- Ask about pros/cons of each
- Discuss homelab vs. production implications

**4. Validate everything:**
- Test one change at a time
- Verify it works in your environment
- Don't trust suggestions blindly

**5. Document your decisions:**
- Capture what you changed
- Explain why you chose this approach
- Note trade-offs accepted
- Include verification steps

---

## Troubleshooting

### Common Issues

**Issue 1: Elasticsearch pod stuck in pending**
- **Symptoms:** Pod shows "Pending" status indefinitely
- **Diagnosis:**
  ```bash
  kubectl describe pod elasticsearch-master-0 -n elastic-stack
  kubectl get pvc -n elastic-stack
  kubectl top nodes
  ```
- **Solutions:**
  - **If PVC not binding:** Check storage class exists and has available capacity
    ```bash
    kubectl get storageclass
    # Ensure your storage provisioner is running
    ```
  - **If insufficient node resources:** Either free up resources or reduce Elasticsearch memory requests in values file
    ```bash
    # Reduce memory in elasticsearch-values.yaml, then upgrade
    helm upgrade elasticsearch elastic/elasticsearch -f elasticsearch-values.yaml -n elastic-stack
    ```

**Issue 2: Filebeat can't connect to Elasticsearch**
- **Symptoms:** Filebeat logs show "connection refused" or TLS errors
- **Diagnosis:**
  ```bash
  kubectl logs daemonset/filebeat-filebeat -n elastic-stack --tail=50
  kubectl get secret elasticsearch-master-certs -n elastic-stack
  ```
- **Solutions:**
  - **If certificate secret missing:** Elasticsearch may not be fully deployed yet. Wait for it to create the certs secret, or recreate the Filebeat deployment
    ```bash
    kubectl rollout restart daemonset/filebeat-filebeat -n elastic-stack
    ```
  - **If wrong service name:** Verify Elasticsearch service exists and update filebeat-values.yaml if needed
    ```bash
    kubectl get svc -n elastic-stack
    # Should show: elasticsearch-master
    ```

**Issue 3: Kibana shows "Elasticsearch is not ready"**
- **Symptoms:** Kibana UI displays connection error or "waiting for Elasticsearch"
- **Diagnosis:**
  ```bash
  kubectl logs deployment/kibana-kibana -n elastic-stack --tail=50
  kubectl exec elasticsearch-master-0 -n elastic-stack -- \
    curl -k -u "elastic:${ELASTIC_PASSWORD}" https://localhost:9200/_cluster/health
  ```
- **Solutions:**
  - **If service account token missing:** Recreate the token as shown in Step 2
    ```bash
    # Create token and secret (see Step 2 for full commands)
    kubectl exec elasticsearch-master-0 -n elastic-stack -- \
      /usr/share/elasticsearch/bin/elasticsearch-service-tokens create elastic/kibana kibana-token
    ```
  - **If Elasticsearch cluster not healthy:** Wait for Elasticsearch to reach yellow/green status, or investigate Elasticsearch pod issues first
  - **If authentication fails:** Verify the token secret name matches what's referenced in the Kibana deployment

---

## Future Considerations

After completing this deployment:

1. **Customize ILM policies** - Set retention periods for your needs:
   ```bash
   # Example: Delete indices older than 30 days
   curl -k -u "elastic:${ELASTIC_PASSWORD}" -X PUT \
     "https://localhost:9200/_ilm/policy/logs_policy" -H 'Content-Type: application/json' -d'
   {
     "policy": {
       "phases": {
         "delete": {
           "min_age": "30d",
           "actions": { "delete": {} }
         }
       }
     }
   }'
   ```

2. **Set up ingress** - Expose Kibana securely:
   - Configure TLS certificates
   - Set up authentication (LDAP, SAML, etc.)
   - Restrict access by IP

3. **Enhance backup strategy** - Beyond Velero:
   - Consider Elasticsearch native snapshots for faster recovery
   - Test restore procedures regularly
   - See the upcoming dedicated backup guide for full details

4. **Consider Loki migration** - If resource-constrained:
   - Loki uses ~80% less resources for simple log aggregation
   - Trade-off: Slower full-text search vs. Elasticsearch
   - Evaluate whether full-text search speed is critical for your use case

5. **Apply to other workloads** - Use these principles for:
   - PostgreSQL, MongoDB, Redis deployments
   - Any stateful Kubernetes application
   - Resource sizing methodology applies universally

---

## References and Additional Resources

### Deployment File References

All configurations discussed in this guide are currently deployed and can be found:

**Cluster (source of truth):**
- Elasticsearch: `kubectl get statefulset elasticsearch-master -n elastic-stack -o yaml`
- Kibana: `kubectl get deployment kibana-kibana -n elastic-stack -o yaml`
- Filebeat: `kubectl get daemonset filebeat-filebeat -n elastic-stack -o yaml`
- Filebeat config: `kubectl get configmap filebeat-filebeat-daemonset-config -n elastic-stack -o yaml`

**GitHub Repository:**
All configuration files referenced in this guide are available at:
https://github.com/npecka/peckacyber/tree/main/guides/elasticsearch-k8s-optimization

- [elasticsearch-values.yaml](https://github.com/npecka/peckacyber/blob/main/guides/elasticsearch-k8s-optimization/elasticsearch-values.yaml)
- [kibana-values.yaml](https://github.com/npecka/peckacyber/blob/main/guides/elasticsearch-k8s-optimization/kibana-values.yaml)
- [filebeat-values.yaml](https://github.com/npecka/peckacyber/blob/main/guides/elasticsearch-k8s-optimization/filebeat-values.yaml)

### Additional Resources

- **Elasticsearch on Kubernetes:** https://www.elastic.co/guide/en/cloud-on-k8s/current/index.html
- **Filebeat Configuration:** https://www.elastic.co/guide/en/beats/filebeat/current/filebeat-reference-yml.html
- **ILM Policies:** https://www.elastic.co/guide/en/elasticsearch/reference/current/index-lifecycle-management.html
- **Kubernetes Best Practices:** https://kubernetes.io/docs/concepts/configuration/overview/
- **Claude AI:** https://claude.ai/

**Related Guides on This Site:**
- Coming soon: "Kubernetes Backup Strategy: Velero + MinIO for Homelabs"
- Coming soon: "Migrating from Elasticsearch to Loki for Cost Savings"
- Coming soon: "AI-Assisted Kubernetes Operations: A Workflow Guide"

### Security Considerations

**What to Share with AI Tools:**

**Safe to share:**
- Error messages
- Configuration structures (sanitized)
- Resource metrics
- Architecture diagrams

**Never share:**
- Actual passwords or API keys
- Internal IP addresses (use placeholders)
- Hostnames revealing network structure
- Sensitive log data

**Example sanitization:**
```yaml
# Shared with AI:
password: "${ELASTICSEARCH_PASSWORD}"  # Reference only

# NOT shared:
password: "actualP@ssw0rd123"
```

**Production Hardening:**

For production deployments, additionally consider:

1. **Network policies** - Restrict pod-to-pod communication
2. **RBAC** - Limit service account permissions
3. **Secrets management** - Use Vault or Sealed Secrets
4. **TLS everywhere** - Inter-node communication encryption
5. **Audit logging** - Track all Elasticsearch access
6. **Regular patching** - Keep Elastic Stack up to date

---

## Changelog

**v1.0** - October 31, 2025 - Initial release
- Complete deployment guide from initial setup through optimization
- All configurations tested and validated in homelab
- Documents real problems and proven solutions

---

**Questions or Issues?** Feel free to reach out via the contact form on peckacyber.com

**Found this helpful?** Subscribe to the newsletter for more guides.
