## 🚀 Prometheus – Learning from Basics
Prometheus is a time-series metrics system: it scrapes numeric measurements (metrics) from targets, stores them efficiently, lets you query them with PromQL, and fires alerts via Alertmanager. Great for monitoring services, infrastructure, and Kubernetes.

### **✅ What is Prometheus?**

Prometheus is a:

- Metrics collection system
- Time-series database
- Query engine (PromQL)
- Alert source

It follows a **pull model**:

    Prometheus goes to your app and asks: “Give me your metrics”


## Core concepts
<img width="1364" height="758" alt="image" src="https://github.com/user-attachments/assets/f9c2898f-6c12-4241-8ad7-0f60285d577f" />

**Prometheus Server:**
- What it is: The main engine of the Prometheus system.
- What it does: Scrapes metrics from targets, Stores metrics as time-series data, Answers queries (PromQL), Evaluates alert rules
- Key responsibilities:
  
      Collect → Store → Query → Alert
- Example: Every 15 seconds:

      “Hey app, give me your metrics”
- Inside Prometheus Server
  | Sub-part      | Purpose               |
| ------------- | --------------------- |
| Scraper       | Fetches `/metrics`    |
| TSDB          | Time-series database  |
| PromQL engine | Executes queries      |
| Rule engine   | Evaluates alert rules |

**Scraping**
- Prometheus regularly scrapes targets (your app instances, exporters) by pulling metrics using HTTP.
- The scrape interval is configurable (default 15s).
- Targets are either fixed (static_configs) or discovered dynamically (via service discovery on Kubernetes, AWS, etc.).
- Scraping pulls metrics actively; Prometheus does not receive pushed metrics except through special tools like Pushgateway.

**Pushgateway** :
- What it is: Pushgateway is a Prometheus component that accepts pushed metrics from short-lived jobs and exposes them so Prometheus can scrape them.
- Purpose: let ephemeral or batch jobs report metrics when they can’t be continuously scraped.
- How it works: job pushes metrics (HTTP POST/PUT) to Pushgateway → Pushgateway stores them as time series with labels → Prometheus scrapes Pushgateway’s /metrics endpoint.
- Typical use case: CI jobs, cron jobs, short-lived batch tasks, containers that exit quickly.
- Not for regular apps: do not use Pushgateway as a general push-based ingestion for long-running services.
- 
**Targets**
- What it is: Any endpoint that exposes metrics.
- target=>
  
      Application (/metrics)
      Server (node_exporter)
      Database exporter
      Kubernetes pods
- Example: target = metrics endpoint URL = http://myapp:8080/metrics
  Prometheus treats this as one scrape target.

**Exporters (Metric Translators 🔌)**
- What it is: A small service that converts system/app data into Prometheus format.
- why it's needed: Most systems do not expose **/metrics** by default. Exporters solve this.
- Common Exporters:
  | Exporter          | Monitors          |
| ----------------- | ----------------- |
| node_exporter     | CPU, memory, disk |
| mysql_exporter    | MySQL DB          |
| redis_exporter    | Redis             |
| kafka_exporter    | Kafka             |
| blackbox_exporter | Endpoint health   |

- Example Flow

      Linux Server
      ↓
      node_exporter
      ↓
      Prometheus scrapes /metrics


**Metrics Endpoint (/metrics)**
- What it is: An HTTP endpoint where metrics are exposed.
- ✅ Prometheus Format  `metric_name{label1="value"} 123`
- example: `http_requests_total{method="GET",status="200"} 4567
- Prometheus understands ONLY this format.

**PromQL (Query Language 🧠)**
- What it is: Language to query, aggregate, and analyze metrics.
- Used for: Dashboards, Alerts, Troubleshooting`
- ✅ Example Queries: Requests per second:=> `rate(http_requests_total[5m])`

**Time-Series Database (TSDB 💾)**
- What it stores: Each metric is stored as: `(metric_name + labels) → time → value`
- Prometheus stores all scraped data locally in its custom time series database. Data is stored as timestamped samples of metrics with associated labels.
- Data retention is configurable (e.g., 15 days by default).

**Alerting Rules 🚨**
- Define conditions when something is wrong.
- Example:   `IF error rate > 5% for 5m → Alert`
- Prometheus evaluates alerts periodically, But does NOT send notifications directly.
- When a rule’s condition is true for a given time, Prometheus sends alerts to Alertmanager.

**Alertmanager (Notification Brain 📢)**
- Responsible for sending alerts to humans/tools.
- What it handles: Deduplication, Grouping alerts, Routing, Silencing
- Alert Destinations: Slack, Email, PagerDuty, OpsGenie, Webhooks

**Service Discovery & Relabeling**
- Prometheus supports dynamic discovery of scrape targets from cloud providers, Kubernetes, Consul, etc.
- Relabeling transforms or filters target labels before scraping or alert evaluation.
- Useful to drop unnecessary targets, modify target labels, or add metadata without changing actual services.

<img width="1280" height="720" alt="image" src="https://github.com/user-attachments/assets/a16a6691-2ce3-4d70-875d-74a150215a34" />

## Complete Architecture Flow (Simple)

    App / Exporter
       ↓ (/metrics)
    Prometheus
       ↓
    TSDB
       ↓
    Grafana (View)
       ↓
    Alert rules
       ↓
    Alertmanager
       ↓
    Slack / Email

