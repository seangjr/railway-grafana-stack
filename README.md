# Self-Hosted Observability Stack

A complete, production-ready observability stack using OpenTelemetry, Grafana, Prometheus, Loki, and Tempo. This stack provides unified monitoring for logs, traces, and metrics from your applications.

## Quick Deploy

Deploy the entire observability stack to Railway with one click:

[![Deploy on Railway](https://railway.app/button.svg)](https://railway.app/template/your-template-slug)

Or manually deploy using the Railway CLI:
```bash
git clone https://github.com/biancarosa/railway-grafana-stack.git
cd railway-grafana-stack
railway up
```

## Architecture

```
┌─────────────────────────────────┐
│   Your Application              │
│   (with OTEL instrumentation)   │
└────────────┬────────────────────┘
             │ OTLP/HTTP (4318)
             │ OTLP/gRPC (4317)
             ▼
┌─────────────────────────────────┐
│   OpenTelemetry Collector       │
│   - Receives telemetry          │
│   - Batches and processes       │
│   - Routes to backends          │
└─────────────┬───────────────────┘
              │
    ┌─────────┼─────────┐
    │         │         │
    ▼         ▼         ▼
┌──────┐ ┌──────┐ ┌────────────┐
│ Loki │ │Tempo │ │Prometheus  │
│(Logs)│ │(Trace│ │(Metrics)   │
└──┬───┘ └──┬───┘ └─────┬──────┘
   │        │           │
   └────────┴───────────┘
            │
            ▼
    ┌───────────────┐
    │   Grafana     │
    │  Dashboards   │
    └───────────────┘
```

## Components

### 1. OpenTelemetry Collector
- **Purpose**: Central telemetry receiver and processor
- **Receives**: OTLP over HTTP (4318) and gRPC (4317)
- **Exports to**: Tempo (traces), Prometheus (metrics), Loki (logs)
- **Config**: `otel-collector/otel-config.yaml`

### 2. Grafana
- **Purpose**: Unified visualization and dashboards
- **Version**: 11.5.2 (OSS)
- **Features**: Pre-provisioned datasources for Loki, Tempo, and Prometheus
- **Config**: `grafana/datasources/datasources.yml`

### 3. Prometheus
- **Purpose**: Metrics storage and querying
- **Receives**: Metrics from OTEL Collector via remote write
- **Config**: `prometheus/prom.yml`

### 4. Loki
- **Purpose**: Log aggregation and storage
- **Receives**: Logs from OTEL Collector via OTLP HTTP
- **Config**: `loki/loki.yml`

### 5. Tempo
- **Purpose**: Distributed tracing backend
- **Receives**: Traces from OTEL Collector via OTLP
- **Config**: `tempo/tempo.yml`

## Deployment

### Railway (Recommended)

Each service includes a `railway.json` configuration file for easy deployment:

1. **Deploy OTEL Collector**:
   ```bash
   cd otel-collector
   railway up
   ```

2. **Deploy Tempo, Loki, Prometheus**:
   ```bash
   railway up
   ```

3. **Deploy Grafana**:
   ```bash
   cd grafana
   railway up
   ```

### Environment Variables

Set these in your Railway dashboard:

#### OTEL Collector
```bash
TEMPO_ENDPOINT=http://tempo.railway.internal:3200
LOKI_ENDPOINT=http://loki.railway.internal:3100/otlp
PROMETHEUS_ENDPOINT=http://prometheus.railway.internal:9090
```

#### Grafana
```bash
GF_SECURITY_ADMIN_USER=admin
GF_SECURITY_ADMIN_PASSWORD=<your-secure-password>
GF_AUTH_ANONYMOUS_ENABLED=false
LOKI_INTERNAL_URL=http://loki.railway.internal:3100
PROMETHEUS_INTERNAL_URL=http://prometheus.railway.internal:9090
TEMPO_INTERNAL_URL=http://tempo.railway.internal:3200
```

## Local Development

### Build OTEL Collector Locally

To build the OpenTelemetry Collector Docker image locally, run from the root directory:

```bash
docker build -f otel-collector/Dockerfile -t otel-collector .
```

**Important**: Build from the root directory (not from inside `otel-collector/`) because:
- The Dockerfile references `otel-collector/otel-config.yaml`
- Railway builds from the root context as specified in `otel-collector/railway.json`
- This ensures consistency between local builds and Railway deployments

### Testing Locally with Docker Compose

You can test the full stack locally (coming soon - docker-compose.yml).

## Instrumenting Your Application

### Python/Flask Example

```python
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.sdk.metrics import MeterProvider
from opentelemetry.sdk._logs import LoggerProvider
from opentelemetry.exporter.otlp.proto.http.trace_exporter import OTLPSpanExporter
from opentelemetry.exporter.otlp.proto.http.metric_exporter import OTLPMetricExporter
from opentelemetry.exporter.otlp.proto.http._log_exporter import OTLPLogExporter
from opentelemetry.instrumentation.flask import FlaskInstrumentor
from opentelemetry import trace, metrics

# Configure endpoint
endpoint = "http://your-otel-collector:4318"

# Set up providers
trace.set_tracer_provider(TracerProvider())
metrics.set_meter_provider(MeterProvider())

# Add exporters
trace.get_tracer_provider().add_span_processor(
    BatchSpanProcessor(OTLPSpanExporter(endpoint=f"{endpoint}/v1/traces"))
)

# Auto-instrument Flask
FlaskInstrumentor().instrument_app(app)
```

### Environment Variables for Applications

```bash
OTEL_EXPORTER_OTLP_ENDPOINT=http://opentelemetry-collector-contrib.railway.internal:4318
OTEL_SERVICE_NAME=your-service-name
OTEL_EXPORTER_OTLP_PROTOCOL=http/protobuf
```

## Security Best Practices

### Network Isolation

1. **Internal services**: OTEL Collector, Prometheus, Loki, and Tempo should only be accessible via Railway's internal network (`.railway.internal`)
2. **Public access**: Only Grafana should have a public URL
3. **Application → OTEL**: Applications send telemetry using internal domains

### Authentication

1. **Grafana**: Set strong admin credentials via environment variables
   ```bash
   GF_SECURITY_ADMIN_USER=admin
   GF_SECURITY_ADMIN_PASSWORD=<use-railway-secrets>
   GF_AUTH_ANONYMOUS_ENABLED=false
   ```

2. **OTEL Collector**: Consider adding authentication for production deployments (see OTEL auth extension)

### Secrets Management

- **Never commit credentials** to git
- Use Railway's environment variables for all sensitive values
- Rotate passwords regularly

## Troubleshooting

### No traces appearing in Tempo

1. Check OTEL Collector logs:
   ```bash
   railway logs --service "OTEL Collector"
   ```

2. Verify your application has `TracerProvider` configured
3. Check endpoint configuration matches Railway internal URL

### No logs in Loki

1. Verify `LoggerProvider` is configured in your application
2. Check OTEL Collector is receiving logs on `/v1/logs` endpoint
3. Verify Loki endpoint in OTEL config: `http://loki.railway.internal:3100/otlp`

### No metrics in Prometheus

1. Verify `MeterProvider` is configured in your application
2. Check Prometheus remote write endpoint in OTEL config
3. Check Prometheus logs for write errors

### Grafana datasources not loading

1. Verify environment variables are set correctly
2. Check internal URLs are using `.railway.internal` domains
3. Restart Grafana service after changing environment variables

## Configuration Files

```
railway-grafana-stack/
├── railway.json              # Railway template for one-click deploy
├── otel-collector/
│   ├── Dockerfile
│   ├── otel-config.yaml      # OTLP receivers and exporters
│   └── railway.json
├── grafana/
│   ├── dockerfile
│   ├── datasources/
│   │   └── datasources.yml   # Pre-configured datasources
│   └── railway.json
├── prometheus/
│   ├── dockerfile
│   ├── prom.yml              # Scrape configs
│   └── railway.json
├── loki/
│   ├── dockerfile
│   ├── loki.yml              # Loki configuration
│   └── railway.json
└── tempo/
    ├── dockerfile
    ├── tempo.yml             # Tempo configuration
    └── railway.json
```

## Related Resources

- [OpenTelemetry Documentation](https://opentelemetry.io/docs/)
- [Grafana Documentation](https://grafana.com/docs/)
- [Tempo Documentation](https://grafana.com/docs/tempo/latest/)
- [Loki Documentation](https://grafana.com/docs/loki/latest/)
- [Prometheus Documentation](https://prometheus.io/docs/)

## License

See [LICENSE](LICENSE) file for details.

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.