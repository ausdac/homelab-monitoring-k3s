# Homelab Monitoring on K3s

## Overview

This project documents the deployment of a lightweight monitoring stack running on a single-node Kubernetes cluster using K3s. The environment is hosted on an Ubuntu LTS mini PC and is designed to monitor both system health and basic internet connectivity within a home network.

The goal of this project is to build a practical, production-style observability stack in a constrained homelab environment while gaining hands-on experience with Kubernetes, Prometheus, and Grafana.

---

## Architecture

### Infrastructure

* **Host:** Ubuntu LTS mini PC
* **Kubernetes:** K3s (single-node cluster)
* **Package Manager:** Helm

### Monitoring Stack

* Prometheus (metrics collection)
* Grafana (visualization)
* Alertmanager (included, not yet configured)
* Prometheus Operator (manages monitoring resources)

### Supporting Tools

* k9s (terminal-based Kubernetes UI)
* node_exporter (host-level metrics)

---

## Network Layout

The home network consists of two segments:

* **Primary Network (10.0.0.0/24)**

  * Managed by an Xfinity/Comcast gateway
  * Provides wired connectivity throughout the apartment
  * The mini PC is connected here via Ethernet

* **Secondary Network (192.168.1.0/24)**

  * Managed by a TP-Link router
  * Provides Wi-Fi access
  * Runs behind NAT and is isolated from the mini PC

### Key Implications

* The mini PC can directly monitor its own host and internet connectivity
* SNMP-based monitoring is limited due to ISP equipment restrictions
* Wi-Fi devices are not directly observable from the monitoring host

---

## Deployment Steps

### 1. Update System

```bash
apt update && apt upgrade -y
```

### 2. Install K3s

```bash
curl -sfL https://get.k3s.io | sh -
k3s kubectl get nodes
```

### 3. Configure kubeconfig

```bash
export KUBECONFIG=/etc/rancher/k3s/k3s.yaml
```

### 4. Install Helm

```bash
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```

### 5. Deploy Monitoring Stack

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
k3s kubectl create namespace monitoring
helm install monitoring prometheus-community/kube-prometheus-stack -n monitoring
```

---

## Accessing Grafana

### Retrieve Admin Password

```bash
k3s kubectl get secret monitoring-grafana -n monitoring -o jsonpath="{.data.admin-password}" | base64 -d && echo
```

### Access UI

Expose Grafana via NodePort or port-forward:

```bash
k3s kubectl port-forward svc/monitoring-grafana -n monitoring 3000:80
```

Then open:

```
http://<mini-pc-ip>:3000
```

---

## Host Monitoring (node_exporter)

### Install on Host

```bash
apt install prometheus-node-exporter -y
systemctl enable prometheus-node-exporter
systemctl start prometheus-node-exporter
```

### Verify Metrics

```bash
curl http://localhost:9100/metrics
```

### Kubernetes Integration

A ServiceMonitor and Endpoints resource are used to allow Prometheus to scrape the host from within the cluster.

---

## Observability Capabilities

### Currently Implemented

* Kubernetes cluster health
* Pod and container metrics
* Host CPU, memory, and network usage
* Prometheus metrics collection
* Grafana visualization

### Planned / Deferred

* Blackbox exporter (internet uptime and latency)
* SNMP exporter (limited by hardware)
* Custom Grafana dashboards
* Alerting rules

---

## Challenges Encountered

### Helm Could Not Reach Cluster

**Issue:**

```
Kubernetes cluster unreachable: localhost:8080
```

**Cause:**
Helm was not using the K3s kubeconfig

**Fix:**

```bash
export KUBECONFIG=/etc/rancher/k3s/k3s.yaml
```

---

### Network Monitoring Limitations

* ISP gateway does not expose SNMP
* Internal network segmentation prevents visibility into downstream devices

**Resolution:**
Focus shifted to:

* host-level monitoring
* internet reachability (future blackbox exporter)

---

## Repository Structure

```text
homelab-monitoring-k3s/
├── README.md
├── docs/
│   ├── architecture.md
│   ├── installation-notes.md
│   └── troubleshooting.md
├── manifests/
│   ├── node-exporter.yaml
│   ├── blackbox-probes.yaml
│   └── grafana-service.yaml
├── screenshots/
└── .gitignore
```

---

## Future Improvements

* Implement blackbox exporter for latency and uptime monitoring
* Add alerting (email, Slack, etc.)
* Build custom Grafana dashboards
* Introduce centralized logging (Loki or ELK)
* Replace network hardware for better observability (optional)

---

## Summary

This project demonstrates how a full monitoring stack can be deployed on a single-node Kubernetes cluster using K3s. While limited by consumer-grade networking hardware, the system provides meaningful insight into host performance and serves as a strong foundation for learning observability, Kubernetes operations, and infrastructure monitoring.

The environment is intentionally simple but extensible, making it ideal for continued experimentation and incremental improvements.
