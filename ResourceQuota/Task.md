# Kubernetes ResourceQuota Configuration

This document outlines the configuration for a `ResourceQuota` in the `dev` namespace. A `ResourceQuota` helps manage and limit the resources (CPU, memory, and pods) used within a given namespace.

## Example ResourceQuota YAML

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: example-quota
  namespace: dev
spec:
  hard:
    requests.cpu: "2"
    requests.memory: 4Gi
    limits.cpu: "4"
    limits.memory: 8Gi
    pods: "10"
