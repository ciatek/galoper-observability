# galoper-observability

Observability for the **Galoper** product ecosystem (ERP, POS, and related services): a **central hub** fed by **lightweight agents on each monitored machine**. Telemetry flows in over the network; operators use Grafana to explore logs, metrics, traces, and profiles instead of logging into hosts ad hoc.

The **hub** (`server/`) **receives, stores, and serves** data: **Loki** for logs, **Grafana Mimir** for Prometheus-style metrics, **Grafana Tempo** for traces, **Pyroscope** for profiling, **Grafana** for UI and correlation, **Alertmanager** (and optionally **Grafana OnCall**) for alerting. **Nginx** (or similar) fronts ingress to UIs and receivers. **Pushgateway** and **nginx-prometheus-exporter** stay where they make sense (often on the hub). An optional **central Alloy** can aggregate or receive the same kinds of streams as the edge agents.

Each **client** host runs **Grafana Alloy** as the main shipper—logs (files, HTTP ingest, optional syslog), OTLP traces, optional app receiver—and **host monitors**: **node_exporter** (machine metrics), **cAdvisor** (container metrics). Syslog may go through **Alloy** or a dedicated **syslog-ng** stack; use one clear path per environment so streams are not duplicated.

## Architecture (high level)

```text
┌─────────────────────────────────────────────────────────────────┐
│                        SERVER (hub)                              │
│  Grafana · Loki · Mimir · Tempo · Pyroscope · Alertmanager       │
│  · optional central Alloy · Pushgateway · Nginx · OnCall         │
└────────────────────────────▲────────────────────────────────────┘
                             │ remote write / push / OTLP / HTTP
                             │
        ┌────────────────────┼────────────────────┐
        │                    │                    │
   ┌────┴────┐          ┌────┴────┐          ┌────┴────┐
   │ CLIENT  │          │ CLIENT  │          │ CLIENT  │
   │ Alloy   │          │ Alloy   │          │ Alloy   │
   │ +host   │          │ +host   │          │ +host   │
   │ monitors│          │ monitors│          │ monitors│
   └─────────┘          └─────────┘          └─────────┘
```

| Plane    | Hub                         | Edge (each client) |
|----------|-----------------------------|---------------------|
| Logs     | Loki ingest + query         | Alloy → Loki; files / HTTP / syslog as configured |
| Metrics  | Mimir (+ Grafana)           | node_exporter, cAdvisor; scrape or remote_write toward hub |
| Traces   | Tempo (e.g. OTLP HTTP)      | Alloy OTLP → Tempo |
| Profiles | Pyroscope                   | Alloy or language agents as configured |

Metrics from clients and scrapers should land in **Mimir** via its Prometheus-compatible remote write (or scrape federation), unless we explicitly introduce a separate Prometheus for a defined reason.

## Hub components

| Area            | Services |
|-----------------|----------|
| Visualization | Grafana |
| Metrics         | Mimir, Pushgateway, nginx-prometheus-exporter (hub Nginx) |
| Logging         | Loki |
| Traces          | Tempo |
| Profiling       | Pyroscope |
| Alerting / IRM  | Alertmanager, Grafana OnCall (+ Redis, workers) |
| Ingress         | Nginx |
| Aggregation     | Optional central Alloy (syslog / OTLP / app / Loki HTTP patterns) |

## Client stack (per machine)

| Role              | Service        | Purpose |
|-------------------|----------------|---------|
| Telemetry agent   | Grafana Alloy  | Ship logs, traces, optional app / HTTP / syslog |
| Host metrics      | node_exporter  | CPU, memory, disk, network, filesystem |
| Container metrics | cAdvisor       | Per-container resource usage |

## Repository layout

| Path      | Role |
|-----------|------|
| `client/` | Edge compose and Alloy config |
| `server/` | Hub compose and backend configs |

## Engineering preferences

- Use explicit URLs and env vars for remote endpoints (e.g. Loki push URL).
- Avoid high-cardinality values as Loki stream labels; put detail in the log line or JSON.
- When the hub layout changes, keep Grafana datasources and dashboards in sync with the new endpoints.

---

`server.backup/` is an **archived snapshot** of an older full deployment (compose, configs, provisioning). Use it only when you need historical detail; it is **not** the specification for how new hub or client layouts should behave.
