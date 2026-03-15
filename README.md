# k8s-Observability Helm Chart
<img width="1408" height="768" alt="Image" src="https://github.com/user-attachments/assets/283d1dd9-eeaa-41b9-99e1-1363acc8f967" />

A comprehensive, production-ready Helm chart for deploying a complete observability stack on Kubernetes. This chart bundles **metrics collection**, **distributed tracing**, **log aggregation**, and **visualization** into a single, extensible package.

> **Note:** This chart is designed to work with AWS EKS and Kubernetes clusters with AWS ALB Ingress Controller. Configuration can be adapted for other cloud providers.

## Table of Contents

- [Overview](#overview)
- [Architecture](#architecture)
- [Use Cases](#use-cases)
- [Prerequisites](#prerequisites)
- [Quick Start](#quick-start)
- [Chart Components](#chart-components)
- [Tools Usage Guide](#tools-usage-guide)
  - [Prometheus Usage](#prometheus-complete-usage-guide)
  - [Grafana Usage](#grafana-complete-usage-guide)
  - [Loki Usage](#loki-complete-usage-guide)
  - [Tempo Usage](#tempo-complete-usage-guide)
  - [Alloy Usage](#alloy-complete-usage-guide)
- [Sample Trace Generation Code](#sample-trace-generation-code)
  - [Python](#1-python---opentelemetry-trace-generation)
  - [Node.js](#2-nodejs-javascript---opentelemetry-trace-generation)
  - [Go](#3-go---opentelemetry-trace-generation)
  - [Java](#4-java---opentelemetry-trace-generation)
- [values.yaml Detailed Explanation](#valuesyaml-detailed-explanation)
- [Configuration](#configuration)
- [Accessing the Dashboard](#accessing-the-dashboard)
- [Customization](#customization)
- [Troubleshooting](#troubleshooting)
- [Contributing](#contributing)
- [License](#license)

---

## Overview

The **k8s-Observability** Helm chart provides an integrated observability solution that collects and visualizes **metrics**, **logs**, and **traces** from Kubernetes workloads. The stack follows the "Three Pillars of Observability" and provides:

- **Metrics**: System and application performance indicators (CPU, memory, requests/second, latency, etc.)
- **Logs**: Structured and unstructured log data from pods and containers
- **Traces**: Distributed request traces across microservices and application components

This chart is intentionally modular—each component can be enabled, disabled, or customized independently. It's designed as a foundation that can be extended with additional tools, dashboards, and custom configurations.

---

## Architecture

<img width="786" height="759" alt="Image" src="https://github.com/user-attachments/assets/7cbbcb21-b355-4b10-94c6-9690a6a3d19f" />

---

## Use Cases

### 1. **Microservices Monitoring and Debugging**

**Scenario:** You have a multi-tier microservices architecture with services A → B → C.

**How the observability stack helps:**

- **Traces**: Request flows through all services are captured via OpenTelemetry. When a user reports slow requests, you can:
  - View the full request path from entry point to database
  - Identify which service(s) are causing latency
  - See all errors and exceptions that occurred during the request
  - Correlate traces with metrics and logs

**Example Workflow:**

```
1. User reports: "Payment service is slow"
2. You open Grafana → Tempo trace view
3. Filter traces by service="payment-service" and duration > 5s
4. Click on a trace to see:
   - Time in payment-service: 3.5s
   - Time in inventory-service: 1.2s
   - Time in database queries: 0.8s
5. See the specific slow query and the log lines from that time period
6. Identify the root cause and deploy a fix
```

**Tools Used:** Tempo (tracing), Prometheus (latency metrics), Loki (logs)

---

### 2. **Infrastructure and Resource Utilization Analysis**

**Scenario:** Your cluster is growing and you need to understand resource consumption patterns.

**How the observability stack helps:**

- **Metrics**: Prometheus collects CPU, memory, disk, and network metrics from every node and pod
- **Dashboards**: Pre-configured Grafana dashboards visualize:
  - Node resource usage (CPU, memory, disk I/O)
  - Pod resource consumption by namespace
  - Cluster capacity planning metrics
  - Container image sizes and pull times

**Example Workflow:**

```
1. Grafana dashboard shows: "Memory usage trending upward"
2. Drill down to namespace level → see which namespace is growing
3. Drill down to pod level → identify the memory leak
4. Check logs during the time period using Loki
5. Correlate with application traces to find the code path causing the leak
```

**Tools Used:** Prometheus (metrics), Grafana (visualization), Loki (logs)

---

### 3. **Application Performance Monitoring (APM)**

**Scenario:** Your SaaS application needs to track API performance, response times, and error rates.

**How the observability stack helps:**

- **Metrics**: Application-level metrics (request count, latency, error rate) from Prometheus
- **Traces**: Detailed spans showing where time is spent in each request
- **Service Maps**: Automatic generation of service dependencies from traces
- **RED Metrics**: Rate, Errors, Duration - automatically computed from trace data

**Example Workflow:**

```
1. Grafana alert triggers: "Error rate > 5%"
2. Navigate to Tempo → view recent traces with errors
3. Identify that 90% of errors come from database connection timeout
4. Check database logs in Loki
5. See that database was restarted 5 minutes ago
6. Trigger automatic failover or alert on-call team
```

**Tools Used:** Tempo (tracing + RED metrics), Prometheus (metrics), Loki (logs)

---

### 4. **Log Analysis and Log-Based Debugging**

**Scenario:** An application error occurs and you need to investigate what went wrong.

**How the observability stack helps:**

- **Loki**: Stores logs with labels (namespace, pod, container, service)
- **Search**: Find logs by multiple criteria (timestamps, labels, content)
- **Trace Integration**: Jump from a trace to its corresponding logs
- **Pattern Detection**: Identify common error patterns

**Example Workflow:**

```
1. Application crashes with error code
2. Search Loki for: {namespace="production", service="api"} | "OutOfMemoryError"
3. Find 15 matching log lines across 5 pods
4. Click on one to see full context
5. Scroll up to see what happened before the crash
6. Correlate with metrics using timestamps
7. Find that crash coincides with high memory usage spike
```

**Tools Used:** Loki (logs), Grafana (search and correlation)

---

### 5. **Cluster Health Monitoring**

**Scenario:** You need constant visibility into cluster health and want to be alerted about problems.

**How the observability stack helps:**

- **Prometheus**: Monitors kube-state-metrics and kubelet metrics
- **Alertmanager**: Sends alerts for:
  - Node NotReady
  - High CPU/memory usage
  - Pod crashes and restarts
  - Persistent volume filling up
  - API server latency
- **Dashboards**: Visual representation of cluster health

**Example Workflow:**

```
1. Node runs out of disk space
2. Prometheus alert fires: "Node disk > 90%"
3. Alertmanager sends Slack notification
4. On-call engineer acknowledges alert in Grafana
5. Investigates using Node Exporter dashboard
6. Finds that old logs are consuming space
7. Logs are rotated, alarm clears
8. Alert acknowledgement auto-closes
```

**Tools Used:** Prometheus (metrics), Alertmanager (alerting), Grafana (dashboards)

---

### 6. **CI/CD Pipeline Observability**

**Scenario:** You run GitOps (ArgoCD) and want to monitor deployments and their impact.

**How the observability stack helps:**

- **Traces**: Track deployment through build, test, and deploy stages
- **Metrics**: Automatically capture deployment duration, success rate, and resource changes
- **Logs**: Collect logs from CI/CD tools (ArgoCD, Jenkins, etc.)
- **Dashboards**: Pre-built ArgoCD dashboard showing sync status, health, and metrics

**Example Workflow:**

```
1. ArgoCD syncs new version of payment service
2. Prometheus automatically detects:
   - Pod replacement rollout
   - Resource requests increase/decrease
   - Traffic rerouting
3. Tempo traces show new request latency
4. System alerts if error rate increases post-deployment
5. You can automatically rollback if metrics deviate from baseline
```

**Tools Used:** Prometheus (metrics), Tempo (tracing), Loki (logs), Grafana (ArgoCD dashboard)

---

### 7. **Multi-Cloud and Hybrid Cluster Visibility**

**Scenario:** You have multiple Kubernetes clusters (prod, staging, dev, different regions/clouds).

**How the observability stack helps:**

- **Central Stack**: Deploy k8s-Observability to each cluster independently OR to a central monitoring cluster
- **Unified Dashboards**: Single Grafana instance with datasources pointing to all clusters
- **Alerting**: Centralized alert routing across all clusters
- **Cost Allocation**: Track resource usage per cluster and per customer

**Example Workflow:**

```
1. Central Grafana dashboard shows metrics from:
   - AWS EKS cluster (prod)
   - AWS EKS cluster (staging)
   - On-prem Kubernetes cluster
2. Alert on any cluster shows which cluster it came from
3. Each application's traces are collected and correlated
4. Single pane of glass for entire infrastructure
```

**Tools Used:** Prometheus (federation), Grafana (multi-datasource), Loki, Tempo

---

### 8. **Performance Baseline and Regression Detection**

**Scenario:** You want to detect performance regressions before customers notice them.

**How the observability stack helps:**

- **Historical Metrics**: 7+ days of metric history enables trend analysis
- **Baseline Comparison**: Compare current metrics to baseline from last week/month
- **Anomaly Detection**: Set thresholds that alert on unusual behavior
- **Trace Comparison**: Compare trace latencies between code versions

**Example Workflow:**

```
1. New code is deployed
2. Prometheus data collection continues
3. Tempo captures traces from new code
4. Baseline comparison shows: API latency increased by 25%
5. Alert fires: "Performance regression detected"
6. You compare traces from old vs new code
7. Find the slow database query in new code
8. Rollback or fix the query
```

**Tools Used:** Prometheus (historical data), Tempo (tracing), Grafana (baseline comparison)

---

### 9. **Dependency and Network Analysis**

**Scenario:** You need to understand how services communicate and identify bottlenecks.

**How the observability stack helps:**

- **Service Maps**: Automatic generation from trace data showing:
  - Which services call which other services
  - Request count per edge
  - Error rate per edge
  - Latency per edge
- **Network Metrics**: Pod-to-pod communication patterns
- **Dependency Visualization**: Identify single points of failure

**Example Workflow:**

```
1. Service map shows: service-a → service-b → database
2. Every call from service-a to service-b adds 200ms latency
3. You investigate: service-b has timeout to database
4. Optimize: add connection pooling
5. Latency drops to 10ms
6. Service map updates automatically
```

**Tools Used:** Tempo (service maps), Prometheus (network metrics)

---

### 10. **Cost Optimization and Resource Planning**

**Scenario:** Cloud costs are high and you need to understand where money is being spent.

**How the observability stack helps:**

- **Metrics**: Track CPU and memory requests vs actual usage
- **Historical Analysis**: Identify over-provisioned resources
- **Right-sizing**: Know which services need bigger/smaller instances
- **Utilization Metrics**: Understand when traffic spikes occur and capacity needed

**Example Workflow:**

```
1. Prometheus metrics show:
   - Service A: requests 8 cores, uses 0.5 cores average
   - Service B: requests 2 cores, uses 1.8 cores average
2. Recommendation: reduce A's requests, increase B's requests
3. Implement changes
4. Cost drops by $2000/month without impacting performance
```

**Tools Used:** Prometheus (metrics), Grafana (visualization)

---

## Prerequisites

### Cluster Requirements

- **Kubernetes 1.20+**
- **Helm 3.10+**
- **Storage Class**: At least one storage class available (e.g., `gp3` for AWS EBS)
- **Minimum Compute**:
  - Control Plane: Standard cluster nodes
  - Worker Nodes: At least 2 nodes with 2 CPU and 4GB RAM each
  - **Recommended**: Dedicated observability node or node group with `intent=observability` label

### AWS EKS Specific

- **AWS ALB Ingress Controller**: For Grafana ingress to work (or substitute your ingress controller)
- **IAM Permissions**: For reading EBS volume attributes (if using EBS as storage class)
- **Security Groups**: Allow ingress on ports 80/443 for Grafana

### Network Requirements

- Internal pod-to-pod communication must be allowed
- DNS resolution for service discovery must work
- Outbound HTTPS for downloading Helm chart dependencies (optional if using local charts)

---

## Quick Start

### 1. Clone or Download the Repository

```bash
git clone https://github.com/your-org/k8s-Observability.git
cd k8s-Observability
```

### 2. View Helm Chart

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update
```

### 3. Preview the Templates (Dry Run)

```bash
helm template observability . --namespace observability
```

Or with a custom values file:

```bash
helm template observability . -f my-values.yaml --namespace observability
```

### 4. Install the Chart

```bash
helm upgrade --install observability . \
  --namespace observability \
  --create-namespace
```

**With custom values:**

```bash
helm upgrade --install observability . \
  --namespace observability \
  --create-namespace \
  -f my-values.yaml
```

### 5. Verify Installation

```bash
# Check all pods are running
kubectl get pods -n observability

# Check services
kubectl get svc -n observability

# Check persistent volumes
kubectl get pvc -n observability
```

Expected output (after ~2-3 minutes):

```
NAME                                             READY   STATUS    RESTARTS   AGE
observability-grafana-xxxxx                      1/1     Running   0          2m
observability-kube-prometh-prometheus-0          1/1     Running   0          2m
observability-loki-0                             1/1     Running   0          2m
observability-tempo-0                            1/1     Running   0          2m
observability-alloy-xxxxx                        1/1     Running   0          2m
...
```

---

## Chart Components

### **1. Prometheus (via `kube-prometheus-stack`)**

**Purpose:** Metrics collection, alerting engine, and time-series database

**What it monitors:**

- Kubernetes metrics: node CPU/memory, pod usage, container stats
- Application metrics: request count, latency, errors (if apps export Prometheus metrics)
- Cluster health: API server latency, etcd performance, kubelet health

**Key Features:**

- **7-day retention** (configurable) of all metrics
- **Service Discovery**: Automatically finds and scrapes metrics from:
  - `ServiceMonitor` resources
  - `PodMonitor` resources
  - StaticConfigs in prometheus.yaml
- **Alerting**: AlertManager component for routing alerts to Slack/PagerDuty/email
- **Remote Write**: Can push metrics to external storage (configured to send to Alloy)

**Configuration in values.yaml:**

```yaml
kube-prometheus-stack:
  prometheus:
    prometheusSpec:
      retention: 168h # 7 days
      scrapeInterval: 30s # How often to scrape metrics
      remoteWrite:
        - url: "http://observability-alloy.observability.svc.cluster.local:9080/api/v1/prom/receive"
```

**Useful Prometheus Queries:**

```promql
# Pod CPU usage
sum(rate(container_cpu_usage_seconds_total[5m])) by (pod)

# Pod memory usage
sum(container_memory_usage_bytes) by (pod)

# Request rate by service
sum(rate(http_requests_total[5m])) by (service)

# Error rate
sum(rate(http_requests_total{status=~"5.."}[5m])) by (service)
```

---

### **2. Grafana**

**Purpose:** Unified visualization and dashboarding platform

**What it provides:**

- **Web Dashboard**: Beautiful, interactive dashboards with time-series charts
- **Data Sources**: Connected to Prometheus (metrics), Loki (logs), and Tempo (traces)
- **Alerting**: Create alerts based on metric thresholds (e.g., "if CPU > 80%")
- **User Management**: Multi-user support with role-based access control
- **Plugins**: Install additional visualization plugins (explore traces, status panels, etc.)
- **Annotations**: Mark events on dashboards (deployments, incidents, etc.)

**Default Credentials:**

- Username: `admin`
- Password: `admin` (configurable in values.yaml)

**Pre-configured Datasources:**

1. **Prometheus**: `http://observability-kube-prometh-prometheus.observability.svc.cluster.local:9090`
2. **Loki**: `http://observability-loki.observability.svc.cluster.local:3100`
3. **Tempo**: `http://observability-tempo.observability.svc.cluster.local:3200`

**Access Method:**

```bash
# Port forward for local access
kubectl port-forward -n observability svc/observability-grafana 3000:80

# Then visit: http://localhost:3000
```

Or via ingress (for AWS ALB):

```
https://domain.com/grafana
```

---

### **3. Loki**

**Purpose:** Log aggregation and storage optimized for Kubernetes

**What it collects:**

- Pod logs from all containers
- System logs from kubelet, kube-proxy, etc.
- Application logs in JSON or plain text format
- Structured logs with metadata (namespace, pod name, container, etc.)

**How it works:**

1. **Alloy DaemonSet** runs on every node and collects logs
2. Logs are parsed and labeled with metadata (namespace, pod, container, app)
3. Logs are sent to Loki for storage
4. Loki indexes only the labels (efficient) and stores raw log text
5. Grafana queries Loki using LogQL

**Key Features:**

- **7-day retention** for most logs
- **Per-stream retention**: Different namespaces can have different retention
  - System namespaces (kube-system, observability): 3 days
  - User namespaces: 14 days
- **Pattern detection**: Identify common log patterns and errors
- **Log context**: Easily jump to logs during a specific time period

**Storage:**

- Uses local filesystem storage (`/tmp/loki/chunks`)
- Configured with 20GB persistent volume for scalability
- Automatic compaction and cleanup

**Sample LogQL Queries:**

```logql
# All logs from a pod
{pod="my-app-abc123"}

# Error logs from a namespace
{namespace="production"} | "error"

# Logs with status code 500
{namespace="production"} | json status=status | status=500

# Count of errors over time
count_over_time({namespace="production"} | "error" [1m])
```

---

### **4. Tempo**

**Purpose:** Distributed tracing backend for capturing request flows across services

**What it captures:**

- Full request trace from entry point through all services
- Span data: service name, operation, duration, status, attributes
- Automatic service map generation
- Generated RED metrics (Request rate, Error rate, Duration)

**How it works:**

1. **Application or middleware** instruments code using OpenTelemetry SDK
2. Spans are sent to Alloy OTLP receiver (port 4317 gRPC, 4318 HTTP)
3. Alloy forwards to Tempo for storage
4. Tempo stores traces and generates metrics
5. Grafana visualizes traces and correlates with metrics/logs

**Key Features:**

- **Trace correlation**: Jump from a trace to metrics and logs using timestamps
- **Service maps**: Automatic visualization of service dependencies and call rates
- **Span search**: Find traces by service, operation, duration, error status
- **Metrics from traces**: RED metrics and Service Graph metrics automatically computed

**Storage:**

- Uses 10GB persistent volume for trace data
- Automatic TTL (Time To Live) management

**Sending Traces to Tempo (OTLP Protocol):**

```python
from opentelemetry import trace
from opentelemetry.exporter.otlp.proto.grpc.trace_exporter import OTLPSpanExporter
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.sdk.trace.export import BatchSpanProcessor

exporter = OTLPSpanExporter(endpoint="observability-tempo.observability.svc.cluster.local:4317")
trace.set_tracer_provider(TracerProvider())
trace.get_tracer_provider().add_span_processor(BatchSpanProcessor(exporter))

tracer = trace.get_tracer(__name__)
with tracer.start_as_current_span("my-operation"):
    # Your code here
    pass
```

---

### **5. Alloy**

**Purpose:** Unified telemetry collector and aggregator (formerly Grafana Agent)

**What it does:**

1. **Log Collection**: DaemonSet on every node collecting pod logs
2. **Metric Relay**: Receives metrics from Prometheus, optionally proxies them
3. **Trace Reception**: OTLP receiver for traces from applications
4. **Processing**: Apply transformations, filtering, and sampling before storage

**Key Features:**

- **DaemonSet Mode**: One pod per node to collect logs from all containers
- **OTLP Receivers**:
  - gRPC on port 4317
  - HTTP on port 4318
- **Log Processing Pipeline**:
  - Auto-label logs with namespace, pod, container, app
  - Drop old logs
  - Parse JSON logs and extract fields
  - Send to Loki

**Resource Usage:**

- Very lightweight: 100m CPU, 128MB memory requests
- Can run on all nodes including control plane (with appropriate tolerations)

---

## Configuration

### Customizing Components

Edit `values.yaml` to customize each component:

#### **Disable a component:**

```yaml
grafana:
  enabled: false

loki:
  enabled: false
```

#### **Change Grafana admin password:**

```yaml
grafana:
  adminUser: admin
  adminPassword: your-secure-password
```

#### **Change ingress hostname:**

```yaml
grafana:
  ingress:
    hosts:
      - your-domain.com
```

#### **Change storage class or size:**

```yaml
grafana:
  persistence:
    storageClassName: my-storage-class
    size: 10Gi

loki:
  singleBinary:
    persistence:
      storageClass: my-storage-class
      size: 50Gi
```

#### **Adjust resource limits:**

```yaml
prometheus:
  prometheusSpec:
    resources:
      requests:
        cpu: 500m
        memory: 2Gi
      limits:
        cpu: 1000m
        memory: 4Gi
```

#### **Change log retention:**

```yaml
loki:
  limits_config:
    retention_period: 336h # 14 days instead of 7
```

### Node Scheduling

To run observability components on dedicated observability nodes:

1. Label the nodes:

```bash
kubectl label nodes <node-name> intent=observability
```

2. Uncomment nodeSelector and tolerations in values.yaml:

```yaml
grafana:
  nodeSelector:
    intent: observability
  tolerations:
    - key: "observability"
      operator: "Equal"
      value: "true"
      effect: "NoSchedule"
```

---

## Accessing the Dashboard

### **Local Access (Port Forward)**

```bash
kubectl port-forward -n observability svc/observability-grafana 3000:80
```

Then navigate to: **http://localhost:3000**

### **via Kubernetes Ingress (AWS ALB)**

The chart is pre-configured with AWS ALB ingress. To use it:

1. **Ensure AWS Load Balancer Controller is installed:**

```bash
helm repo add eks https://aws.github.io/eks-charts
helm install aws-load-balancer-controller eks/aws-load-balancer-controller -n kube-system
```

2. **Update the ingress hostname in values.yaml:**

```yaml
grafana:
  ingress:
    hosts:
      - your-actual-domain.com
```

3. **Apply certificate ARN (if using HTTPS):**

```yaml
grafana:
  ingress:
    annotations:
      alb.ingress.kubernetes.io/certificate-arn: arn:aws:acm:region:account:certificate/id
```

4. Access via: **https://your-domain.com**

### **Default Credentials**

- **Username:** `admin`
- **Password:** `admin` (change immediately in production!)

### **First Steps in Grafana**

1. **Change admin password**:
   - Click profile icon (bottom left)
   - Change Password
   - Use a strong password

2. **Explore datasources**:
   - Configuration → Data Sources
   - Should see: Prometheus, Loki, Tempo (all green and working)

3. **View pre-configured dashboards** (if using community dashboards):
   - Dashboards → Browse
   - Look for: Kubernetes, Node Exporter, ArgoCD, Kafka dashboards

4. **Create a simple dashboard**:
   - Dashboards → Create → New Dashboard
   - Add a Prometheus metric: `node_cpu_seconds_total`
   - Add a Loki log query: `{namespace="kube-system"}`

---

## Customization

### **Adding Custom Dashboards**

1. **In Grafana UI**:
   - Create dashboard
   - Save it
   - Share → Copy the JSON model

2. **In Helm Chart** (for automatic provisioning):

```yaml
grafana:
  dashboards:
    default:
      my-custom-dashboard:
        json: |
          { /* paste JSON here */ }
```

### **Adding Alerting Rules**

```yaml
kube-prometheus-stack:
  prometheusRules:
    - name: custom-alerts
      groups:
        - name: my-services
          rules:
            - alert: HighErrorRate
              expr: rate(http_requests_total{job="my-app",status=~"5.."}[5m]) > 0.05
              for: 5m
              annotations:
                summary: "High error rate detected for {{ $labels.service }}"
```

### **Integrating with External Systems**

**Slack Notifications:**

```yaml
kube-prometheus-stack:
  alertmanager:
    config:
      route:
        receiver: slack
      receivers:
        - name: slack
          slack_configs:
            - api_url: https://hooks.slack.com/services/YOUR/WEBHOOK/URL
              channel: "#alerts"
```

**PagerDuty:**

```yaml
kube-prometheus-stack:
  alertmanager:
    config:
      receivers:
        - name: pagerduty
          pagerduty_configs:
            - service_key: YOUR_SERVICE_KEY
```

---

---

## Tools Usage Guide

### **Prometheus: Complete Usage Guide**

#### **Access Prometheus Web UI**

```bash
# Port forward for local access
kubectl port-forward -n observability svc/observability-kube-prometh-prometheus 9090:9090

# Visit: http://localhost:9090
```

#### **Key Prometheus UI Features**

1. **Graph Tab - Interactive Querying**
   - Query for any metric and see real-time graphs
   - Example queries:

     ```promql
     # CPU usage per pod
     sum(rate(container_cpu_usage_seconds_total[5m])) by (pod_name)

     # Memory usage
     container_memory_usage_bytes{pod_name!=""}

     # HTTP request rate
     sum(rate(http_requests_total[5m])) by (status)

     # API latency percentiles
     histogram_quantile(0.95, sum(rate(http_request_duration_seconds_bucket[5m])) by (le))
     ```

2. **Alerts Tab - View Active Alerts**
   - See all firing alerts
   - View alert evaluation status
   - Check last evaluation time

3. **Targets Tab - Check Data Sources**
   - See which targets Prometheus is scraping
   - Status: UP (healthy) or DOWN (problems)
   - Scrape duration and last scrape time
   - Try to debug: If scrape is failing, see the error

4. **ServiceMonitor Discovery**
   - Prometheus automatically discovers `ServiceMonitor` resources
   - Example creating a monitor:
     ```yaml
     apiVersion: monitoring.coreos.com/v1
     kind: ServiceMonitor
     metadata:
       name: my-app-monitor
       namespace: production
     spec:
       selector:
         matchLabels:
           app: my-app
       endpoints:
         - port: metrics
           interval: 30s
           path: /metrics
     ```

#### **Common Prometheus Queries**

```promql
# Node-level metrics
node_cpu_seconds_total           # CPU time in seconds
node_memory_MemAvailable_bytes   # Available memory
node_disk_io_reads_completed     # Disk read count

# Pod/Container metrics
container_cpu_usage_seconds_total     # CPU usage per container
container_memory_usage_bytes           # Memory per container
container_network_receive_bytes_total  # Network RX

# Kubernetes metrics
kube_pod_status_phase{phase="Running"}     # Running pods count
kube_node_status_condition{condition="Ready",status="true"}  # Healthy nodes
kube_pod_restart_total                     # Pod restarts

# Custom application metrics (if your app exports them)
http_requests_total{job="my-app"}          # HTTP requests
http_request_duration_seconds_bucket       # Request latency histogram
```

#### **Recording Rules (For Performance)**

Create rules to pre-compute expensive queries:

```yaml
kube-prometheus-stack:
  prometheusRules:
    - name: compute-heavy
      groups:
        - name: precompute
          interval: 30s
          rules:
            - record: instance:node_cpu:rate5m
              expr: sum(rate(node_cpu_seconds_total[5m])) by (instance)

            - record: instance:node_memory:ratio
              expr: node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes
```

---

### **Grafana: Complete Usage Guide**

#### **Access Grafana Dashboard**

```bash
# Option 1: Port forward
kubectl port-forward -n observability svc/observability-grafana 3000:80
# Visit: http://localhost:3000

# Option 2: AWS ALB Ingress
# Visit: https://your-domain.com
```

#### **First Login & Configuration**

```
Username: admin
Password: admin (default - MUST CHANGE!)
```

**Change Password:**

1. Click profile icon (bottom-left)
2. Select "Change Password"
3. Enter new password
4. Click "Change Password"

#### **Navigate Grafana Dashboards**

1. **Home Dashboard** (default)
   - Shows all favorite dashboards
   - Quick access to recent dashboards

2. **Dashboards → Browse**
   - All available dashboards listed
   - Create new dashboard
   - Import community dashboards

3. **Explore (Flux icon)**
   - Ad-hoc queries without saving
   - Choose datasource (Prometheus, Loki, Tempo)
   - Real-time data exploration

#### **Creating a Custom Dashboard**

```
1. Click "Create" (+ icon, top left)
2. Select "Dashboard"
3. Click "Add a new panel"
4. Choose datasource (e.g., Prometheus)
5. Write query
6. Configure visualization (Graph, Stat, Table, etc.)
7. Set title and description
8. Click "Save dashboard"
```

#### **Example Metrics Panels**

**Container CPU Usage:**

```promql
sum(rate(container_cpu_usage_seconds_total{pod_name!=""}[5m])) by (pod_name)
```

- Visualization: Graph
- Y-axis label: "CPU Cores"

**Memory Usage:**

```promql
container_memory_usage_bytes{pod_name!=""} / 1024 / 1024
```

- Visualization: Stat / Gauge
- Unit: "bytes (SI)"
- Thresholds: 50% yellow, 80% red

**Request Rate & Error Rate:**

```promql
sum(rate(http_requests_total[5m])) by (status)
```

- Visualization: Graph with legend

#### **Datasource Connection Status**

Verify all datasources are connected:

1. Configuration → Data Sources
2. Should see:
   - ✅ **Prometheus** - green
   - ✅ **Loki** - green
   - ✅ **Tempo** - green

If any shows red ❌, click to troubleshoot.

---

### **Loki: Complete Usage Guide**

#### **Access Loki Query Interface (via Grafana)**

```
Grafana → Explore → Select "Loki" from datasource dropdown
```

#### **LogQL Query Language**

LogQL = Prometheus-like query language for logs

**Basic Syntax:**

```logql
# Select logs by label
{job="my-app"}                    # All logs from my-app
{namespace="production"}          # All logs from production
{namespace="prod", status="error"}  # Multiple labels (AND)

# Filter by content
{job="my-app"} |= "error"         # Contains "error"
{job="my-app"} != "DEBUG"         # Does NOT contain "DEBUG"
{job="my-app"} |~ "timeout|slow"  # Regex match
```

**Metric Queries (Aggregate logs):**

```logql
# Count logs over time
count_over_time({namespace="production"} [5m])

# Sum/average/max/min operations
sum(bytes_total{job="my-app"}) by (level)

# Rate of errors
rate({job="my-app"} | "ERROR" [5m])
```

#### **Example Log Searches**

**Find All Errors in Production:**

```logql
{namespace="production"} | level="ERROR"
```

**Find HTTP 500 errors:**

```logql
{namespace="production"} | json | status=500
```

**Count requests by pod per minute:**

```logql
sum(count_over_time({namespace="production"} [1m])) by (pod)
```

**Show logs from specific time range:**

```logql
{namespace="production"} | "OutOfMemory"
```

- Then select time range in Grafana UI

#### **Log Retention Configuration**

In `values.yaml`, control log retention:

```yaml
loki:
  limits_config:
    retention_period: 168h # 7 days global
    retention_stream:
      - selector: '{namespace="system"}'
        period: 72h # 3 days for system logs
      - selector: '{namespace="production"}'
        period: 336h # 14 days for prod
```

---

### **Tempo: Complete Usage Guide**

#### **Access Tempo Traces (via Grafana)**

```
Grafana → Explore → Select "Tempo" from datasource dropdown
```

#### **Finding Traces**

1. **By Service Name**
   - Service name dropdown
   - Select your service

2. **By Operation**
   - Operation dropdown
   - Find specific endpoint (e.g., `/api/users`)

3. **By Duration**
   - Minimum duration: `100ms`
   - Maximum duration: `5s`

4. **By Status**
   - All
   - Success
   - Error

5. **By Tags**
   - Add custom tags: `user.id=123`
   - `http.method=GET`
   - `database=postgres`

#### **Understanding Trace Details**

When you click on a trace, you see:

```
Service A (operation="/api/users", duration=245ms)
  ├─ Service B (operation="GetUser", duration=150ms)
  │   └─ Database (operation="SELECT", duration=100ms)
  └─ Service C (operation="ValidateUser", duration=50ms)
```

Each span shows:

- **Operation name**: What was executed
- **Duration**: How long it took
- **Status**: Success or Error
- **Tags**: Custom metadata
- **Logs**: Events that occurred
- **Parent trace ID**: For correlation

#### **Service Map Visualization**

Tempo automatically generates a service map showing:

- Box for each service
- Arrows showing request flow
- Request count on each arrow
- Error count on each arrow
- Average latency on each arrow

#### **Sending Traces from Your Application**

Your app must export traces using OpenTelemetry. See the [Sample Trace Generation](#sample-trace-generation-code) section below.

---

### **Alloy: Complete Usage Guide**

#### **What Alloy Does**

Alloy is a universal telemetry collector that:

1. **Collects logs** from every pod (DaemonSet)
2. **Receives metrics** from Prometheus scraping
3. **Receives traces** via OTLP protocol
4. **Processes** data before sending to backends

#### **Access Alloy Configuration**

```bash
# View Alloy ConfigMap
kubectl get configmap -n observability observability-alloy

# Edit configuration
kubectl edit configmap observability-alloy -n observability

# View Alloy logs
kubectl logs -n observability -l app=alloy -f
```

#### **Alloy Configuration Sections**

In `values.yaml`, the Alloy config includes:

1. **Kubernetes Pod Discovery**
   - Automatically finds all pods in the cluster
   - Extracts labels (namespace, pod name, container, app)

2. **Log Processing Pipeline**
   - Receives logs from containers
   - Parses JSON if present
   - Extracts and labels fields
   - Drops old logs
   - Sends to Loki

3. **OTLP Receivers** (for traces)
   - gRPC on port 4317
   - HTTP on port 4318
   - Forwards to Tempo

#### **Check Alloy Health**

```bash
# Check if Alloy is running
kubectl get pods -n observability -l app=alloy

# View recent logs
kubectl logs -n observability -l app=alloy --tail=50

# Check for errors
kubectl logs -n observability -l app=alloy | grep -i error
```

#### **Custom Log Processing**

To add custom log processing rules, edit `values.yaml`:

```yaml
alloy:
  alloy:
    configMap:
      content: |-
        # Add custom stage rules here
        stage.match {
          selector = "{job=\"my-app\"}"
          stage.json {
            expressions = {
              severity = "\"severity\"",
              trace_id = "\"traceID\"",
            }
          }
          stage.labels {
            values = {
              severity = "severity",
              trace_id = "trace_id",
            }
          }
        }
```

---

## Sample Trace Generation Code

### **1. Python - OpenTelemetry Trace Generation**

**Install dependencies:**

```bash
pip install opentelemetry-api opentelemetry-sdk opentelemetry-exporter-otlp
```

**Python code to send traces:**

```python
from opentelemetry import trace, metrics
from opentelemetry.exporter.otlp.proto.grpc.trace_exporter import OTLPSpanExporter
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.sdk.trace.export import BatchSpanProcessor
from opentelemetry.sdk.resources import Resource
import time
import random
import logging

# Configure logging
logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

# Create resource (identifies your application)
resource = Resource.create({
    "service.name": "payment-service",
    "service.version": "1.0.0",
    "deployment.environment": "production"
})

# Create OTLP exporter (points to Tempo via Alloy)
otlp_exporter = OTLPSpanExporter(
    endpoint="observability-alloy.observability.svc.cluster.local:4317",
    insecure=True
)

# Create tracer provider
tracer_provider = TracerProvider(resource=resource)
tracer_provider.add_span_processor(BatchSpanProcessor(otlp_exporter))
trace.set_tracer_provider(tracer_provider)

# Get tracer instance
tracer = trace.get_tracer(__name__)

def process_payment(user_id: str, amount: float):
    """Simulates a payment processing flow with multiple spans."""

    with tracer.start_as_current_span("process_payment") as span:
        span.set_attribute("user.id", user_id)
        span.set_attribute("amount", amount)
        span.set_attribute("currency", "USD")

        logger.info(f"Processing payment for user {user_id}")

        try:
            # Validate user
            with tracer.start_as_current_span("validate_user") as validate_span:
                validate_span.set_attribute("user.id", user_id)
                time.sleep(random.uniform(0.1, 0.3))
                validate_span.set_attribute("result", "valid")

            # Check fraud
            with tracer.start_as_current_span("fraud_check") as fraud_span:
                fraud_span.set_attribute("user.id", user_id)
                fraud_span.set_attribute("amount", amount)
                time.sleep(random.uniform(0.1, 0.5))
                is_fraud = random.random() < 0.05

                if is_fraud:
                    fraud_span.set_attribute("fraud_detected", True)
                    fraud_span.set_attribute("status", "blocked")
                    raise Exception("Fraudulent transaction detected")

                fraud_span.set_attribute("fraud_detected", False)

            # Debit account
            with tracer.start_as_current_span("debit_account") as debit_span:
                debit_span.set_attribute("user.id", user_id)
                debit_span.set_attribute("amount", amount)
                time.sleep(random.uniform(0.2, 0.4))
                debit_span.set_attribute("status", "success")

            # Send confirmation email
            with tracer.start_as_current_span("send_email") as email_span:
                email_span.set_attribute("user.id", user_id)
                email_span.set_attribute("email_type", "payment_confirmation")
                time.sleep(random.uniform(0.1, 0.3))
                email_span.set_attribute("status", "sent")

            span.set_attribute("transaction.status", "completed")
            return {"status": "success", "user_id": user_id}

        except Exception as e:
            span.set_attribute("transaction.status", "failed")
            span.set_attribute("error.message", str(e))
            span.record_exception(e)
            raise

if __name__ == "__main__":
    # Generate sample traces
    for i in range(10):
        try:
            user_id = f"user_{random.randint(1000, 9999)}"
            amount = round(random.uniform(10, 500), 2)
            result = process_payment(user_id, amount)
            print(f"✓ Transaction {i+1}: {result}")
        except Exception as e:
            print(f"✗ Transaction {i+1} failed: {e}")

        time.sleep(2)

    # Flush remaining spans
    tracer_provider.force_flush()
    print("\nAll traces sent to Tempo!")
```

**Run in Kubernetes pod:**

```bash
# Create a pod to generate traces
kubectl run trace-generator \
  --image=python:3.10 \
  --restart=Never \
  -n observability \
  -- bash -c "
    pip install opentelemetry-api opentelemetry-sdk opentelemetry-exporter-otlp &&
    python /etc/config/trace_generator.py
  "

# View in Grafana: Explore → Tempo → Filter by service_name=payment-service
```

---

### **2. Node.js/JavaScript - OpenTelemetry Trace Generation**

**Install dependencies:**

```bash
npm install @opentelemetry/api @opentelemetry/sdk-node @opentelemetry/sdk-trace-node @opentelemetry/exporter-trace-otlp-grpc
```

**JavaScript code:**

```javascript
const { NodeSDK } = require("@opentelemetry/sdk-node");
const {
  getNodeAutoInstrumentations,
} = require("@opentelemetry/auto-instrumentations-node");
const {
  OTLPTraceExporter,
} = require("@opentelemetry/exporter-trace-otlp-grpc");
const { Resource } = require("@opentelemetry/resources");
const {
  SemanticResourceAttributes,
} = require("@opentelemetry/semantic-conventions");

// Create OTLP exporter
const exporter = new OTLPTraceExporter({
  url: "http://observability-alloy.observability.svc.cluster.local:4317",
});

// Create resource
const resource = Resource.default().merge(
  new Resource({
    [SemanticResourceAttributes.SERVICE_NAME]: "order-service",
    [SemanticResourceAttributes.SERVICE_VERSION]: "1.0.0",
  }),
);

// Initialize SDK
const sdk = new NodeSDK({
  resource: resource,
  traceExporter: exporter,
  instrumentations: [getNodeAutoInstrumentations()],
});

sdk.start();

// Import after SDK initialization
const { trace } = require("@opentelemetry/api");
const tracer = trace.getTracer("order-service-tracer");

// Example: Process order with tracing
async function processOrder(orderId, userId) {
  const span = tracer.startSpan("process_order");
  span.setAttributes({
    "order.id": orderId,
    "user.id": userId,
  });

  try {
    // Validate inventory
    const inventorySpan = tracer.startSpan("check_inventory", {
      parent: span,
    });
    await sleep(200);
    inventorySpan.end();

    // Process payment
    const paymentSpan = tracer.startSpan("process_payment", {
      parent: span,
    });
    await callPaymentAPI(orderId, userId);
    paymentSpan.end();

    // Update database
    const dbSpan = tracer.startSpan("update_database", {
      parent: span,
    });
    await updateOrderStatus(orderId, "completed");
    dbSpan.end();

    span.setStatus({ code: 0 });
  } catch (error) {
    span.recordException(error);
    span.setStatus({ code: 2, message: error.message });
  } finally {
    span.end();
  }
}

// Helper functions
function sleep(ms) {
  return new Promise((resolve) => setTimeout(resolve, ms));
}

async function callPaymentAPI(orderId, userId) {
  console.log(`Processing payment for order ${orderId}`);
  await sleep(300);
}

async function updateOrderStatus(orderId, status) {
  console.log(`Updated order ${orderId} to ${status}`);
  await sleep(100);
}

// Generate sample traces
async function generateTraces() {
  for (let i = 0; i < 5; i++) {
    const orderId = `order_${Date.now()}_${i}`;
    const userId = `user_${Math.floor(Math.random() * 1000)}`;

    try {
      await processOrder(orderId, userId);
      console.log(`✓ Order ${orderId} processed`);
    } catch (error) {
      console.log(`✗ Order ${orderId} failed: ${error.message}`);
    }

    await sleep(1000);
  }

  // Shutdown after delays
  setTimeout(() => {
    sdk.shutdown().then(() => console.log("SDK shut down successfully"));
  }, 5000);
}

generateTraces();
```

---

### **3. Go - OpenTelemetry Trace Generation**

**Install dependencies:**

```bash
go get go.opentelemetry.io/otel
go get go.opentelemetry.io/otel/exporter/otlp/otlptrace/otlptracegrpc
go get go.opentelemetry.io/otel/sdk/trace
```

**Go code:**

```go
package main

import (
	"context"
	"fmt"
	"log"
	"math/rand"
	"time"

	"go.opentelemetry.io/otel"
	"go.opentelemetry.io/otel/attribute"
	"go.opentelemetry.io/otel/exporters/otlp/otlptrace/otlptracegrpc"
	"go.opentelemetry.io/otel/sdk/resource"
	sdktrace "go.opentelemetry.io/otel/sdk/trace"
	semconv "go.opentelemetry.io/otel/semconv/v1.17.0"
)

func initTracer() (*sdktrace.TracerProvider, error) {
	// Create OTLP exporter
	exporter, err := otlptracegrpc.New(context.Background(),
		otlptracegrpc.WithEndpoint("observability-alloy.observability.svc.cluster.local:4317"),
		otlptracegrpc.WithInsecure(),
	)
	if err != nil {
		return nil, err
	}

	// Create resource
	res, err := resource.New(context.Background(),
		resource.WithAttributes(
			semconv.ServiceNameKey.String("api-gateway"),
			semconv.ServiceVersionKey.String("1.0.0"),
		),
	)
	if err != nil {
		return nil, err
	}

	// Create tracer provider
	tp := sdktrace.NewTracerProvider(
		sdktrace.WithBatcher(exporter),
		sdktrace.WithResource(res),
	)

	otel.SetTracerProvider(tp)
	return tp, nil
}

func processRequest(ctx context.Context, tracer otel.Tracer, requestID string) error {
	ctx, span := tracer.Start(ctx, "process_request")
	defer span.End()

	span.SetAttributes(
		attribute.String("request.id", requestID),
		attribute.String("user.id", fmt.Sprintf("user_%d", rand.Intn(1000))),
	)

	// Authenticate user
	ctx, authSpan := tracer.Start(ctx, "authenticate_user")
	time.Sleep(time.Duration(rand.Intn(200)) * time.Millisecond)
	authSpan.End()

	// Call backend service
	ctx, serviceSpan := tracer.Start(ctx, "call_backend_service")
	time.Sleep(time.Duration(rand.Intn(500)) * time.Millisecond)
	serviceSpan.SetAttributes(
		attribute.String("service.name", "user-service"),
		attribute.Int("http.status_code", 200),
	)
	serviceSpan.End()

	// Cache result
	ctx, cacheSpan := tracer.Start(ctx, "cache_result")
	time.Sleep(time.Duration(rand.Intn(100)) * time.Millisecond)
	cacheSpan.End()

	return nil
}

func main() {
	tp, err := initTracer()
	if err != nil {
		log.Fatalf("failed to initialize tracer: %v", err)
	}
	defer func() {
		if err := tp.Shutdown(context.Background()); err != nil {
			log.Fatalf("failed to shutdown tracer: %v", err)
		}
	}()

	tracer := otel.GetTracerProvider().Tracer("api-gateway")

	// Generate sample traces
	for i := 0; i < 10; i++ {
		requestID := fmt.Sprintf("req_%d", time.Now().UnixNano())
		ctx := context.Background()

		if err := processRequest(ctx, tracer, requestID); err != nil {
			log.Printf("request failed: %v", err)
		}

		fmt.Printf("✓ Trace generated for request %s\n", requestID)
		time.Sleep(1 * time.Second)
	}

	fmt.Println("All traces sent to Tempo!")
}
```

---

### **4. Java - OpenTelemetry Trace Generation**

**Maven dependency:**

```xml
<dependency>
    <groupId>io.opentelemetry</groupId>
    <artifactId>opentelemetry-sdk</artifactId>
    <version>1.30.0</version>
</dependency>
<dependency>
    <groupId>io.opentelemetry.exporter</groupId>
    <artifactId>opentelemetry-exporter-otlp</artifactId>
    <version>1.30.0</version>
</dependency>
```

**Java code:**

```java
import io.opentelemetry.api.OpenTelemetry;
import io.opentelemetry.api.trace.Span;
import io.opentelemetry.api.trace.Tracer;
import io.opentelemetry.exporter.otlp.trace.OtlpGrpcSpanExporter;
import io.opentelemetry.sdk.OpenTelemetrySdk;
import io.opentelemetry.sdk.resources.Resource;
import io.opentelemetry.sdk.trace.SdkTracerProvider;
import io.opentelemetry.sdk.trace.export.BatchSpanProcessor;
import io.opentelemetry.semconv.resource.attributes.ResourceAttributes;
import java.util.concurrent.TimeUnit;

public class TraceGenerator {
    public static void main(String[] args) throws Exception {
        // Create OTLP exporter
        OtlpGrpcSpanExporter exporter = OtlpGrpcSpanExporter.builder()
                .setEndpoint("http://observability-alloy.observability.svc.cluster.local:4317")
                .build();

        // Create resource
        Resource resource = Resource.getDefault()
                .merge(Resource.create(io.opentelemetry.sdk.resources.ResourceAttributes
                        .from(io.opentelemetry.api.common.Attributes.builder()
                                .put(ResourceAttributes.SERVICE_NAME, "payment-service")
                                .put(ResourceAttributes.SERVICE_VERSION, "1.0.0")
                                .build())));

        // Create tracer provider
        SdkTracerProvider tracerProvider = SdkTracerProvider.builder()
                .addSpanProcessor(BatchSpanProcessor.builder(exporter).build())
                .setResource(resource)
                .build();

        // Create OpenTelemetry instance
        OpenTelemetry openTelemetry = OpenTelemetrySdk.builder()
                .setTracerProvider(tracerProvider)
                .build();

        Tracer tracer = openTelemetry.getTracer("payment-service");

        // Generate sample traces
        for (int i = 0; i < 5; i++) {
            Span span = tracer.spanBuilder("process_transaction")
                    .setAttribute("transaction.id", "txn_" + i)
                    .setAttribute("amount", 100.0 + i * 10)
                    .startSpan();

            try (var scope = span.makeCurrent()) {
                // Simulate processing
                Span validateSpan = tracer.spanBuilder("validate_payment").startSpan();
                try {
                    Thread.sleep(100);
                    validateSpan.setAttribute("validation.status", "passed");
                } finally {
                    validateSpan.end();
                }

                Span chargeSpan = tracer.spanBuilder("charge_card").startSpan();
                try {
                    Thread.sleep(200);
                    chargeSpan.setAttribute("charge.status", "success");
                } finally {
                    chargeSpan.end();
                }

                System.out.println("✓ Trace " + (i + 1) + " generated");
            } finally {
                span.end();
            }

            Thread.sleep(500);
        }

        // Shutdown
        tracerProvider.shutdown();
        System.out.println("All traces sent to Tempo!");
    }
}
```

---

### **Trace Generator Deployment Pod**

Use the included `trace-generator.yaml` to deploy a trace generator:

```bash
kubectl apply -f trace-generator.yaml

# View logs
kubectl logs -n monitoring k6-trace-generator -f

# View generated traces in Grafana
# Grafana → Explore → Tempo → Filter by service="k6-trace-generator"
```

---

## values.yaml Detailed Explanation

This section explains every configuration option in `values.yaml` line-by-line so you understand what each setting does and how to customize it.

### **Top-Level Namespace Configuration**

```yaml
namespace:
  name: observability
```

**Meaning:**

- **`namespace.name`**: The Kubernetes namespace where all observability components will be deployed
- **Default**: `observability`
- **Use case**: Change if you want to deploy to a different namespace like `monitoring` or `telemetry`
- **Example**:
  ```yaml
  namespace:
    name: monitoring # Deploy to 'monitoring' namespace instead
  ```

---

### **Global Settings (Optional)**

```yaml
# global:
#   nodeSelector: {}
#   tolerations:
#     - operator: "Exists"
#   affinity:
#     nodeAffinity:
#       preferredDuringSchedulingIgnoredDuringExecution:
#         - weight: 100
```

**Meaning:**

- **`global.nodeSelector`**: Apply node selection rules to ALL components (if they don't override)
- **`global.tolerations`**: Apply tolerations to all pods (useful if all nodes have taints)
- **`global.affinity`**: Apply affinity rules to all pods
- **These are commented out** because individual components have their own settings
- **When to use**: If you have dedicated observability nodes, uncomment these to apply to everything
- **Example**: Schedule everything on nodes labeled `intent=observability`
  ```yaml
  global:
    nodeSelector:
      intent: observability
  ```

---

### **GRAFANA Configuration**

```yaml
grafana:
  enabled: true
```

- **`enabled: true`**: Grafana will be installed (set to `false` to skip)

```yaml
adminUser: admin
adminPassword: admin
```

- **`adminUser`**: Default admin username for Grafana login
- **`adminPassword`**: Default admin password
- **⚠️ WARNING**: Change in production! Default `admin/admin` is a security risk
- **Change it:**
  ```yaml
  adminPassword: "YourSecurePassword123!"
  ```

#### **Grafana Service Configuration**

```yaml
service:
  type: ClusterIP
  port: 80
```

- **`service.type`**:
  - `ClusterIP`: (Default) Only accessible within cluster. Use port-forward for access
  - `NodePort`: Accessible on all nodes on a specific port (e.g., `localhost:30000`)
  - `LoadBalancer`: AWS creates an NLB (network load balancer)
- **`service.port`**: Port exposed inside the cluster
- **Example - NodePort access**:
  ```yaml
  service:
    type: NodePort
    port: 80
    nodePort: 30500 # Access on http://node-ip:30500
  ```

#### **Grafana Ingress Configuration**

```yaml
ingress:
  enabled: true
  ingressClassName: aws-alb
```

- **`ingress.enabled`**: Enable ingress routing (needed for AWS ALB)
- **`ingressClassName`**: Type of ingress controller
  - `aws-alb`: AWS Application Load Balancer (requires ALB controller)
  - `nginx`: Nginx ingress
  - Leave blank for default ingress class

```yaml
annotations:
  kubernetes.io/ingress.class: aws-alb
  alb.ingress.kubernetes.io/scheme: internet-facing
  alb.ingress.kubernetes.io/target-type: ip
  alb.ingress.kubernetes.io/listen-ports: '[{"HTTPS":443}]'
```

- **`ingress.annotations`**: Instructions for the ALB controller
  - `internet-facing`: ALB is publicly accessible
  - `target-type: ip`: Route traffic directly to pod IP
  - `listen-ports`: Listen on HTTPS port 443 (requires certificate)

```yaml
alb.ingress.kubernetes.io/certificate-arn: arn:aws:acm:us-east-1:account:certificate/id
```

- **`certificate-arn`** (uncomment if using HTTPS): ARN of your AWS Certificate Manager certificate
- **Find your cert ARN**:
  ```bash
  aws acm list-certificates --region us-east-1
  ```

```yaml
alb.ingress.kubernetes.io/group.name: apps
alb.ingress.kubernetes.io/ssl-redirect: "443"
```

- **`group.name`**: Group multiple ingresses into one ALB (saves cost)
- **`ssl-redirect`**: Redirect HTTP to HTTPS

```yaml
hosts:
  - domian.com
path: /
pathType: Prefix
```

- **`hosts`**: Domain names where Grafana is accessible
- **Change to your domain**:
  ```yaml
  hosts:
    - grafana.example.com
  ```
- **`path`**: URL path where Grafana is served (`/` means root)
- **`pathType: Prefix`**: Match all paths starting with `/`

```yaml
tls:
  - hosts: [domian.com]
```

- **`tls`**: TLS/SSL configuration for HTTPS
- **Must match the certificate domain**

#### **Grafana DataSources**

```yaml
datasources:
  datasources.yaml:
    apiVersion: 1
    datasources:
      - name: Prometheus
        type: prometheus
        url: http://observability-kube-prometh-prometheus.observability.svc.cluster.local:9090
        access: proxy
        isDefault: true
```

- **`datasources`**: Connect Grafana to data backends
- **`name: Prometheus`**: Display name in Grafana
- **`type: prometheus`**: Prometheus metric data source
- **`url`**: Internal Kubernetes DNS to reach Prometheus
- **`isDefault: true`**: Use Prometheus as default datasource
- **`access: proxy`**: Grafana queries Prometheus server-side (secure)

```yaml
- name: Loki
  type: loki
  url: http://observability-loki.observability.svc.cluster.local:3100
  access: proxy
  uid: loki
```

- **`type: loki`**: Log aggregation datasource
- **`uid: loki`**: Unique identifier for referencing in dashboards

```yaml
- name: Tempo
  type: tempo
  url: http://observability-tempo.observability.svc.cluster.local:3200
  access: proxy
  uid: tempo
  jsonData:
    httpMethod: GET
    serviceMap:
      datasourceUid: prometheus
    tracesToMetrics:
      datasourceUid: prometheus
    tracesToLogs:
      datasourceUid: "Loki"
      filterByTraceID: true
```

- **`type: tempo`**: Distributed tracing datasource
- **`serviceMap.datasourceUid: prometheus`**: Link traces to Prometheus metrics
- **`tracesToLogs.datasourceUid: Loki`**: Jump from traces to logs
- **`filterByTraceID: true`**: Automatically filter logs by trace ID

#### **Grafana Persistent Storage**

```yaml
persistence:
  enabled: true
  storageClassName: gp3
  accessModes:
    - ReadWriteOnce
  size: 5Gi
```

- **`persistence.enabled`**: Store Grafana dashboard data persistently
- **`storageClassName: gp3`**: AWS EBS volume type (fast, cost-effective)
  - `gp3`: General purpose SSD (recommended)
  - `gp2`: Older general purpose
  - `io1`: High IOPS (expensive)
- **`size: 5Gi`**: Volume size
  - Increase if you have many dashboards: `10Gi` or `20Gi`
- **Change storage class**:
  ```yaml
  storageClassName: my-custom-storage-class
  ```

#### **Grafana Resource Requests and Limits**

```yaml
resources:
  requests:
    cpu: 100m
    memory: 512Mi
  limits:
    cpu: 300m
    memory: 1Gi
```

- **`requests`**: Minimum resources needed (Kubernetes reserves these)
  - `cpu: 100m`: 0.1 CPU cores
  - `memory: 512Mi`: 512 megabytes
- **`limits`**: Maximum resources allowed (pod killed if exceeded)
  - `cpu: 300m`: 0.3 CPU cores max
  - `memory: 1Gi`: 1 gigabyte max
- **Adjust based on dashboard count**:
  ```yaml
  # For many dashboards
  resources:
    requests:
      cpu: 200m
      memory: 1Gi
    limits:
      cpu: 500m
      memory: 2Gi
  ```

---

### **Prometheus (kube-prometheus-stack) Configuration**

```yaml
kube-prometheus-stack:
  grafana:
    enabled: false
```

- **`grafana.enabled: false`**: Don't deploy Grafana from this chart (we use standalone above)

```yaml
prometheus:
  prometheusSpec:
    enableRemoteWriteReceiver: true
```

- **`enableRemoteWriteReceiver`**: Allow other Prometheus instances to push metrics here

```yaml
enableFeatures:
  - exemplar-storage
  - native-histograms
```

- **`exemplar-storage`**: Store exemplars (trace IDs) with metrics for trace correlation
- **`native-histograms`**: Use native histograms (more efficient than buckets)

```yaml
podMonitorSelector: {}
podMonitorNamespaceSelector: {}
serviceMonitorSelector: {}
serviceMonitorNamespaceSelector: {}
```

- **Empty selectors** (`{}`): Prometheus discovers ALL ServiceMonitors and PodMonitors
- **To select only specific monitors**:
  ```yaml
  serviceMonitorSelector:
    matchLabels:
      prometheus: mine
  ```
  Then add these labels to ServiceMonitor resources.

#### **Prometheus Data Retention**

```yaml
retention: 168h
retentionSize: 8GiB
```

- **`retention: 168h`**: Keep metrics for 7 days (7 \* 24 = 168 hours)
  - Change to `336h` for 14 days (uses more storage)
  - Change to `72h` for 3 days (saves storage)
- **`retentionSize: 8GiB`**: Hard limit - delete oldest metrics if disk > 8GB
  - Prevents Prometheus from filling disk
  - Adjust based on your cluster size

#### **Prometheus Scrape Configuration**

```yaml
scrapeInterval: 30s
evaluationInterval: 30s
```

- **`scrapeInterval: 30s`**: Prometheus scrapes metrics every 30 seconds
  - Smaller = more detailed data (costs more storage)
  - Larger = less frequent (miss fast changes)
  - 30s is good balance for most clusters
- **`evaluationInterval`**: How often alerts are evaluated

#### **Prometheus Resource Requests**

```yaml
resources:
  requests:
    cpu: 300m
    memory: 1Gi
  limits:
    cpu: 800m
    memory: 1.5Gi
```

- **Prometheus needs resources**: It continuously scrapes and stores metrics
- **Increase for large clusters**:
  ```yaml
  resources:
    requests:
      cpu: 500m
      memory: 2Gi
    limits:
      cpu: 1000m
      memory: 3Gi
  ```

#### **Prometheus Remote Write**

```yaml
remoteWrite:
  - url: "http://observability-alloy.observability.svc.cluster.local:9080/api/v1/prom/receive"
```

- **`remoteWrite`**: Forward metrics to external system (Alloy)
- **URL**: Alloy receiver endpoint
- **Use case**:
  - Central metrics aggregation
  - Long-term storage outside Prometheus
  - Send metrics to cloud provider

---

### **LOKI Configuration**

```yaml
loki:
  loki:
    commonConfig:
      replication_factor: 1
```

- **`replication_factor: 1`**: Store each log only once (good for single node)
  - Use `3` for HA (high availability) with multiple Loki instances

```yaml
storage:
  type: "filesystem"
  bucketNames:
    chunks: chunks
    ruler: ruler
    admin: admin
```

- **`type: filesystem`**: Store logs on local filesystem (simple, single instance)
- **For production**, use object storage:
  ```yaml
  storage:
    type: "s3"
    s3:
      endpoint: s3.amazonaws.com
      bucket: my-loki-bucket
      region: us-east-1
  ```

```yaml
schemaConfig:
  configs:
    - from: "2024-04-01"
      store: tsdb
      object_store: filesystem
      schema: v13
      index:
        prefix: loki_index_
        period: 24h
```

- **`schema: v13`**: Log storage format (latest)
- **`index.period: 24h`**: Create new index every 24 hours

#### **Loki Log Retention**

```yaml
limits_config:
  retention_period: 168h # Global: 7 days
  retention_stream:
    - selector: '{namespace=~".+"}'
      period: 336h # 14 days
    - selector: '{namespace=~"(kube-system|observability)"}'
      period: 72h # 3 days
```

- **`retention_period: 168h`**: Keep all logs for 7 days by default
- **`retention_stream`**: Different retention per label
  - User namespaces: 14 days
  - System namespaces: 3 days
- **Change global retention**:
  ```yaml
  retention_period: 336h # 14 days
  ```

```yaml
limits_config:
  allow_structured_metadata: true
  volume_enabled: true
```

- **`allow_structured_metadata`**: Allow custom log metadata
- **`volume_enabled`**: Track log volume metrics

#### **Loki SingleBinary Deployment**

```yaml
singleBinary:
  replicas: 1
```

- **`replicas: 1`**: Single Loki instance (good for dev/small clusters)
- **For HA**: Use `distributor`, `ingester`, `querier` mode with `replicas: 3`

```yaml
persistence:
  enabled: true
  storageClass: gp3
  size: 20Gi
```

- **`size: 20Gi`**: Disk size for storing logs
  - **Increase for large clusters**: `50Gi`, `100Gi`
  - **Formula**: `(daily_log_volume_GB) × (retention_days) × 1.2 (overhead)`

#### **Loki Resources**

```yaml
resources:
  requests:
    cpu: "1"
    memory: "2Gi"
  limits:
    cpu: "2"
    memory: "4Gi"
```

- **Loki needs more memory**: Stores all logs in memory before flushing
- **Increase for high log volume**:
  ```yaml
  resources:
    requests:
      cpu: "2"
      memory: "4Gi"
    limits:
      cpu: "4"
      memory: "8Gi"
  ```

#### **Loki Compactor**

```yaml
compactor:
  enabled: true
  working_directory: /data/loki/compactor
  shared_store: filesystem
  compaction_interval: 10m
  retention_enabled: true
  retention_delete_delay: 2h
```

- **`compactor.enabled: true`**: Clean up old log chunks
- **`compaction_interval: 10m`**: Run cleanup every 10 minutes
- **`retention_enabled: true`**: Actually delete old logs (not just mark as deleted)

---

### **TEMPO Configuration**

```yaml
tempo:
  tempo:
    metricsGenerator:
      enabled: true
      remoteWriteUrl: "http://observability-kube-prometh-prometheus.observability.svc.cluster.local:9090/api/v1/write"
```

- **`metricsGenerator.enabled: true`**: Compute metrics from traces (RED metrics)
- **`remoteWriteUrl`**: Send computed metrics to Prometheus
- **Benefit**: Get request rate, error rate, duration automatically from traces

```yaml
overrides:
  defaults:
    metrics_generator:
      processors: [service-graphs, span-metrics, local-blocks]
```

- **`service-graphs`**: Build service dependency map
- **`span-metrics`**: Generate latency metrics from spans
- **`local-blocks`**: Store trace blocks locally

#### **Tempo Storage**

```yaml
persistence:
  enabled: true
  storageClass: gp3
  size: 10Gi
```

- **`size: 10Gi`**: Disk for storing traces
- **Increase for more traces**:
  ```yaml
  size: 50Gi # For high-volume tracing
  ```

#### **Tempo Resources**

```yaml
resources:
  requests:
    cpu: 200m
    memory: 512Mi
  limits:
    cpu: 400m
    memory: 1Gi
```

- **Lightweight** compared to Prometheus/Loki
- **Increase only if tracing high volume**

---

### **ALLOY Configuration**

```yaml
alloy:
  alloy:
    configMap:
      content: |-
        logging {
          level = "info"
          format = "logfmt"
        }
```

- **`logging.level`**: How verbose Alloy logs are
  - `info`: Normal (recommended)
  - `debug`: Very verbose (troubleshooting)

```yaml
discovery.kubernetes "pods" {
role = "pod"
}
```

- **Auto-discover all pods** in the cluster for log collection

```yaml
discovery.relabel "pods" {
targets = discovery.kubernetes.pods.targets
rule {
source_labels = ["__meta_kubernetes_namespace"]
target_label = "namespace"
action = "replace"
}
```

- **Extract labels from Kubernetes metadata**
- **Creates labels**: namespace, app, container, pod

```yaml
loki.source.kubernetes "pods" {
targets    = discovery.relabel.pods.output
forward_to = [loki.process.process.receiver]
}
```

- **Collect logs from all discovered pods**
- **Send to processing pipeline**

```yaml
loki.process "process" {
stage.drop {
older_than = "1h"
drop_counter_reason = "too old"
}
```

- **Drop logs older than 1 hour** (prevent stale data)

```yaml
otelcol.receiver.otlp "otlp_receiver" {
grpc {
endpoint = "0.0.0.0:4317"
}
http {
endpoint = "0.0.0.0:4318"
}
```

- **Listen for traces on 2 protocols**:
  - `grpc` (port 4317): Fast, binary
  - `http` (port 4318): REST-based, JSON
- **Both send to Tempo**

#### **Alloy Resource Limits**

```yaml
resources:
  limits:
    cpu: 200m
    memory: 128Mi
  requests:
    cpu: 100m
    memory: 128Mi
```

- **Very lightweight**: Minimal overhead
- **Runs as DaemonSet on all nodes**, so total resources = `requests × node_count`

#### **Alloy Node Scheduling**

```yaml
nodeSelector: {}
tolerations:
  - operator: Exists
```

- **`tolerations: [operator: Exists]`**: Run on ALL nodes including tainted ones
- **This ensures logs are collected from every node**

---

### **Storage Class Configuration**

Throughout values.yaml, you see `storageClassName: gp3`. This is AWS EBS volume type.

**Storage class options:**

```yaml
# AWS EBS
storageClassName: gp3          # General purpose SSD (recommended, cheapest)
storageClassName: gp2          # Older general purpose
storageClassName: io1          # High IOPS (expensive)
storageClassName: st1          # Throughput optimized

# Other clouds
storageClassName: standard     # Generic
storageClassName: fast         # Custom fast storage
storageClassName: slow         # Custom slow storage
```

**Check available storage classes:**

```bash
kubectl get storageclass
```

**Example changing storage class:**

```yaml
grafana:
  persistence:
    storageClassName: io1 # Use high IOPS storage

loki:
  singleBinary:
    persistence:
      storageClassName: st1 # Use throughput optimized
```

---

### **Node Scheduling (Affinity & Taints)**

Uncomment these to schedule on specific nodes:

```yaml
grafana:
  nodeSelector:
    intent: observability
  tolerations:
    - key: "observability"
      operator: "Equal"
      value: "true"
      effect: "NoSchedule"
```

**This means:**

- `nodeSelector`: Only run on nodes with label `intent=observability`
- `tolerations`: Tolerate the taint `observability=true:NoSchedule`

**Label your nodes:**

```bash
kubectl label nodes <node-name> intent=observability
```

**Taint your nodes:**

```bash
kubectl taint nodes <node-name> observability=true:NoSchedule
```

---

## Summary of Common Customizations

### **For Small Cluster (< 5 nodes)**

```yaml
grafana:
  resources:
    requests:
      cpu: 100m
      memory: 512Mi

prometheus:
  prometheusSpec:
    retention: 72h # Shorter retention
    resources:
      requests:
        cpu: 200m

loki:
  singleBinary:
    persistence:
      size: 10Gi # Smaller PV
```

### **For Production (Large Cluster)**

```yaml
grafana:
  adminPassword: "secure-password"
  persistence:
    size: 20Gi
  resources:
    limits:
      cpu: 500m
      memory: 2Gi

prometheus:
  prometheusSpec:
    retention: 336h # 14 days
    resources:
      limits:
        cpu: 1000m
        memory: 3Gi

loki:
  singleBinary:
    persistence:
      size: 100Gi
    resources:
      limits:
        cpu: 2
        memory: 8Gi

tempo:
  persistence:
    size: 50Gi
```

### **For Multi-Cloud Setup**

```yaml
grafana:
  datasources:
    datasources.yaml:
      datasources:
        # Add multiple Prometheus instances
        - name: Prometheus-AWS
          type: prometheus
          url: http://prometheus-aws.remote:9090

        - name: Prometheus-GCP
          type: prometheus
          url: http://prometheus-gcp.remote:9090
```

---

## Trace Generator for Testing

A sample trace generator pod is included in `trace-generator.yaml`. This is useful for testing the tracing pipeline:

```bash
kubectl apply -f trace-generator.yaml

# View generated traces
kubectl logs -n monitoring k6-trace-generator -f
```

The trace generator sends traces to Tempo. View them in Grafana under Tempo datasource.

---

## Troubleshooting

### **Pods not starting:**

```bash
# Check pod status
kubectl describe pod -n observability <pod-name>

# View logs
kubectl logs -n observability <pod-name>

# Check resource requests/limits
kubectl get resourcequota -n observability
```

### **No data in Prometheus:**

```bash
# Check if scrape targets are healthy
kubectl port-forward -n observability svc/observability-kube-prometh-prometheus 9090:9090

# Visit http://localhost:9090/targets
# All targets should show "UP"
```

### **Loki running out of disk:**

```bash
# Check disk usage
kubectl exec -it -n observability observability-loki-0 -- df -h /tmp/loki

# Reduce retention or add more storage
kubectl edit values observability -n observability
```

### **Grafana not accessible:**

```bash
# Check ingress status
kubectl get ingress -n observability

# Check ALB is running
kubectl get pods -n kube-system | grep aws-load-balancer
```

### **Traces not appearing:**

```bash
# Check if application is actually sending traces
# Verify OTLP endpoint is reachable
kubectl port-forward -n observability svc/observability-alloy 4317:4317

# Check Alloy logs
kubectl logs -n observability -l app=alloy
```

---

## Development & Testing

### **Lint the chart:**

```bash
helm lint .
```

### **Render templates locally:**

```bash
helm template observability .

# With custom values
helm template observability . -f custom-values.yaml
```

### **Package for distribution:**

```bash
helm package .
```

This creates `observability-1.0.0.tgz` that can be deployed anywhere.

---

## Contributing

Contributions are welcome! Please follow these guidelines:

1. Fork and clone the repository
2. Create a feature branch: `git checkout -b feature/my-improvement`
3. Make changes and test locally: `helm template observability .`
4. Lint the chart: `helm lint .`
5. Commit with descriptive messages: `git commit -am "Add feature X"`
6. Push and create a Pull Request

For reporting issues, please include:

- Kubernetes version
- Helm version
- Output of `helm template`
- Error messages and pod logs

---

## License

This project is open source and available under the [MIT License](LICENSE).

A copy of the full license text is also included in the `LICENSE` file.

---

### MIT License

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
