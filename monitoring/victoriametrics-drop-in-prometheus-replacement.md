> 📖 **Original article:** [VictoriaMetrics: Drop-In Prometheus Replacement?](https://www.valtersit.com/guides/monitoring/victoriametrics-drop-in-prometheus-replacement/)
> *Mirror of the full guide published on [valtersit.com](https://www.valtersit.com)*

---

import \{ Callout, Aside, Tabs, Table \} from './components'

At 02:14 the pager fired. A 64 GB Prometheus box had been happily scraping 40 jobs for eight months, and then somebody shipped a dashboard. A single `histogram_quantile` over a 90-day range hit a head block holding 4.2 million active series, the query allocator ballooned, and the kernel OOM-killer picked a fight it won. By morning three teams had "lost" their SLO dashboards and I had an impromptu architecture review with my name on the invite.

This guide is for SREs, platform engineers, and DevSecOps folks who run Prometheus at a scale where the RAM bill has stopped being funny — and who are tired of vendor slide decks that call everything "drop-in." By the end you'll know exactly what migrates cleanly to VictoriaMetrics, what quietly changes behavior, and which flags keep you out of OOM jail.

:::note[TL;DR]
- VictoriaMetrics is **API-compatible** with Prometheus, but MetricsQL is a dialect — not a clone. Grep your recording rules before you trust them.
- The RAM savings are real because the storage engine is different (merge-tree parts + inverted index), not because of "optimization."
- `remote_write` from existing Prometheus solves nothing for RAM — `vmagent` is the actual migration.
- Set `-search.maxQueryDuration`, `-memory.allowedPercent`, and `-maxLabelsPerTimeseries` on day one, not after the first incident.
- VM ships with **no authentication**. If you expose `:8428` to a network you don't own, you have a data exfiltration problem, not an observability problem.
:::

:::caution[Disclaimer]
Every RAM number in this post comes from *my* workloads. Your cardinality is not my cardinality, your payload sizes are not mine, and your query mix is almost certainly worse than you think. Treat the ratios as directional, not as SLAs.
:::

## Prerequisites

- A working Prometheus (2.x) with at least a `prometheus.yml` you understand line by line
- A host with a dedicated data volume — XFS or ext4, never NFS
- Grafana pointed at Prometheus today, so you can flip datasources later
- `vmagent`, `vmalert`, and `vmctl` binaries (or the Docker images) on hand
- Roughly 2x your current retention disk free for a dual-run window

## The Problem: Why Your Prometheus Box Eats 64 GB and Still Dies

This section matters because if you don't understand *why* Prometheus uses the RAM it uses, you'll migrate for the wrong reasons and tune the wrong knobs.

### How Prometheus Actually Spends RAM

Prometheus keeps every active series in memory as a `memSeries` struct — typically 1–3 KB per series before you count the label strings themselves. Add the inverted index (mapped, but touched eagerly), the current chunk encodings, and the WAL. On restart, that WAL replays *into the head block*, re-materializing series before the process will serve a single query. I once watched a restart take nine minutes because WAL replay had to rebuild roughly 3 million series — during which, obviously, alerting was blind.

### The Cardinality Bomb Nobody Budgets For

Blame assignment: it is almost never Prometheus. It is an exporter that decided a `path` label was a good idea on a URL-routed API. One HTTP exporter with a `path` label and no relabel noise can add 800k series in an afternoon. The scrape config that pulled it in hasn't changed in a year; the payload just grew a shape nobody modeled.

Before you migrate anything, find your offenders. Run these against the Prometheus you have today:

---

> **⚠️ TRUNCATED** — This is a shortened mirror.
> Full guide (with all configs, diagrams and examples): **[https://www.valtersit.com/guides/monitoring/victoriametrics-drop-in-prometheus-replacement/](https://www.valtersit.com/guides/monitoring/victoriametrics-drop-in-prometheus-replacement/)**
