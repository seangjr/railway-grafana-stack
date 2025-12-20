# Self-Hosted Observability Stack

A complete, production-ready observability stack using OpenTelemetry, Grafana, Prometheus, Loki, and Tempo. This stack provides unified monitoring for logs, traces, and metrics from your applications.

## Quick Deploy

Deploy the entire observability stack to Railway with one click:

[![Deploy on Railway](https://railway.com/button.svg)](https://railway.com/deploy/mPYvR2?referralCode=sWx4lg&utm_medium=integration&utm_source=template&utm_campaign=generic)


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

### Railway

Each service includes a `railway.json` configuration file for easy deployment.

## Security Best Practices

### Network Isolation

1. **Internal services**: OTEL Collector, Prometheus, Loki, and Tempo should only be accessible via Railway's internal network (`.railway.internal`)
2. **Public access**: Only Grafana should have a public URL, if needed.
3. **Application → OTEL**: Applications send telemetry using internal domains

### Authentication

1. **Grafana**: Set strong admin credentials via environment variables
   ```bash
   GF_SECURITY_ADMIN_USER=admin
   GF_SECURITY_ADMIN_PASSWORD=<use-railway-secrets>
   GF_AUTH_ANONYMOUS_ENABLED=false
   ```

2. **OTEL Collector**: Consider adding authentication for production deployments (see OTEL auth extension)

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
