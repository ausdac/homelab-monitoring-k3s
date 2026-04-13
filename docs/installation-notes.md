# Installation Notes & Troubleshooting

## K3s Installation

Install K3s:

curl -sfL https://get.k3s.io | sh -

Verify cluster:

k3s kubectl get nodes

---

## kubeconfig Setup (IMPORTANT)

Fix Helm / kubectl access:

export KUBECONFIG=/etc/rancher/k3s/k3s.yaml

(Optional: persist)

echo 'export KUBECONFIG=/etc/rancher/k3s/k3s.yaml' >> ~/.bashrc
source ~/.bashrc

---

## Install Helm

curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash

Verify:

helm version

---

## Deploy Monitoring Stack

Add repo:

helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update

Create namespace:

k3s kubectl create namespace monitoring

Install:

helm install monitoring prometheus-community/kube-prometheus-stack -n monitoring

Verify pods:

k3s kubectl get pods -n monitoring

---

## Grafana Access

Get password:

k3s kubectl get secret monitoring-grafana -n monitoring -o jsonpath="{.data.admin-password}" | base64 -d && echo

Port-forward:

k3s kubectl port-forward svc/monitoring-grafana -n monitoring 3000:80

Access:

http://<mini-pc-ip>:3000

---

## node_exporter Installation

Install:

apt install prometheus-node-exporter -y

Start service:

systemctl enable prometheus-node-exporter
systemctl start prometheus-node-exporter

Verify:

curl http://localhost:9100/metrics

---

## Quick Validation Commands

Cluster:

k3s kubectl get nodes

Monitoring:

k3s kubectl get pods -n monitoring
k3s kubectl get svc -n monitoring

---

# Troubleshooting

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

## General Debug Commands

k3s kubectl get all -n monitoring
k3s kubectl get nodes -o wide
k3s kubectl get events -n monitoring
