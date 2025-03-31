---
title: "Observability Guide"
description: "Learn how to set up monitoring, logging, and tracing in Traefik"
---

# Traefik Observability Guide

This guide will help you set up monitoring, logging, and tracing in Traefik, integrate with popular observability tools, and interpret Traefik's metrics.

## Table of Contents

1. [Metrics](#metrics)
2. [Tracing](#tracing)
3. [Logging](#logging)
4. [Integration with Observability Tools](#integration-with-observability-tools)
5. [Interpreting Traefik Metrics](#interpreting-traefik-metrics)

## Metrics

Traefik provides a comprehensive metrics system that allows you to monitor various aspects of your proxy server. The metrics are exposed through a `Registry` interface, which can be implemented by different metric backends.

### Available Metrics

Traefik offers metrics for the following components:

- Server metrics (e.g., config reloads, open connections)
- TLS metrics
- Entry point metrics
- Router metrics
- Service metrics

### Setting up Metrics

To enable metrics in Traefik, you need to configure a metrics backend. Traefik supports various backends, including Prometheus, Datadog, and InfluxDB. Here's an example of how to set up Prometheus metrics:

```yaml
metrics:
  prometheus:
    buckets:
      - 0.1
      - 0.3
      - 1.2
      - 5.0
    addEntryPointsLabels: true
    addServicesLabels: true
    entryPoint: metrics
```

This configuration will expose Prometheus metrics on the `metrics` entrypoint.

## Tracing

Traefik supports distributed tracing, which allows you to track requests as they flow through your system. The tracing system is based on OpenTelemetry.

### Setting up Tracing

To enable tracing in Traefik, you need to configure a tracing backend. Traefik supports OpenTelemetry (OTLP) as the tracing backend. Here's an example configuration:

```yaml
tracing:
  otlp:
    endpoint: "localhost:4317"
    insecure: true
  serviceName: "traefik"
  sampleRate: 1.0
```

This configuration sets up OpenTelemetry tracing with the following parameters:
- `endpoint`: The address of your OTLP collector
- `insecure`: Whether to use an insecure connection (set to `false` for production)
- `serviceName`: The name of your Traefik service in traces
- `sampleRate`: The rate at which to sample traces (1.0 means all traces)

### Customizing Trace Information

Traefik allows you to customize the information captured in traces:

```yaml
tracing:
  otlp:
    # ... other OTLP configuration
  serviceName: "traefik"
  sampleRate: 1.0
  capturedRequestHeaders:
    - "User-Agent"
    - "Authorization"
  capturedResponseHeaders:
    - "Content-Type"
    - "Content-Length"
```

This configuration will include the specified request and response headers in your traces.

## Logging

Traefik provides detailed logging capabilities to help you monitor and troubleshoot your proxy server.

### Log Levels

Traefik supports the following log levels:

- DEBUG
- INFO
- WARN
- ERROR
- FATAL
- PANIC

### Configuring Logging

You can configure logging in your Traefik static configuration:

```yaml
log:
  level: INFO
  format: json
  filePath: "/path/to/traefik.log"
```

This configuration sets the log level to INFO, uses JSON format, and writes logs to the specified file.

## Integration with Observability Tools

Traefik can be integrated with various observability tools to provide a comprehensive view of your system's performance and health.

### Prometheus and Grafana

1. Set up Prometheus metrics as shown in the [Metrics](#metrics) section.
2. Configure Prometheus to scrape Traefik's metrics endpoint.
3. Use Grafana to create dashboards visualizing Traefik's metrics.

### OpenTelemetry Collector

1. Set up OpenTelemetry tracing as shown in the [Tracing](#tracing) section.
2. Configure an OpenTelemetry Collector to receive traces from Traefik.
3. Export traces from the collector to your preferred tracing backend (e.g., Jaeger, Zipkin).

### ELK Stack (Elasticsearch, Logstash, Kibana)

1. Configure Traefik to output logs in JSON format.
2. Use Filebeat or Logstash to ship logs to Elasticsearch.
3. Create Kibana dashboards to visualize and analyze Traefik logs.

## Interpreting Traefik Metrics

Traefik exposes a wide range of metrics that can help you understand the performance and behavior of your proxy server. Here are some key metrics to monitor:

1. `traefik_config_reloads_total`: The total number of configuration reloads.
2. `traefik_config_last_reload_success`: Indicates if the last config reload was successful (1 for success, 0 for failure).
3. `traefik_open_connections`: The current number of open connections.
4. `traefik_entrypoint_requests_total`: Total number of HTTP requests through an entrypoint.
5. `traefik_entrypoint_request_duration_seconds`: Histogram of request duration through an entrypoint.
6. `traefik_router_requests_total`: Total number of HTTP requests routed.
7. `traefik_service_requests_total`: Total number of HTTP requests made to a service.
8. `traefik_service_request_duration_seconds`: Histogram of request duration for a service.

When analyzing these metrics, pay attention to:

- Sudden spikes in request counts or durations
- Increases in configuration reload frequency
- Changes in the number of open connections
- Differences in performance between services or routers

By monitoring these metrics and setting up appropriate alerts, you can ensure the smooth operation of your Traefik instance and quickly identify any issues that may arise.

Remember to adjust your monitoring strategy based on your specific use case and requirements. Regular review and refinement of your observability setup will help you maintain optimal performance and reliability of your Traefik deployment.