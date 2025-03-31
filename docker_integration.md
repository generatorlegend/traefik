# Docker Integration with Traefik

This guide explains how to use Traefik with Docker, including setting up Traefik as a reverse proxy for Docker containers, using Docker labels for configuration, and managing dynamic service discovery.

## Table of Contents

1. [Introduction](#introduction)
2. [Setting Up Traefik as a Reverse Proxy](#setting-up-traefik-as-a-reverse-proxy)
3. [Using Docker Labels for Configuration](#using-docker-labels-for-configuration)
4. [Dynamic Service Discovery](#dynamic-service-discovery)
5. [Advanced Configuration](#advanced-configuration)

## Introduction

Traefik is a modern HTTP reverse proxy and load balancer that makes deploying microservices easy. It integrates seamlessly with Docker, allowing you to dynamically route traffic to your containers without manual configuration.

## Setting Up Traefik as a Reverse Proxy

To set up Traefik as a reverse proxy for your Docker containers, follow these steps:

1. Create a `docker-compose.yml` file with the following content:

```yaml
version: '3'

services:
  traefik:
    image: traefik:v2.5
    command:
      - "--api.insecure=true"
      - "--providers.docker=true"
      - "--providers.docker.exposedbydefault=false"
      - "--entrypoints.web.address=:80"
    ports:
      - "80:80"
      - "8080:8080"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro

  whoami:
    image: traefik/whoami
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.whoami.rule=Host(`whoami.localhost`)"
      - "traefik.http.routers.whoami.entrypoints=web"
```

2. Run the following command to start Traefik and the example `whoami` service:

```bash
docker-compose up -d
```

3. Access the `whoami` service at `http://whoami.localhost` and the Traefik dashboard at `http://localhost:8080`.

## Using Docker Labels for Configuration

Traefik uses Docker labels to configure routing and other settings for your containers. Here are some common label examples:

- Enable Traefik for a container:
```
traefik.enable=true
```

- Set a routing rule:
```
traefik.http.routers.<router_name>.rule=Host(`example.com`)
```

- Specify the entrypoint:
```
traefik.http.routers.<router_name>.entrypoints=web
```

- Set a service port:
```
traefik.http.services.<service_name>.loadbalancer.server.port=8080
```

## Dynamic Service Discovery

Traefik automatically discovers new containers and updates its configuration accordingly. This dynamic service discovery allows you to scale your applications easily without manual intervention.

To take advantage of this feature:

1. Ensure that the Docker provider is enabled in your Traefik configuration.
2. Use appropriate labels on your Docker containers to configure routing and other settings.
3. Start or stop containers as needed, and Traefik will automatically update its configuration.

## Advanced Configuration

For more advanced configurations, you can:

- Use a `traefik.yml` file to define static configuration options.
- Implement SSL/TLS termination using Let's Encrypt.
- Set up middleware for additional functionality like authentication or rate limiting.

For detailed information on these advanced topics, please refer to the official Traefik documentation.