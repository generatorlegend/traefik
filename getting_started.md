# Getting Started with Traefik

Traefik is a modern HTTP reverse proxy and load balancer that makes deploying microservices easy. This guide will help you get started with Traefik, covering installation, basic configuration, and setting up a simple reverse proxy.

## Table of Contents

1. [Installation](#installation)
2. [Basic Configuration](#basic-configuration)
3. [Setting Up a Simple Reverse Proxy](#setting-up-a-simple-reverse-proxy)
4. [Common Use Cases](#common-use-cases)

## Installation

You can install Traefik using various methods. Here are two popular options:

### Using Docker

To run Traefik in a Docker container, use the following command:

```bash
docker run -d -p 8080:8080 -p 80:80 \
-v $PWD/traefik.yml:/etc/traefik/traefik.yml \
-v /var/run/docker.sock:/var/run/docker.sock \
traefik:v2.9
```

### Binary Installation

1. Download the latest Traefik binary for your operating system from the [official releases page](https://github.com/traefik/traefik/releases).
2. Extract the archive and move the `traefik` binary to a directory in your system's PATH.

## Basic Configuration

Traefik uses a YAML configuration file. Create a file named `traefik.yml` with the following basic configuration:

```yaml
api:
  dashboard: true
  insecure: true

entryPoints:
  web:
    address: ":80"

providers:
  docker:
    endpoint: "unix:///var/run/docker.sock"
    exposedByDefault: false
```

This configuration:
- Enables the Traefik dashboard (accessible at `http://localhost:8080/dashboard/`)
- Creates an entry point for HTTP traffic on port 80
- Enables the Docker provider to discover services

## Setting Up a Simple Reverse Proxy

Let's set up a simple reverse proxy to route traffic to a web application.

1. Create a Docker network:

```bash
docker network create traefik-network
```

2. Run a sample web application:

```bash
docker run -d --name whoami \
--network traefik-network \
--label "traefik.enable=true" \
--label "traefik.http.routers.whoami.rule=Host(`whoami.localhost`)" \
traefik/whoami
```

3. Update your `traefik.yml` to include:

```yaml
providers:
  docker:
    endpoint: "unix:///var/run/docker.sock"
    exposedByDefault: false
    network: traefik-network
```

4. Run Traefik:

```bash
docker run -d -p 8080:8080 -p 80:80 \
--network traefik-network \
-v $PWD/traefik.yml:/etc/traefik/traefik.yml \
-v /var/run/docker.sock:/var/run/docker.sock \
traefik:v2.9
```

5. Test the setup by visiting `http://whoami.localhost` in your browser. You should see the whoami service response.

## Common Use Cases

### Load Balancing

Traefik automatically load balances between multiple instances of a service. To demonstrate, scale the whoami service:

```bash
docker service scale whoami=3
```

Traefik will distribute requests among the three instances.

### SSL/TLS Configuration

To enable automatic SSL/TLS certification using Let's Encrypt:

1. Update your `traefik.yml`:

```yaml
entryPoints:
  web:
    address: ":80"
  websecure:
    address: ":443"

certificatesResolvers:
  myresolver:
    acme:
      email: your-email@example.com
      storage: acme.json
      httpChallenge:
        entryPoint: web

providers:
  docker:
    endpoint: "unix:///var/run/docker.sock"
    exposedByDefault: false
```

2. Update your service labels:

```bash
docker run -d --name secured-app \
--network traefik-network \
--label "traefik.enable=true" \
--label "traefik.http.routers.secured-app.rule=Host(`app.yourdomain.com`)" \
--label "traefik.http.routers.secured-app.tls=true" \
--label "traefik.http.routers.secured-app.tls.certresolver=myresolver" \
your-secured-app-image
```

This configuration enables automatic SSL/TLS certificate generation and renewal for your domains.

By following this guide, you should now have a basic understanding of how to get started with Traefik, including installation, configuration, and setting up a simple reverse proxy. Explore the [official Traefik documentation](https://doc.traefik.io/traefik/) for more advanced features and configurations.