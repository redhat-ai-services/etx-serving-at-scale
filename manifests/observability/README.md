# Run book

1. kubectl apply -f manifests/observability/observability.yaml --dry-run=none   
- wait for all the resources to come up. The OpenTelemetryCollector will fail because the operator is being installed. After 3-5 minute rerun and in the observability-hub namespace it will populate the otel-collector pod. 

- The cluster is now configured to deploy the dashboards 

2. kubectl apply -f manifests/observability/nvidia-dashboard.yaml -n observability-hub

3. kubectl apply -f manifests/observability/vllm-dashboard.yaml -n observability-hub

IMPORTANT: The dashboard is tuned to the vllm namepsace and models. You will need to modify the observability.yaml if you to include custom namespaces 