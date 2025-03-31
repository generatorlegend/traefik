---
title: "TLS and HTTPS Configuration"
description: "Learn how to set up HTTPS and manage TLS certificates in Traefik, including automatic certificate generation with Let's Encrypt and manual certificate configuration."
---

# TLS and HTTPS Configuration in Traefik

This guide explains how to set up HTTPS and manage TLS certificates in Traefik. We'll cover both automatic certificate generation using Let's Encrypt and manual certificate configuration.

## Table of Contents

1. [Introduction](#introduction)
2. [Automatic Certificate Generation with Let's Encrypt](#automatic-certificate-generation-with-lets-encrypt)
3. [Manual Certificate Configuration](#manual-certificate-configuration)
4. [TLS Options](#tls-options)
5. [Common Scenarios](#common-scenarios)

## Introduction

Traefik supports TLS (Transport Layer Security) for secure HTTPS connections. You can configure Traefik to use certificates from various sources, including automatic generation through Let's Encrypt or manual configuration using your own certificates.

## Automatic Certificate Generation with Let's Encrypt

Traefik can automatically generate and renew TLS certificates using Let's Encrypt. To enable this feature, you need to configure the ACME (Automated Certificate Management Environment) provider.

Here's an example configuration:

```yaml
certificatesResolvers:
  myresolver:
    acme:
      email: your-email@example.com
      storage: acme.json
      httpChallenge:
        entryPoint: web
```

Then, in your router configuration, specify the certificate resolver:

```yaml
http:
  routers:
    my-router:
      rule: "Host(`example.com`)"
      entryPoints:
        - websecure
      tls:
        certResolver: myresolver
```

This setup will automatically request and renew certificates for the specified domains.

## Manual Certificate Configuration

If you prefer to use your own certificates, you can configure them manually. First, define a TLS store in your static configuration:

```yaml
tls:
  stores:
    default:
      defaultCertificate:
        certFile: /path/to/cert.pem
        keyFile: /path/to/key.pem
```

Then, in your router configuration, enable TLS:

```yaml
http:
  routers:
    my-router:
      rule: "Host(`example.com`)"
      entryPoints:
        - websecure
      tls: {}
```

## TLS Options

Traefik allows you to customize TLS options, such as minimum TLS version, cipher suites, and curve preferences. Here's an example configuration:

```yaml
tls:
  options:
    default:
      minVersion: VersionTLS12
      cipherSuites:
        - TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256
        - TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384
      curvePreferences:
        - CurveP521
        - CurveP384
```

You can then reference these options in your router configuration:

```yaml
http:
  routers:
    my-router:
      rule: "Host(`example.com`)"
      entryPoints:
        - websecure
      tls:
        options: default
```

## Common Scenarios

### Redirecting HTTP to HTTPS

To automatically redirect HTTP traffic to HTTPS, you can use a middleware:

```yaml
http:
  middlewares:
    https-redirect:
      redirectScheme:
        scheme: https
        permanent: true

  routers:
    http-catchall:
      rule: "HostRegexp(`{host:.+}`)"
      entryPoints:
        - web
      middlewares:
        - https-redirect
      service: noop@internal

    secure:
      rule: "Host(`example.com`)"
      entryPoints:
        - websecure
      tls: {}
      service: my-service
```

### Using Different Certificates for Different Domains

If you need to use different certificates for different domains, you can configure multiple TLS stores:

```yaml
tls:
  stores:
    store1:
      defaultCertificate:
        certFile: /path/to/cert1.pem
        keyFile: /path/to/key1.pem
    store2:
      defaultCertificate:
        certFile: /path/to/cert2.pem
        keyFile: /path/to/key2.pem

http:
  routers:
    router1:
      rule: "Host(`domain1.com`)"
      entryPoints:
        - websecure
      tls:
        store: store1
    router2:
      rule: "Host(`domain2.com`)"
      entryPoints:
        - websecure
      tls:
        store: store2
```

This configuration allows you to use different certificates for `domain1.com` and `domain2.com`.

By following these guidelines and examples, you can effectively set up HTTPS and manage TLS certificates in Traefik for various scenarios.