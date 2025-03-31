# Configuration Reference

This document provides a comprehensive reference for all configuration options in Traefik, including both static and dynamic configurations. Each option is explained in detail, along with its purpose and examples of usage.

## Static Configuration

Static configuration is the global configuration for Traefik. It is set up when Traefik starts and cannot be changed dynamically.

### Global Configuration

The `Global` section contains global configuration options for Traefik.

```toml
[global]
  checkNewVersion = true
  sendAnonymousUsage = true
```

- `checkNewVersion`: Periodically check if a new version of Traefik has been released. (Default: `true`)
- `sendAnonymousUsage`: Send anonymous usage statistics. If not specified, it will be disabled by default. (Default: `false`)

### EntryPoints

EntryPoints define the network entry points into Traefik.

```toml
[entryPoints]
  [entryPoints.web]
    address = ":80"

  [entryPoints.websecure]
    address = ":443"
```

- `address`: Network address to listen on. (Required)

### Providers

Providers are responsible for discovering the services that will be exposed through Traefik.

```toml
[providers]
  [providers.docker]
    endpoint = "unix:///var/run/docker.sock"
    exposedByDefault = false

  [providers.file]
    directory = "/path/to/dynamic/conf"
```

#### Docker Provider

- `endpoint`: Docker server endpoint. Can be a TCP or a Unix socket endpoint. (Default: "unix:///var/run/docker.sock")
- `exposedByDefault`: Expose containers by default. (Default: `true`)

#### File Provider

- `directory`: Load dynamic configuration from one or more .toml or .yml files in a directory.

### API and Dashboard

The API and Dashboard configuration allows you to enable and configure Traefik's built-in API and web UI.

```toml
[api]
  insecure = true
  dashboard = true
```

- `insecure`: Enable API directly on the entryPoint named traefik. (Default: `false`)
- `dashboard`: Enable the dashboard. (Default: `true`)

### Log Configuration

Configure Traefik's logging system.

```toml
[log]
  level = "INFO"
```

- `level`: Log level. Possible values are: "DEBUG", "INFO", "WARN", "ERROR", "FATAL", "PANIC". (Default: "ERROR")

### Access Logs

Configure access logs to track HTTP requests.

```toml
[accessLog]
  filePath = "/path/to/access.log"
  format = "common"
```

- `filePath`: Path to the log file. If not specified, logs are written to stdout.
- `format`: Access log format. Can be either "common" or "json". (Default: "common")

### Tracing

Configure distributed tracing system integration.

```toml
[tracing]
  serviceName = "traefik"
  
  [tracing.otlp]
    [tracing.otlp.grpc]
      endpoint = "localhost:4317"
```

- `serviceName`: Set the name for this service in the tracing system.
- `otlp`: Configure OpenTelemetry (OTLP) tracing.

## Dynamic Configuration

Dynamic configuration can be changed at runtime, without restarting Traefik.

### HTTP Configuration

Configure HTTP routing and middlewares.

```yaml
http:
  routers:
    my-router:
      rule: "Host(`example.com`)"
      service: service-foo
  
  services:
    service-foo:
      loadBalancer:
        servers:
          - url: "http://foo-service:5000"
```

- `routers`: Define rules to route requests.
- `services`: Define how to reach the actual services.

### TCP Configuration

Configure TCP routing.

```yaml
tcp:
  routers:
    my-tcp-router:
      rule: "HostSNI(`example.com`)"
      service: tcp-service-foo
  
  services:
    tcp-service-foo:
      loadBalancer:
        servers:
          - address: "foo-service:5000"
```

- `routers`: Define rules to route TCP requests.
- `services`: Define how to reach the actual TCP services.

### TLS Configuration

Configure TLS for secure communication.

```yaml
tls:
  certificates:
    - certFile: "/path/to/domain.cert"
      keyFile: "/path/to/domain.key"
  
  options:
    default:
      minVersion: "VersionTLS12"
```

- `certificates`: List of certificate-key pairs.
- `options`: Define TLS options like minimum TLS version.

This configuration reference covers the main aspects of Traefik's configuration. For more detailed information on specific features or advanced usage, please refer to the official Traefik documentation.