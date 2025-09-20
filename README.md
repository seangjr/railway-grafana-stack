# Monitoring Stack

This repository contains configuration for monitoring infrastructure components including OpenTelemetry Collector, Grafana, Prometheus, Loki, and Tempo.

## OpenTelemetry Collector

### Local Development

To build the OpenTelemetry Collector Docker image locally, run from the root directory:

```bash
docker build -f otel-collector/Dockerfile -t otel-collector .
```

**Important**: Build from the root directory (not from inside `otel-collector/`) because:
- The Dockerfile references `otel-collector/otel-config.yaml`
- Railway builds from the root context as specified in `otel-collector/railway.json`
- This ensures consistency between local builds and Railway deployments

### Deployment

The service is configured for deployment on Railway with the configuration in `otel-collector/railway.json`.