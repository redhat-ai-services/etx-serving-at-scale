# Observability Stack Deployment Instructions

This guide provides step-by-step instructions for deploying the complete observability stack for ETX Serving at Scale, including Grafana dashboards for GPU monitoring and vLLM metrics.

## Prerequisites

- OpenShift cluster with admin privileges
- NVIDIA GPU Operator installed and running
- `kubectl` or `oc` CLI tool configured

## Deployment Steps

### Step 1: Deploy Core Observability Stack

Deploy the main observability infrastructure including Grafana, Prometheus, Tempo, and OpenTelemetry collectors:

```bash
kubectl apply -f manifests/observability/observability.yaml
```

**Expected behavior:**
- All basic Kubernetes resources will be created immediately
- Operator subscriptions will be installed and may take 3-5 minutes to become ready
- The OpenTelemetryCollector resources will initially fail because the operators are still installing

### Step 2: Wait for Operators to Install

Monitor the operator installation progress:

```bash
# Check operator status
kubectl get csv -n openshift-grafana-operator
kubectl get csv -n openshift-opentelemetry-operator
kubectl get csv -n openshift-tempo-operator

# Check if CRDs are available
kubectl get crd | grep -E "(grafana|opentelemetry|tempo)"
```

Wait until all operators show `Succeeded` status before proceeding.

### Step 3: Reapply Configuration (if needed)

After operators are ready, reapply the configuration to create custom resources:

```bash
kubectl apply -f manifests/observability/observability.yaml
```

### Step 4: Verify Core Stack

Check that all pods are running in the observability-hub namespace:

```bash
kubectl get pods -n observability-hub
```

Expected pods:
- `grafana-*` - Grafana instance
- `otel-collector-*` - OpenTelemetry collector
- `minio-tempo-*` - MinIO storage for Tempo
- `tempo-*` - Tempo distributed tracing components

### Step 5: Deploy Dashboards

Deploy the specialized dashboards for GPU and vLLM monitoring:

```bash
# Deploy NVIDIA GPU monitoring dashboard
kubectl apply -f manifests/observability/nvidia-dashboard.yaml

# Deploy vLLM performance dashboard
kubectl apply -f manifests/observability/vllm-dashboard.yaml
```

### Step 6: Access Grafana

Get the Grafana route URL:

```bash
kubectl get route grafana-route -n observability-hub -o jsonpath='{.spec.host}'
```

Default credentials:
- Username: `rhel`
- Password: `rhel`

## Configuration Notes

### Namespace Configuration

The monitoring configuration is currently tuned for the `vllm` namespace. To monitor models in different namespaces:

1. Edit `manifests/observability/observability.yaml`
2. Update the PodMonitor resources (lines 750-790) to include your target namespaces
3. Reapply the configuration

### GPU Metrics

The NVIDIA dashboard requires:
- NVIDIA GPU Operator with DCGM exporter enabled
- ServiceMonitor for `nvidia-dcgm-exporter` (automatically created by GPU operator)

### Custom Models

To monitor custom vLLM models, update the PodMonitor selectors in the observability configuration to include your model names.

## Troubleshooting

### Common Issues

1. **OpenTelemetryCollector pods failing**
   - Ensure operators are fully installed before reapplying configuration
   - Check operator logs: `kubectl logs -n openshift-opentelemetry-operator deployment/opentelemetry-product-operator`

2. **Missing GPU metrics**
   - Verify NVIDIA GPU Operator is running: `kubectl get pods -n nvidia-gpu-operator`
   - Check DCGM exporter: `kubectl get svc nvidia-dcgm-exporter -n nvidia-gpu-operator`

3. **Dashboard not showing data**
   - Verify Prometheus is scraping metrics
   - Check ServiceMonitor configurations
   - Ensure target applications are exposing metrics

### Validation Commands

```bash
# Check if Grafana is accessible
kubectl get route grafana-route -n observability-hub

# Verify metrics collection
kubectl get servicemonitor -A | grep -E "(nvidia|vllm)"

# Check Prometheus targets (from Grafana or port-forward to Prometheus)
kubectl port-forward -n openshift-monitoring svc/prometheus-k8s 9090:9090
```

## References

- [RHOAI User Workload Monitoring Dashboards](https://github.com/rh-aiservices-bu/rhoai-uwm/tree/main)
- [LLS Observability Stack](https://github.com/rh-ai-kickstart/lls-observability)
- [OpenShift Monitoring Documentation](https://docs.openshift.com/container-platform/latest/monitoring/index.html)
