# Elasticsearch on Kubernetes: Homelab Optimization Guide

This directory contains configuration files for deploying and optimizing Elasticsearch on Kubernetes in a homelab environment.

## Overview

These configurations represent a production-tested Elasticsearch stack deployment that handles 44+ million documents across multiple namespaces in a resource-constrained homelab cluster.

**What's included:**
- [GUIDE.md](./GUIDE.md) - Complete deployment and optimization guide
- Elasticsearch StatefulSet (single-node, optimized)
- Kibana visualization interface
- Filebeat log shipper (DaemonSet)

## Files

- `GUIDE.md` - Full deployment and optimization guide (blog post format)
- `README.md` - This file - quick start overview
- `elasticsearch-values.yaml` - Elasticsearch Helm values (4GB RAM, 2g heap - optimized after monitoring)
- `kibana-values.yaml` - Kibana Helm values with service account token auth
- `filebeat-values.yaml` - Filebeat Helm values with ILM, namespace routing, and emptyDir fix

**Note:** These are the final, optimized configurations. The GUIDE.md walks through the initial deployment and problems encountered before arriving at these solutions.

## Key Optimizations

### 1. Resource Sizing
- **Elasticsearch:** 4GB RAM (2GB heap + 2GB OS cache)
- **Kibana:** 2GB RAM with Node.js memory limits
- **Filebeat:** 100-200MB per pod
- **Storage:** Use formula: `(Daily Log Volume GB × Retention Days) × 1.5`
  - Example: 300MB/day × 30 days × 1.5 = ~13.5GB minimum
  - Recommended: Add 2-4x headroom for growth

### 2. Index Lifecycle Management (ILM)
- Automatic daily index rollover
- Namespace-based index routing (security vs. application logs)
- Single shard, zero replicas (single-node optimization)

### 3. Stability Fixes
- Filebeat uses `emptyDir` volumes to prevent lock file crashes
- Proper security contexts (non-root, dropped capabilities)
- Optimized health checks and probes

## Deployment

### Prerequisites
- Kubernetes cluster (any distribution)
- Helm 3.x
- At least one node with 4GB+ available RAM

### Quick Start

```bash
# Add Elastic Helm repository
helm repo add elastic https://helm.elastic.co
helm repo update

# Create namespace
kubectl create namespace elastic-stack

# Deploy Elasticsearch
helm install elasticsearch elastic/elasticsearch \
  -f elasticsearch-values.yaml \
  -n elastic-stack

# Wait for Elasticsearch to be ready
kubectl wait --for=condition=ready pod/elasticsearch-master-0 \
  -n elastic-stack --timeout=300s

# Deploy Kibana (after creating service account token - see kibana-values.yaml)
helm install kibana elastic/kibana \
  -f kibana-values.yaml \
  -n elastic-stack

# Deploy Filebeat
helm install filebeat elastic/filebeat \
  -f filebeat-values.yaml \
  -n elastic-stack

# Apply emptyDir fix for Filebeat stability
kubectl patch daemonset filebeat-filebeat -n elastic-stack \
  --type='json' \
  -p='[{"op": "replace", "path": "/spec/template/spec/volumes/3", "value": {"name": "data", "emptyDir": {}}}]'
```

## Monitoring

### Check cluster health
```bash
kubectl port-forward svc/elasticsearch-master 9200:9200 -n elastic-stack

# Get password
ELASTIC_PASSWORD=$(kubectl get secret elasticsearch-master-credentials \
  -n elastic-stack -o jsonpath='{.data.password}' | base64 -d)

# Check health
curl -k -u "elastic:${ELASTIC_PASSWORD}" \
  https://localhost:9200/_cluster/health?pretty
```

### View resource usage
```bash
kubectl top nodes
kubectl top pods -n elastic-stack
```

## Full Guide

For the complete walkthrough with troubleshooting, monitoring, and optimization details, see [GUIDE.md](./GUIDE.md).

Also published at: [peckacyber.com/guides/elasticsearch-k8s-optimization](https://peckacyber.com)

## License

These configurations are provided as-is for educational purposes. Modify for your own use.

## Contributing

Found an issue or improvement? Open an issue or PR on this repository.
