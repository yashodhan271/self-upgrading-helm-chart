# Configuration Guide

This document provides detailed information about configuring the Self-Upgrading Helm Chart.

## Table of Contents

- [Basic Configuration](#basic-configuration)
- [Update Detection](#update-detection)
- [Deployment Strategies](#deployment-strategies)
- [Monitoring](#monitoring)
- [Resource Management](#resource-management)

## Basic Configuration

The chart can be configured using the following values in your `values.yaml`:

```yaml
# Basic chart configuration
nameOverride: ""
fullnameOverride: ""

# Image configuration
image:
  repository: nginx
  tag: ""
  pullPolicy: IfNotPresent

# Service configuration
service:
  type: ClusterIP
  port: 80
```

## Update Detection

Configure how the chart detects and manages updates:

```yaml
updateDetection:
  enabled: true
  interval: 300  # Check every 5 minutes
  sources:
    - type: container
      registry: docker.io
      repositories:
        - name: nginx
          tag: latest
    - type: helm
      repositories:
        - name: bitnami
          url: https://charts.bitnami.com/bitnami
  notifications:
    slack:
      enabled: false
      webhook: ""
    email:
      enabled: false
      recipients: []
```

## Deployment Strategies

### Canary Deployment

```yaml
deployment:
  strategy: canary
  steps:
    - percentage: 25
      interval: 300
      metrics:
        - name: error_rate
          threshold: 1
    - percentage: 50
      interval: 300
    - percentage: 75
      interval: 300
```

### Blue-Green Deployment

```yaml
deployment:
  strategy: blueGreen
  activeService: blue
  transitionDelay: 300
  autoPromote: false
```

## Monitoring

Configure monitoring integrations:

```yaml
monitoring:
  prometheus:
    enabled: true
    serviceMonitor:
      enabled: true
      interval: 30s
  
  grafana:
    enabled: true
    dashboards:
      enabled: true
```

## Resource Management

Configure resource limits and requests:

```yaml
resources:
  limits:
    cpu: 1000m
    memory: 1Gi
  requests:
    cpu: 500m
    memory: 512Mi

# Pod Security Context
securityContext:
  runAsNonRoot: true
  runAsUser: 1000
  fsGroup: 2000
```

## Advanced Configuration

### Custom Metrics

```yaml
metrics:
  custom:
    - name: application_error_rate
      query: sum(rate(http_requests_total{status=~"5.."}[5m])) / sum(rate(http_requests_total[5m]))
      threshold: 0.01
    - name: latency_p95
      query: histogram_quantile(0.95, sum(rate(http_request_duration_seconds_bucket[5m])) by (le))
      threshold: 0.5
```

### Health Checks

```yaml
healthCheck:
  enabled: true
  path: /health
  port: 8080
  initialDelaySeconds: 30
  periodSeconds: 10
  failureThreshold: 3
  successThreshold: 1
```

For more detailed information about specific configurations, please refer to our [examples](../examples) directory.
