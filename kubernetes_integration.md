# Kubernetes Integration

This guide provides detailed instructions on using Traefik with Kubernetes, including how to deploy Traefik as an Ingress controller, configure IngressRoute CRDs, and set up TLS.

## Table of Contents

1. [Introduction](#introduction)
2. [Deploying Traefik as an Ingress Controller](#deploying-traefik-as-an-ingress-controller)
3. [Configuring IngressRoute CRDs](#configuring-ingressroute-crds)
4. [Setting up TLS](#setting-up-tls)
5. [Advanced Configuration](#advanced-configuration)

## Introduction

Traefik is a modern HTTP reverse proxy and load balancer that integrates seamlessly with Kubernetes. It can be used as an Ingress controller to manage external access to services within a Kubernetes cluster.

## Deploying Traefik as an Ingress Controller

To deploy Traefik as an Ingress controller in your Kubernetes cluster, follow these steps:

1. Create a `ClusterRole` and `ClusterRoleBinding` for Traefik:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: traefik-ingress-controller
rules:
  - apiGroups:
      - ""
    resources:
      - services
      - endpoints
      - secrets
    verbs:
      - get
      - list
      - watch
  - apiGroups:
      - extensions
      - networking.k8s.io
    resources:
      - ingresses
      - ingressclasses
    verbs:
      - get
      - list
      - watch
  - apiGroups:
      - extensions
      - networking.k8s.io
    resources:
      - ingresses/status
    verbs:
      - update
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: traefik-ingress-controller
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: traefik-ingress-controller
subjects:
  - kind: ServiceAccount
    name: traefik-ingress-controller
    namespace: kube-system
```

2. Create a `Deployment` for Traefik:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: traefik
  namespace: kube-system
  labels:
    app: traefik
spec:
  replicas: 1
  selector:
    matchLabels:
      app: traefik
  template:
    metadata:
      labels:
        app: traefik
    spec:
      serviceAccountName: traefik-ingress-controller
      containers:
        - name: traefik
          image: traefik:v2.5
          args:
            - --api.insecure
            - --providers.kubernetesingress
          ports:
            - name: web
              containerPort: 80
            - name: dashboard
              containerPort: 8080
```

3. Create a `Service` to expose Traefik:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: traefik
  namespace: kube-system
spec:
  type: LoadBalancer
  ports:
    - port: 80
      targetPort: web
  selector:
    app: traefik
```

Apply these YAML configurations to your cluster using `kubectl apply -f <filename>.yaml`.

## Configuring IngressRoute CRDs

Traefik introduces Custom Resource Definitions (CRDs) to enhance routing capabilities. The `IngressRoute` CRD provides a more flexible way to define routing rules compared to standard Kubernetes Ingress resources.

To use IngressRoute CRDs:

1. Apply the CRD definition to your cluster:

```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: ingressroutes.traefik.containo.us
spec:
  group: traefik.containo.us
  versions:
    - name: v1alpha1
      served: true
      storage: true
      schema:
        openAPIV3Schema:
          type: object
          properties:
            spec:
              type: object
              properties:
                routes:
                  type: array
                  items:
                    type: object
                    properties:
                      match:
                        type: string
                      kind:
                        type: string
                      services:
                        type: array
                        items:
                          type: object
                          properties:
                            name:
                              type: string
                            port:
                              type: integer
  scope: Namespaced
  names:
    plural: ingressroutes
    singular: ingressroute
    kind: IngressRoute
    shortNames:
      - ir
```

2. Create an IngressRoute resource:

```yaml
apiVersion: traefik.containo.us/v1alpha1
kind: IngressRoute
metadata:
  name: myingressroute
  namespace: default
spec:
  entryPoints:
    - web
  routes:
    - match: Host(`example.com`) && PathPrefix(`/api`)
      kind: Rule
      services:
        - name: api-service
          port: 80
```

This example routes traffic for `example.com/api` to the `api-service` on port 80.

## Setting up TLS

To enable TLS for your Traefik Ingress:

1. Create a TLS secret:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: example-tls
  namespace: default
type: kubernetes.io/tls
data:
  tls.crt: <base64-encoded-cert>
  tls.key: <base64-encoded-key>
```

2. Update your IngressRoute to use TLS:

```yaml
apiVersion: traefik.containo.us/v1alpha1
kind: IngressRoute
metadata:
  name: myingressroute
  namespace: default
spec:
  entryPoints:
    - websecure
  routes:
    - match: Host(`example.com`) && PathPrefix(`/api`)
      kind: Rule
      services:
        - name: api-service
          port: 80
  tls:
    secretName: example-tls
```

## Advanced Configuration

For advanced Traefik configuration with Kubernetes, consider exploring:

- Middleware: Use Traefik middleware to modify requests or responses.
- Traffic splitting: Implement canary releases or A/B testing.
- TCP routing: Configure TCP services using Traefik.

Refer to the official Traefik documentation for more detailed information on these advanced topics.