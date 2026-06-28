> 📖 **Original article:** [Log Aggregation with Loki: The Prometheus Approach to Logs](https://www.valtersit.com/guides/monitoring/log-aggregation-with-loki-the-prometheus-approach-to-logs/)
> *Mirror of the full guide published on [valtersit.com](https://www.valtersit.com)*

---

# Log Aggregation with Loki: The Prometheus Approach to Logs
## A Senior DevSecOps Engineer's Guide to Actually Useful Logging

I've seen it too many times: a 20-node Elasticsearch cluster that costs $40,000/month in AWS instances alone, requiring a full-time engineer to tune shards and rebalance indices. The breaking point was when someone forgot to set `index.number_of_shards` on a new index, and the cluster ate 12TB of disk in four hours during a production incident. No alert. No warning. Just a silent OOM cascade.

This guide is for engineers who've been burned by Elasticsearch pricing, tired of paying S3-equivalent storage costs for log data, and want Prometheus-style querying for logs without the operational cancer. After reading, you'll know exactly when Loki is the right tool, how to deploy it without shooting yourself in the foot, and—more importantly—when to walk away and use something else.

:::note[TL;DR]
- Loki doesn't index log content—it indexes labels. This is the single most important architectural decision that drives cost, performance, and query behavior.
- Label cardinality is your #1 enemy. A single high-cardinality label (like `request_id` or `user_ip`) will kill your cluster faster than any hardware failure.
- Start with a single binary deployment. You don't need 12 microservices for 100GB/day. Scale only when metrics tell you to.
- Loki is not Elasticsearch. If you need full-text search across billions of logs, use a dedicated search engine. Loki is for operational log aggregation with metrics-style querying.
:::

## Prerequisites

Before you start, you'll need:
- A Linux server (Ubuntu 22.04+ or RHEL 9+) with Docker installed (or a Kubernetes cluster if you're going that route)
- An S3-compatible object storage bucket (AWS S3, MinIO, or GCS)
- Grafana 9.x+ installed and accessible
- Basic familiarity with Prometheus-style label-based querying
- At least 4GB RAM and 2 CPU cores for the Loki instance (more for production workloads)

## Why Loki Exists — The Elasticsearch Wake-Up Call

Elasticsearch solves a fundamentally different problem than Loki. ES builds an inverted index on every field you send it—which gives you blazing-fast full-text search but also means you're paying for CPU, memory, and disk I/O proportional to your log volume. At scale, that cost curve is exponential.

Loki takes the opposite approach: it indexes nothing about the log content itself. Instead, it indexes metadata (labels) and stores the raw log lines as compressed chunks in object storage. This is the same philosophy Prometheus uses for metrics—and it works because 90% of your log queries are "show me all logs for service X in environment Y with status code 5xx."

The trade-off is explicit: you can't do `LIKE '%something%'` across all logs without a line filter that scans chunks. That's by design. If you need full-text search, you need a different tool.

```yaml
# docker-compose.yml — Minimum Viable Loki Stack
version: '3.8'

services:
  loki:
    image: grafana/loki:2.9.4
    ports:
      - "3100:3100"
    volumes:
      - ./loki-config.yaml:/etc/loki/local-config.yaml
      - ./data/loki:/loki
    command: -config.file=/etc/loki/local-config.yaml
    restart: unless-stopped

  promtail:
    image: grafana/promtail:2.9.4
    volumes:
      - ./promtail-config.yaml:/etc/promtail/config.yaml
      - /var/log:/var/log:ro
    command: -config.file=/etc/promtail/config.yaml
    depends_on:
      - loki
```

---

> **⚠️ TRUNCATED** — This is a shortened mirror.
> Full guide (with all configs, diagrams and examples): **[https://www.valtersit.com/guides/monitoring/log-aggregation-with-loki-the-prometheus-approach-to-logs/](https://www.valtersit.com/guides/monitoring/log-aggregation-with-loki-the-prometheus-approach-to-logs/)**
