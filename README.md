# Kubernetes Observability Stack (Production-Style)

A production-ready Kubernetes observability stack deployed on a VPS using **k3s** and **Helm**.

This project demonstrates how to monitor a real Kubernetes cluster with
industry-standard tools and best practices, focusing on **operational readiness**
rather than local demos.

---

## 🔍 What’s Included

- **Prometheus**
  - Metric collection and storage
  - Custom scrape and retention configuration
- **Grafana**
  - NodePort-based secure access
  - Persistent storage
  - Production-grade dashboards
- **kube-state-metrics**
  - Kubernetes object and state visibility
- **node-exporter**
  - Node-level CPU, memory, disk, and network metrics
- **cAdvisor (via kubelet)**
  - Container-level resource metrics
- **Alertmanager**
  - Alert lifecycle management
- **Custom Alert Rules**
  - Node disk pressure
  - High CPU usage
  - Pod crash looping

---

## 🧠 Why This Project?

Most tutorials stop after “Grafana opens in the browser”.

This project goes further:
- Persistent storage (PVC-backed Prometheus & Grafana)
- Secure access patterns (no public Prometheus)
- Helm-based lifecycle management
- Alerting that reflects real production scenarios

It is designed to reflect how monitoring is actually deployed and operated in
real-world DevOps teams.

---

## 🏗 Architecture Overview

```text
Kubernetes (k3s on VPS)

kubelet (/metrics, /metrics/cadvisor)
kube-state-metrics
node-exporter
        ↓
   Prometheus
        ↓
 Alertmanager
        ↓
     Grafana
```
## ⚙️ Prerequisites

VPS (2 vCPU / 4+ GB RAM recommended)

k3s installed

Helm v3

##  Installation

```
kubectl create namespace monitoring --dry-run=client -o yaml | kubectl apply -f -

helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update

helm upgrade --install obs prometheus-community/kube-prometheus-stack \
  -n monitoring \
  -f charts/values-kube-prometheus-stack.yaml
```
## Access

- Grafana (NodePort)
  
```
http://<VPS_IP>:32000
```

- Prometheus & Alertmanager

Not publicly exposed

Accessed via kubectl port-forward when needed

## 📊 Dashboards

The following production-grade dashboards are imported and validated:

- Node Exporter Full

- Kubernetes / Compute Resources / Cluster

- Kubernetes / Pods

Dashboard JSON files are exported and version-controlled under dashboards/.

<img width="1908" height="904" alt="image" src="https://github.com/user-attachments/assets/7a517b60-428c-41ee-89b1-26569b70d860" />


## 🚨 Alerting

Custom Prometheus rules are defined under alerts/:

- NodeDiskAlmostFull

- HighCPU

- PodCrashLooping

Alerts are evaluated continuously and handled by Alertmanager.
## 🔐 Security Considerations

- Grafana admin credentials stored in Kubernetes Secrets

- Prometheus and Alertmanager are not exposed publicly

- NodePort is used only for Grafana in a single-node VPS scenario

## Validation

```
rate(container_cpu_usage_seconds_total[2m])
container_memory_working_set_bytes
```

# ✅ Updates: Production Alerts + Telegram Notifications

This project includes a production-oriented alerting setup and Telegram notifications:

## Alert Rules (PrometheusRule)
Defined under `alerts/prod-rules.yaml`:
- **PrometheusDown** (critical)
- **NodeNotReady** (critical)
- **NodeDiskAlmostFull** (critical)
- **PodCrashLooping** (critical)
- **HighCPU** (warning)
- **LowMemory** (warning)
- **NodeExporterDown** (warning)

## Telegram Notifications (Alertmanager → Webhook → Telegram)
Alert notifications are delivered to Telegram using:
- A lightweight webhook receiver deployed in-cluster (`telegram-webhook-config.yaml`)
- Alertmanager routing to the webhook (`alertmanager-telegram.yaml`)
- Credentials stored as Kubernetes Secrets (token/chat_id are **not** committed to Git)

> Note: In production, only actionable alerts should be routed to chat channels to avoid alert fatigue.

