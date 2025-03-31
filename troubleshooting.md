# Troubleshooting Traefik

This guide provides solutions for common issues you may encounter when using Traefik, including configuration problems, TLS certificate issues, and routing errors. It also offers step-by-step debugging processes and guidance on how to interpret Traefik logs.

## Table of Contents

1. [Common Configuration Issues](#common-configuration-issues)
2. [TLS Certificate Problems](#tls-certificate-problems)
3. [Routing Errors](#routing-errors)
4. [Interpreting Traefik Logs](#interpreting-traefik-logs)
5. [Debugging Steps](#debugging-steps)

## Common Configuration Issues

### 1. Incorrect YAML Syntax

One of the most common issues is incorrect YAML syntax in the Traefik configuration file.

**Problem**: Traefik fails to start or apply configuration changes.

**Solution**: 
- Use a YAML validator to check your configuration file.
- Pay attention to indentation and use spaces instead of tabs.
- Ensure all key-value pairs are properly formatted.

Example of correct YAML syntax:

```yaml
http:
  routers:
    my-router:
      rule: "Host(`example.com`)"
      service: my-service
  services:
    my-service:
      loadBalancer:
        servers:
          - url: "http://localhost:8080"
```

### 2. Mismatched Port Configuration

**Problem**: Traefik is not listening on the expected ports.

**Solution**:
- Check the `entryPoints` configuration in your Traefik static configuration.
- Ensure that the ports specified in your dynamic configuration match the entryPoints.

Example:

```yaml
entryPoints:
  web:
    address: ":80"
  websecure:
    address: ":443"
```

## TLS Certificate Problems

### 1. Certificate Not Found

**Problem**: Traefik cannot find the specified TLS certificate.

**Solution**:
- Verify the path to your certificate files.
- Ensure Traefik has read permissions for the certificate files.
- Check if the certificate names in your configuration match the actual filenames.

Example configuration:

```yaml
tls:
  certificates:
    - certFile: "/path/to/your/certificate.crt"
      keyFile: "/path/to/your/private.key"
```

### 2. Let's Encrypt Certificate Generation Failure

**Problem**: Automatic certificate generation with Let's Encrypt fails.

**Solution**:
- Ensure your domain's DNS is properly configured and pointing to your server.
- Check that ports 80 and 443 are open and accessible from the internet.
- Verify that the ACME challenge can be completed (HTTP-01 or DNS-01).

Example Let's Encrypt configuration:

```yaml
certificatesResolvers:
  myresolver:
    acme:
      email: your-email@example.com
      storage: acme.json
      httpChallenge:
        entryPoint: web
```

## Routing Errors

### 1. Rules Not Matching

**Problem**: Requests are not being routed to the expected service.

**Solution**:
- Double-check your routing rules for typos or logical errors.
- Ensure that the Host rule matches your domain exactly.
- Use Traefik's debug mode to see which rules are being evaluated.

Example of a correct routing rule:

```yaml
http:
  routers:
    my-router:
      rule: "Host(`example.com`) && PathPrefix(`/api`)"
      service: my-api-service
```

### 2. Service Not Found

**Problem**: Traefik reports that it cannot find the specified service.

**Solution**:
- Verify that the service name in your router configuration matches the service definition.
- Check if the service is properly defined in your dynamic configuration.
- Ensure that the backend servers are running and accessible to Traefik.

## Interpreting Traefik Logs

Traefik logs are crucial for understanding what's happening in your reverse proxy. Here's how to interpret common log entries:

1. **Access Logs**: Show incoming requests and their outcomes.
   Example: `172.17.0.1 - - [25/May/2023:10:12:34 +0000] "GET /api HTTP/1.1" 200 1234 "-" "Mozilla/5.0 ..."`

2. **Error Logs**: Indicate issues with routing, services, or Traefik itself.
   Example: `time="2023-05-25T10:15:00Z" level=error msg="Service not found" serviceName=my-service routerName=my-router`

3. **Debug Logs**: Provide detailed information about Traefik's operations (when debug mode is enabled).
   Example: `time="2023-05-25T10:20:00Z" level=debug msg="Handling connection" src=172.17.0.1:12345`

To enable debug logs, set the log level to "DEBUG" in your Traefik configuration:

```yaml
log:
  level: DEBUG
```

## Debugging Steps

When troubleshooting Traefik, follow these steps:

1. **Check Configuration**: Validate your Traefik configuration files for syntax errors and logical mistakes.

2. **Verify Connectivity**: Ensure Traefik can reach your backend services and that necessary ports are open.

3. **Enable Debug Logging**: Set log level to DEBUG to get more detailed information about Traefik's operations.

4. **Analyze Logs**: Look for error messages or unexpected behavior in Traefik's logs.

5. **Use Traefik Dashboard**: If enabled, the Traefik dashboard can provide valuable insights into your configuration and routing.

6. **Check Backend Services**: Verify that your backend services are running and responding correctly.

7. **Test with cURL**: Use cURL to make requests directly to Traefik and analyze the responses.

   Example:
   ```
   curl -v -H "Host: example.com" http://localhost:80
   ```

8. **Isolate the Problem**: If possible, simplify your configuration to isolate the issue to a specific component or service.

By following these troubleshooting steps and understanding common issues, you should be able to resolve most problems encountered with Traefik. If you're still facing difficulties, consider seeking help from the Traefik community forums or opening an issue on the GitHub repository.