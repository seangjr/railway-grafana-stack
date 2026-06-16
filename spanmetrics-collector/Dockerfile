FROM otel/opentelemetry-collector-contrib:latest

COPY otel-config.yaml /etc/otel-collector-config.yaml

EXPOSE 4317 4318 13133

CMD ["--config=/etc/otel-collector-config.yaml"]
