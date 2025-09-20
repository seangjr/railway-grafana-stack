FROM otel/opentelemetry-collector-contrib:latest

# Copy the configuration file
COPY otel-collector/otel-config.yaml /etc/otel-collector-config.yaml

# Expose the necessary ports
EXPOSE 4317 4318 13133

# Start the collector with the configuration
CMD ["--config=/etc/otel-collector-config.yaml"]