# Architecture

## Overview

This homelab environment is built on a single-node Kubernetes cluster using K3s, running on an Ubuntu LTS mini PC. The system is designed to provide lightweight observability for host performance and internet connectivity using a Prometheus-based monitoring stack.

---

## High-Level Architecture
            ┌──────────────────────┐
            │     Internet         │
            └─────────┬────────────┘
                      │
            ┌─────────▼────────────┐
            │  Xfinity Gateway     │
            │   (10.0.0.1)         │
            └─────────┬────────────┘
                      │
            ┌─────────▼────────────┐
            │  Apartment Switch    │
            └─────────┬────────────┘
                      │
            ┌─────────▼────────────┐
            │   Mini PC (Ubuntu)   │
            │   K3s + Monitoring   │
            └─────────┬────────────┘
                      │
            ┌─────────▼────────────┐
            │  TP-Link Router      │
            │  (192.168.1.1 NAT)   │
            └─────────┬────────────┘
                      │
                  WiFi Clients

                  
---

## Kubernetes Architecture

### Cluster Type
- Single-node K3s cluster
- Control plane and worker run on the same host

### Components

#### K3s
- Lightweight Kubernetes distribution
- Uses containerd as runtime
- Simplified control plane

#### Prometheus Stack (kube-prometheus-stack)
- Prometheus (metrics collection)
- Grafana (visualization)
- Alertmanager (alerting - not yet configured)
- Prometheus Operator (CRD management)

#### node_exporter
- Runs on the host OS (outside Kubernetes)
- Exposes system metrics via HTTP (port 9100)

#### k9s
- CLI-based Kubernetes management tool
- Used for cluster visibility and debugging

---


### Flow Description

1. `node_exporter` exposes host metrics on port 9100
2. Prometheus scrapes metrics using ServiceMonitor + Endpoints
3. Metrics are stored in Prometheus TSDB
4. Grafana queries Prometheus for visualization

---

## Network Segmentation

### Primary Network (10.0.0.0/24)
- Managed by Xfinity gateway
- Provides wired connectivity
- Mini PC resides here

### Secondary Network (192.168.1.0/24)
- Managed by TP-Link router
- Provides WiFi access
- Isolated behind NAT

---

## Observability Scope

### Available
- Host system metrics (CPU, memory, disk, network)
- Kubernetes cluster health
- Pod and container metrics
- Internet reachability (planned via blackbox exporter)

### Not Available
- SNMP-based router metrics (ISP restriction)
- WiFi client visibility (NAT isolation)
- Layer 2 switch metrics (unmanaged hardware)

---

## Design Decisions

### Why K3s?
- Lightweight and fast to deploy
- Ideal for single-node environments
- Minimal resource overhead

### Why kube-prometheus-stack?
- Pre-packaged monitoring solution
- Includes Prometheus Operator
- Built-in dashboards and exporters

### Why node_exporter on host?
- Provides accurate host-level metrics
- Simpler than container-based alternatives

---

## Limitations

- No high availability (single-node cluster)
- Limited network telemetry due to consumer hardware
- No centralized logging (future improvement)

---

## Future Architecture Enhancements

- Add blackbox exporter for uptime/latency monitoring
- Implement alerting via Alertmanager
- Introduce centralized logging (Loki/ELK)
- Replace network hardware for deeper observability
- Expand to multi-node Kubernetes cluster
