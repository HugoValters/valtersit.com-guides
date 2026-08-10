> 📖 **Original article:** [Kubernetes Ingress Controllers: Nginx vs Traefik vs Caddy](https://www.valtersit.com/guides/kubernetes/kubernetes-ingress-controllers-nginx-vs-traefik-vs-caddy/)
> *Mirror of the full guide published on [valtersit.com](https://www.valtersit.com)*

---

# Kubernetes Ingress Controllers Compared: Nginx vs Traefik vs Caddy — A 15-Year Veteran's Honest Take

Stop treating ingress controllers like they're interchangeable. They're not. I've watched teams paint themselves into corners with the wrong choice, and it's always the same story: "we just picked the default." That default was Nginx in a managed cluster, and it worked fine until they needed WebSocket support with sticky sessions and gRPC — then the annotation soup became unmanageable. This guide is the one I wish I'd had before my third production outage caused by a misconfigured `proxy-buffer-size` that truncated API responses for two days.

There is no "best" controller. There is only the best controller for *your* traffic patterns, team skill level, and operational pain tolerance. We're going to dissect the three most popular open-source options (Nginx, Traefik, Caddy) from a production-hardened perspective. You'll learn performance benchmarks, configuration philosophies, debugging nightmares, and the hidden operational costs that don't show up on a feature comparison matrix.

You know what a Pod is. You've yaml'd a Deployment. You're here because you're tired of copy-pasting configs without understanding the consequences. By the end, you'll know exactly which controller to pick — and more importantly, how to migrate if you picked wrong.

:::note[TL;DR]
- Nginx is the boring, battle-tested default. It's powerful but requires deep tuning knowledge to avoid footguns.
- Traefik's CRD-based dynamic config is superior for complex microservices with WebSockets and gRPC.
- Caddy's automatic HTTPS eliminates cert-manager complexity but has a smaller ecosystem.
- The most expensive part of this decision is the migration cost later. Abstract your routing config from day one.
:::

## Prerequisites

- A Kubernetes cluster (v1.28+ recommended) with `kubectl` configured
- Basic understanding of `Ingress` resources and DNS resolution
- `helm` installed for controller deployment
- Access to a test environment — do not experiment on production

## The Lay of the Land: Why Ingress Controllers Exist (And Why You're Using Them Wrong)

### The Kubernetes Ingress API is a Contract, Not a Solution

The `Ingress` resource is just a spec. The controller is the implementation. Most people conflate the two, which is why they're shocked when annotations don't work across controllers. Here's the fundamental syntax difference immediately:

```yaml
# Nginx uses annotations to control behavior
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: api-gateway
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /$2
    nginx.ingress.kubernetes.io/proxy-body-size: 10m
spec:
  rules:
  - host: api.example.com
    http:
      paths:
      - path: /v1(/|$)(.*)
        pathType: Prefix
        backend:
          service:
            name: api-service
            port:
              number: 8080
```

```yaml
# Traefik uses CRDs — cleaner, self-documenting
apiVersion: traefik.io/v1alpha1
kind: IngressRoute
metadata:
  name: api-gateway
spec:
  entryPoints:
    - websecure
  routes:
  - match: Host(`api.example.com`) && PathPrefix(`/v1`)
    kind: Rule
    services:
    - name: api-service
      port: 8080
```

CRDs (Traefik/Caddy) are superior for complex routing. Annotations are a hack that turns your YAML into a swamp of magic strings. Fight me.

### The Architecture Divide: Proxy vs. Edge Router vs. Reverse Proxy

---

> **⚠️ TRUNCATED** — This is a shortened mirror.
> Full guide (with all configs, diagrams and examples): **[https://www.valtersit.com/guides/kubernetes/kubernetes-ingress-controllers-nginx-vs-traefik-vs-caddy/](https://www.valtersit.com/guides/kubernetes/kubernetes-ingress-controllers-nginx-vs-traefik-vs-caddy/)**
