> 📖 **Original article:** [OpenTelemetry Collector: Unified Observability Without Vendor Lock-in](https://www.valtersit.com/guides/monitoring/opentelemetry-collector-unified-observability-without-vendor-lock-in/)
> *Mirror of the full guide published on [valtersit.com](https://www.valtersit.com)*

---

If your observability stack is held together by vendor-specific agents and proprietary protocols, you're not building a monitoring system—you're building a migration headache.

Every vendor ships their own agent. Datadog has its Agent. New Relic has its infrastructure agent. Splunk has its universal forwarder. Each one comes with its own configuration format, resource footprint, and data protocol. The result is agent sprawl: a single host running multiple agents, each consuming 50-200MB of RAM, conflicting on port bindings, and fighting over log rotation. I've seen a production box with seven different agents competing for resources while the actual application starved.

The OpenTelemetry Collector is the only sane abstraction layer for metrics, logs, and traces—if you do it right. This guide covers production-hardened config patterns, deployment architectures that don't fall over, and the hard-earned lessons from migrating 500+ hosts off proprietary agents. By the end, you'll have a concrete migration plan and a config that won't OOM-kill your gateway at 3 AM.

:::note[TL;DR]
- One collector replaces 3-7 proprietary agents per host, reducing memory footprint by 60-80%
- Always configure `memory_limiter` — Go's GC won't save you from traffic spikes
- Use a three-layer config structure with environment variables, never hardcode secrets
- Dual-run for 1-2 weeks before cutting over, and always have a rollback plan
- Gateways need horizontal scaling and persistent buffering for edge deployments
:::

## Prerequisites

- Basic understanding of OpenTelemetry concepts (traces, metrics, logs)
- A running OpenTelemetry Collector (v0.90.0+ recommended)
- YAML familiarity
- A healthy skepticism of "zero-config" solutions

## Why You Need a Collector—Not Just an Agent

### The Agent Sprawl Problem

I once audited a host running seven different agents: Datadog, New Relic, Splunk, Prometheus node_exporter, Fluentd, Filebeat, and a custom shell script that scraped /proc. Each agent had its own memory overhead, update cycle, and failure mode. The Datadog Agent alone was using 180MB RSS. New Relic's infrastructure agent added another 120MB. The total observability tax on that single host was over 700MB of RAM and 2.3 CPU cores.

Run this on your own infrastructure and weep:

```bash
ps aux | grep -E 'agent|collector|exporter|fluentd|filebeat|telegraf' | awk '{print $1, $6/1024, $11}' | sort -k2 -rn | head -20
```

If your observability strategy involves installing one agent per tool, you're not a DevOps engineer—you're a software hoarder.

### The Vendor Lock-in Trap

Proprietary protocols like Datadog's DogStatsD, New Relic's NRQL, and Splunk's HEC make migration a full rewrite. Your configs are locked into vendor-specific DSLs: TOML for Datadog, YAML for New Relic, HCL for some Prometheus setups. Each vendor reinvents the wheel, and you're the one pushing it.

Here's what you're trading:

| Aspect | Proprietary Agents | OpenTelemetry Collector |
|--------|-------------------|------------------------|
| Config format | Vendor-specific (TOML, custom DSL) | Standard YAML |
| Data format | Proprietary binary/protobuf | OTLP (open standard) |
| Resource usage | 50-200MB per agent × N agents | 50-150MB total |
| Upgrade path | Vendor-dependent, often breaking | Backward-compatible OTLP |
| Multi-vendor support | Impossible without multiple agents | Single pipeline, multiple exporters |
| Migration cost | Complete rewrite per vendor | Config change only |

---

> **⚠️ TRUNCATED** — This is a shortened mirror.
> Full guide (with all configs, diagrams and examples): **[https://www.valtersit.com/guides/monitoring/opentelemetry-collector-unified-observability-without-vendor-lock-in/](https://www.valtersit.com/guides/monitoring/opentelemetry-collector-unified-observability-without-vendor-lock-in/)**
