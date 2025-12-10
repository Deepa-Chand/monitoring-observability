## What Prometheus is and why it matters

**What Prometheus is**
- Prometheus is an open-source monitoring and alerting toolkit designed for collecting numeric time-series data (metrics). It focuses on reliability, a powerful query language (PromQL), and a modular ecosystem of exporters, alerting, and visualization tools.

**Key strengths (why teams choose Prometheus)**
- Multi-dimensional data model: every metric can carry labels (key/value pairs) so you can slice and dice metrics by service, instance, region, etc.
- PromQL: a concise, expressive query language for aggregations, rates, percentiles and alert expressions.
- Pull-based scrapes: Prometheus scrapes metrics endpoints from targets at regular intervals (no agent push required), simplifying service discovery and retries.
- Simplicity & portability: single binary for the server, local on-disk time series DB, easy to run standalone.
- Rich ecosystem: many ready-made exporters (nodeexporter, mysqldexporter, blackbox_exporter), client libraries for apps, and integrations (Alertmanager, Grafana).

**Core components and roles**
- Prometheus server: scrapes configured targets, stores time-series on local disk, evaluates rules, exposes HTTP API and web UI.
- Exporters: lightweight processes that expose metrics for systems that don’t natively speak Prometheus (node_exporter for host metrics, cadvisor for containers, DB exporters, etc.).
- Alertmanager: receives alerts from Prometheus, deduplicates and routes them to receivers (email, Slack, PagerDuty), supports silences and templating.
- Pushgateway: accepts pushed metrics for short-lived batch jobs that cannot be scraped.
- Grafana (or other UIs): visualizes Prometheus metrics using dashboards and panels.

**When pull vs push**
- Pull (scraping) is the default: easier target discovery, natural retries, and simpler security model (Prometheus initiates connections).
- Push is only for short-lived jobs that can’t be scraped — use Pushgateway sparingly (it’s not a general-purpose metrics store).

**Common use-cases**
- Infrastructure monitoring: CPU, memory, disk, network across hosts.
- Application metrics: request rates, error rates, latencies, custom business metrics.
- SLO/SLA/SLO monitoring: compute SLIs with PromQL and alert when SLOs degrade.
- Alerting and incident detection: automated alerts sent to teams via Alertmanager.

Quick conceptual demo (no install required yet)
- Prometheus periodically scrapes targets (e.g., http://my-service:8080/metrics).
- Each scrape returns metrics like:
  - myapprequests_total{method="GET",handler="/login",instance="10.0.0.5:8080"} 1245
- To get request rate over the last minute you use PromQL: rate(myapprequests_total[1m])
- To alert on high error rate: create a rule that evaluates a PromQL expression and forwards a firing alert to Alertmanager.

---

## Prometheus data model, metric types, and labeling best practices

Brief overview
- Prometheus stores time series as metric name + set of labels → sequence of (timestamp, value). Understanding metric types and labels is essential to correct queries, reliable alerting, and efficient storage.

**1) Metric types**
<img width="1024" height="444" alt="image" src="https://github.com/user-attachments/assets/8127707c-2971-4c04-b921-0e6a7415ff22" />

- **Counter**
  - Definition: a cumulative metric that only increases (or resets to 0 on restart). Use for counts/events.
  - Naming convention: suffix with total or count (e.g., httprequeststotal).
  - Common use: request counts, error counts.
  - Query patterns: use rate() or increase() on counters.
    - Example: rate(httprequeststotal[5m]) returns per-second request rate.

- **Gauge**
  - Definition: a metric that can go up and down arbitrarily. Represents instantaneous values.
  - Naming: descriptive, e.g., memoryusagebytes, temperature_celsius.
  - Common use: current memory usage, queue length, number of active sessions.
  - Query patterns: use directly or compute changes with delta().

- **Histogram**
  - Definition: records observations into configurable buckets and exposes:
    - myhistogrambucket{le="..."} (cumulative counts per bucket)
    - myhistogramsum (sum of observed values)
    - myhistogramcount (total count)
  - Use: latency distributions, sizes. Enables quantile estimates via histogram_quantile().
  - Important: buckets are cumulative and defined at instrumentation time.
  - Example usage: histogramquantile(0.95, sum(rate(httprequestdurationseconds_bucket[5m])) by (le, job))

- **Summary**
  - Definition: client-side summary that provides quantiles, plus sum and count.
  - Difference to histogram: summaries calculate quantiles per-client (not combinable across dimensions reliably). Use when you need local quantiles and don't need global aggregation.
  - Recommendation: prefer histograms when you need to aggregate across instances (e.g., cluster-wide p95).

**2) Labels and multi-dimensional model**
- Label structure: metric_name{label1="value1", label2="value2", ...}
- Purpose: slice metrics by dimensions — service, instance, region, method, etc.
- Examples:
  - httprequeststotal{job="api", handler="/login", method="POST", instance="10.0.0.5:8080"}
- Best practices:
  - Use labels for low-cardinality dimensions (job, region, environment).
  - Avoid high-cardinality labels (unique IDs, userid, requestid); they explode series and storage.
  - Prefer fixed sets or enumerations as label values (e.g., status="200" or env="prod|staging").
  - Use label names consistently across metrics (e.g., job, instance, service, region).
  - If you must tag high-cardinality data, consider pushing to a logging/trace system instead (Elasticsearch, Jaeger).

**3) Naming conventions**
- Metric names:
  - Use lowercase and underscores.
  - Start with the component or service prefix when appropriate (e.g., nodecpuseconds_total).
  - Include unit in name when helpful (bytes, seconds, seconds_total).
- Suffixes:
  - _total for counters
  - seconds or duration_seconds for durations
  - _bytes for memory/size
  - bucket/sum/_count for histograms (exporters and client libs do this automatically)

**4) Timestamps and scrape model**
- Prometheus assigns the scrape time as the metric timestamp by default (exporters can include timestamps but discouraged).
- Scrape interval (global or per-job) controls data resolution; shorter intervals → more data points and higher cardinality.
- Consider retention and scrape frequency trade-offs for storage.

---

## Exercise

## ✅ Option 1: Install Prometheus on a Linux Server (VM / EC2 / Bare Metal)

### 🔹 Prerequisites

* Linux (Ubuntu 20.04+ recommended)
* sudo access
* Open port **9090** in firewall / security group

---

### 🔹 Step 1: Create Prometheus User

```bash
sudo useradd --no-create-home --shell /bin/false prometheus
```

---

### 🔹 Step 2: Create Directories

```bash
sudo mkdir /etc/prometheus
sudo mkdir /var/lib/prometheus

sudo chown prometheus:prometheus /etc/prometheus
sudo chown prometheus:prometheus /var/lib/prometheus
```

---

### 🔹 Step 3: Download Prometheus

```bash
cd /tmp
wget https://github.com/prometheus/prometheus/releases/download/v2.52.0/prometheus-2.52.0.linux-amd64.tar.gz
tar xvf prometheus-2.52.0.linux-amd64.tar.gz
```

---

### 🔹 Step 4: Move Files

```bash
sudo cp prometheus-2.52.0.linux-amd64/prometheus /usr/local/bin/
sudo cp prometheus-2.52.0.linux-amd64/promtool /usr/local/bin/

sudo cp -r prometheus-2.52.0.linux-amd64/consoles /etc/prometheus
sudo cp -r prometheus-2.52.0.linux-amd64/console_libraries /etc/prometheus

sudo chown -R prometheus:prometheus /etc/prometheus
```

---

### 🔹 Step 5: Create Config File 
Link: https://prometheus.io/docs/introduction/first_steps/#configuring-prometheus

```bash
sudo nano /etc/prometheus/prometheus.yml
```

```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: "prometheus"
    static_configs:
      - targets: ["localhost:9090"]
```

```bash
sudo chown prometheus:prometheus /etc/prometheus/prometheus.yml
```

---

### 🔹 Step 6: Create Systemd Service

```bash
sudo nano /etc/systemd/system/prometheus.service
```

```ini
[Unit]
Description=Prometheus Monitoring
Wants=network-online.target
After=network-online.target

[Service]
User=prometheus
Group=prometheus
Type=simple
ExecStart=/usr/local/bin/prometheus \
  --config.file=/etc/prometheus/prometheus.yml \
  --storage.tsdb.path=/var/lib/prometheus/

[Install]
WantedBy=multi-user.target
```

---

### 🔹 Step 7: Start Prometheus

```bash
sudo systemctl daemon-reload
sudo systemctl start prometheus
sudo systemctl enable prometheus
sudo systemctl status prometheus
```

---

### 🔹 Step 8: Access UI

```
http://<server-ip>:9090
```

![Image](https://user-images.githubusercontent.com/21349004/136120997-36c58a17-523a-48ea-b73b-73d73aa5ca7a.png?utm_source=chatgpt.com)


---

## ✅ Option 2: Install Prometheus on Kubernetes (Recommended)

### 🔹 Prerequisites

* Kubernetes cluster
* kubectl configured
* Helm installed

---

### 🔹 Step 1: Add Helm Repo

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
```

---

### 🔹 Step 2: Install Prometheus Stack

```bash
helm install monitoring prometheus-community/kube-prometheus-stack
```

This installs:

* Prometheus
* Alertmanager
* Grafana
* Node Exporter
* kube-state-metrics

---

### 🔹 Step 3: Check Pods

```bash
kubectl get pods
```

---

### 🔹 Step 4: Access Prometheus

```bash
kubectl port-forward svc/monitoring-kube-prometheus-prometheus 9090
```

Then open:

```
http://localhost:9090
```

![Image](https://www.apptio.com/wp-content/uploads/kube-prometheus-1.png?utm_source=chatgpt.com)

![Image](https://raw.githubusercontent.com/instrumentisto/grafana-dashboard-kubernetes-prometheus/master/screens/total.png?utm_source=chatgpt.com)


---

