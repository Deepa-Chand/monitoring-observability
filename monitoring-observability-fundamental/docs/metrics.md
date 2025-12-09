## **Metrics**
**What Are Metrics?**
Metrics are numerical measurements collected over time to help us understand the health and performance of a system or application.
Think of them like car dashboard gauges — they quickly tell you important numbers, such as speed, fuel level, or engine temperature.

Examples of metrics:

- CPU usage (%)
- Memory usage (MB)
- Request count (req/sec)
- Error rate (%)
- API latency (ms)

👉 Metrics tell you **WHAT** is happening, not necessarily **WHY**.

### **Why Metrics Are Important**

Metrics help you:

- Detect outages early
- Understand performance trends
- Trigger alerts automatically
- Plan capacity
- Measure reliability (SLO / SLA)

Without metrics:

    “Users say app is slow, but we don’t know where.”

With metrics:

    “Checkout latency increased after DB CPU crossed 85%.”

### 📦 Core Components of Metrics Monitoring
**1️⃣ Metric Source**

Where metrics come from:

- OS (node exporter)

- Application (custom metrics)

- Database (MySQL exporter, Redis exporter)

- Cloud services

**2️⃣ Metrics Collection**

A monitoring system collects metrics periodically.

Two models:

- Pull model → Prometheus (recommended)
- Push model → CloudWatch, App pushes metrics

**3️⃣ Metrics Storage**

Metrics are stored as time-series data:

    (metric_name, labels, timestamp, value)


**Example:**

    http_requests_total{method="GET",status="200"} 15423

**4️⃣ Visualization**

Dashboards help humans understand metrics.

- Grafana
- CloudWatch dashboards

**5️⃣ Alerting**

Rules that trigger notifications when thresholds are crossed.

- Email
- Slack
- PagerDuty

<img width="1280" height="720" alt="image" src="https://github.com/user-attachments/assets/5cdf8bf1-98e7-4bd0-90da-b2af763cdbc6" />


### **🧱 Types of Metrics (VERY IMPORTANT)**
**1️⃣ Infrastructure Metrics**: Track server health.

| Metric  | Example           |
| ------- | ----------------- |
| CPU     | cpu_usage_percent |
| Memory  | memory_free       |
| Disk    | disk_usage        |
| Network | network_in_bytes  |

Use case:

    “Is my server overloaded?”

**2️⃣ Application Metrics**: Track application behavior.

| Metric     | Example             |
| ---------- | ------------------- |
| Throughput | requests_per_second |
| Errors     | error_rate          |
| Latency    | response_time_ms    |

Use case:

    “Is my app slow or failing?”

**3️⃣ Business Metrics**: Track user impact.

| Metric       | Example              |
| ------------ | -------------------- |
| Orders       | orders_created       |
| Payments     | payment_success_rate |
| Active users | daily_active_users   |

Use case:

    “Is business affected?”

**⭐ Golden Signals (Must-Know)**

From Google SRE — minimum metrics you SHOULD monitor.

| Signal     | Meaning                |
| ---------- | ---------------------- |
| Latency    | How slow               |
| Traffic    | How much               |
| Errors     | How many failures      |
| Saturation | How full resources are |

Even with hundreds of metrics → always track these 4.

.

## **🆚 Metrics vs Logs vs Traces (Quick Context)**

| Type    | Answers                |
| ------- | ---------------------- |
| Metrics | *Is something wrong?*  |
| Logs    | *What happened?*       |
| Traces  | *Where did it happen?* |

Metrics are almost always the first signal.


## **⚙️ Metrics Monitoring with Prometheus (Simple Flow)**

		App exposes /metrics
		↓
		Prometheus scrapes every 15s
		↓
		Stores time-series data
		↓
		Grafana shows dashboards
		↓
		Alertmanager sends alerts
