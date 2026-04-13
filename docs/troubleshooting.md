# Troubleshooting

## Quick Troubleshooting Guide

| Symptom | Likely Cause | Fix |
|--------|-------------|-----|
| Helm fails with `localhost:8080` | kubeconfig not set | `export KUBECONFIG=/etc/rancher/k3s/k3s.yaml` |
| k9s shows no cluster | kubeconfig not set | same as above |
| Pods stuck in Pending | resource constraints or image pull issues | `kubectl describe pod` |
| Grafana not loading | service not exposed | use port-forward |
| No metrics in Grafana | Prometheus not scraping targets | check ServiceMonitor / endpoints |
| node_exporter not showing | service not running or wrong IP | verify service + curl |
| Blackbox metrics missing | Probe not configured or exporter issue | test `/probe` endpoint |

---

## Helm: Cluster Unreachable

Error:

Kubernetes cluster unreachable: localhost:8080

Fix:

export KUBECONFIG=/etc/rancher/k3s/k3s.yaml

---

## K3s Not Running

Check service:

systemctl status k3s --no-pager

Restart:

systemctl restart k3s

---

## Pods Not Starting

Check pods:

k3s kubectl get pods -n monitoring

Describe pod:

k3s kubectl describe pod <pod-name> -n monitoring

Logs:

k3s kubectl logs <pod-name> -n monitoring

---

## Grafana Not Accessible

Port-forward:

k3s kubectl port-forward svc/monitoring-grafana -n monitoring 3000:80

OR check service:

k3s kubectl get svc -n monitoring

---

## Prometheus Not Scraping node_exporter

Check node_exporter:

systemctl status prometheus-node-exporter

Test endpoint:

curl http://localhost:9100/metrics

Check Kubernetes objects:

k3s kubectl get servicemonitor -n monitoring
k3s kubectl get endpoints -n monitoring

---

## No Metrics in Grafana

Check Prometheus:

k3s kubectl port-forward svc/monitoring-kube-prometheus-prometheus -n monitoring 9090:9090

Then open browser and test query:

up

---

## k9s Not Working

Fix kubeconfig:

export KUBECONFIG=/etc/rancher/k3s/k3s.yaml

Run:

k9s

---

## Blackbox Exporter Not Working

Check pod:

k3s kubectl get pods -n monitoring | grep blackbox

Check logs:

k3s kubectl logs -n monitoring deploy/blackbox-prometheus-blackbox-exporter

Test manually:

k3s kubectl port-forward -n monitoring svc/blackbox-prometheus-blackbox-exporter 9115:9115

Then:

curl "http://127.0.0.1:9115/probe?target=8.8.8.8&module=icmp"

---

## General Debug Commands

k3s kubectl get all -n monitoring
k3s kubectl get nodes -o wide
k3s kubectl get events -n monitoring
