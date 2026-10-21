# Module 03 — Prometheus for beginners

Prometheus scrapes metrics from targets and stores them as time series. You query them with **PromQL**.

## Concepts (60 seconds)

| Term | Meaning |
| --- | --- |
| Scrape | Prometheus pulls metrics from an HTTP endpoint |
| Target | Something that exposes `/metrics` |
| Metric | Named measurement (`http_requests_total`) |
| PromQL | Query language for metrics |
| ServiceMonitor | CRD that tells Prometheus what to scrape (with operator) |

## Exercise 1 — Install kube-prometheus-stack

This installs Prometheus + Grafana + Alertmanager together.

```bash
kubectl create namespace monitoring

helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update

helm upgrade --install kps prometheus-community/kube-prometheus-stack \
  -n monitoring \
  --set grafana.adminPassword='LabPass123!' \
  --set prometheus.service.type=LoadBalancer \
  --set grafana.service.type=LoadBalancer \
  --wait
```

Check pods:

```bash
kubectl get pods -n monitoring
kubectl get svc -n monitoring
```

## Exercise 2 — Open Prometheus UI

```bash
# Port-forward if LoadBalancer is slow/unavailable
kubectl -n monitoring port-forward svc/kps-kube-prometheus-stack-prometheus 9090:9090
```

Open http://localhost:9090

Try these queries in the UI:

```promql
up
```

```promql
count(up)
```

```promql
sum(rate(container_cpu_usage_seconds_total[5m]))
```

**Expected:** `up` returns `1` for healthy scrape targets.

## Exercise 3 — Explore targets and metrics

In Prometheus UI:

1. Go to **Status → Targets**
2. Confirm endpoints are `UP`
3. Go to **Graph**, search metric names like `node_memory` or `kube_pod`

From CLI (quick check):

```bash
kubectl -n monitoring get prometheus
kubectl -n monitoring get servicemonitors
```

## Exercise 4 — Deploy an app that exposes metrics

```bash
kubectl create namespace demo-metrics

cat <<'EOF' | kubectl apply -f -
apiVersion: apps/v1
kind: Deployment
metadata:
  name: metrics-demo
  namespace: demo-metrics
spec:
  replicas: 1
  selector:
    matchLabels:
      app: metrics-demo
  template:
    metadata:
      labels:
        app: metrics-demo
      annotations:
        prometheus.io/scrape: "true"
        prometheus.io/port: "8080"
        prometheus.io/path: "/metrics"
    spec:
      containers:
        - name: app
          image: quay.io/brancz/prometheus-example-app:v0.5.0
          ports:
            - containerPort: 8080
---
apiVersion: v1
kind: Service
metadata:
  name: metrics-demo
  namespace: demo-metrics
  labels:
    app: metrics-demo
spec:
  selector:
    app: metrics-demo
  ports:
    - name: http
      port: 8080
      targetPort: 8080
EOF
```

Create a ServiceMonitor (works with kube-prometheus-stack):

```bash
cat <<'EOF' | kubectl apply -f -
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: metrics-demo
  namespace: demo-metrics
  labels:
    release: kps
spec:
  selector:
    matchLabels:
      app: metrics-demo
  endpoints:
    - port: http
      path: /metrics
      interval: 15s
EOF
```

Wait ~1 minute, then in Prometheus search:

```promql
http_requests_total
```

## Exercise 5 — Beginner PromQL drills

```promql
# Instant value
http_requests_total

# Per-second rate over 5 minutes
rate(http_requests_total[5m])

# Sum across all instances
sum(rate(http_requests_total[5m]))
```

Generate traffic:

```bash
kubectl -n demo-metrics port-forward svc/metrics-demo 8080:8080
# In another terminal:
curl -s http://localhost:8080/ | head
for i in $(seq 1 20); do curl -s http://localhost:8080/ >/dev/null; done
```

## Module cleanup

```bash
kubectl delete ns demo-metrics --ignore-not-found
# Optional full remove:
# helm uninstall kps -n monitoring
# kubectl delete ns monitoring
```

## Checkpoint quiz

1. Does Prometheus push or pull metrics by default?
2. What does `rate(metric[5m])` mean?
3. Why use a ServiceMonitor?
