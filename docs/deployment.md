# Deployment Guide

This guide explains how to deploy and manage applications using the Self-Upgrading Helm Chart.

## Table of Contents

- [Prerequisites](#prerequisites)
- [Installation](#installation)
- [Deployment Strategies](#deployment-strategies)
- [Monitoring and Maintenance](#monitoring-and-maintenance)
- [Troubleshooting](#troubleshooting)

## Prerequisites

Before deploying, ensure you have:

1. Kubernetes cluster (v1.19+)
2. Helm 3.0+
3. kubectl configured with cluster access
4. Prometheus Operator (optional, for monitoring)

## Installation

### Basic Installation

```bash
# Add the Helm repository
helm repo add self-upgrading https://charts.example.com/self-upgrading

# Update your repositories
helm repo update

# Install with default configuration
helm install my-release self-upgrading/self-upgrading-chart
```

### Custom Configuration

```bash
# Install with custom values file
helm install my-release self-upgrading/self-upgrading-chart -f values.yaml

# Install with specific values
helm install my-release self-upgrading/self-upgrading-chart \
  --set updateDetection.enabled=true \
  --set deployment.strategy=canary
```

## Deployment Strategies

### Canary Deployment

Canary deployment gradually rolls out changes to a subset of users:

```yaml
deployment:
  strategy: canary
  steps:
    - percentage: 25  # Start with 25% of traffic
      interval: 300   # Wait 5 minutes
      metrics:        # Monitor these metrics
        - name: error_rate
          threshold: 1
    - percentage: 50  # Increase to 50% if successful
      interval: 300
    - percentage: 75  # Then to 75%
      interval: 300
    - percentage: 100 # Finally, full deployment
```

### Blue-Green Deployment

Blue-Green deployment maintains two identical environments:

```yaml
deployment:
  strategy: blueGreen
  activeService: blue
  transitionDelay: 300
  autoPromote: false
```

## Monitoring and Maintenance

### Health Checks

Monitor application health:

```bash
# Get deployment status
kubectl get deployments

# Check pod health
kubectl get pods -l app=my-release

# View logs
kubectl logs -l app=my-release
```

### Prometheus Metrics

Access metrics through Prometheus:

1. Port-forward Prometheus:
```bash
kubectl port-forward svc/prometheus-operated 9090:9090
```

2. Access the Prometheus UI at `http://localhost:9090`

### Grafana Dashboards

View metrics in Grafana:

1. Port-forward Grafana:
```bash
kubectl port-forward svc/grafana 3000:3000
```

2. Access Grafana at `http://localhost:3000`

## Troubleshooting

### Common Issues

1. **Pods Not Starting**
```bash
# Check pod status
kubectl describe pod <pod-name>

# Check logs
kubectl logs <pod-name>
```

2. **Update Detection Issues**
```bash
# Check update controller logs
kubectl logs -l app=update-controller

# Check configuration
kubectl get configmap my-release-config -o yaml
```

3. **Rollback Issues**
```bash
# List revisions
helm history my-release

# Rollback to previous version
helm rollback my-release 1
```

### Getting Help

If you encounter issues:

1. Check the [troubleshooting guide](troubleshooting.md)
2. Review the [FAQ](faq.md)
3. [Open an issue](https://github.com/yashodhan271/self-upgrading-helm-chart/issues)
