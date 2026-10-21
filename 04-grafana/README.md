# Module 04 — Grafana for beginners

Grafana turns Prometheus metrics into dashboards people can actually read.

> Tip: If you installed `kube-prometheus-stack` in Module 03, Grafana is already running in `monitoring`.

## Concepts (60 seconds)

| Term | Meaning |
| --- | --- |
| Data source | Where Grafana reads data (Prometheus, Loki, etc.) |
| Dashboard | Collection of panels |
| Panel | One chart/stat/table |
| Variable | Dropdown filter (namespace, pod, etc.) |

## Exercise 1 — Access Grafana

```bash
# Find Grafana service
kubectl -n monitoring get svc | grep grafana

# Port-forward
kubectl -n monitoring port-forward svc/kps-grafana 3000:80
```

Open http://localhost:3000

Default login (from Module 03 install):

- Username: `admin`
- Password: `LabPass123!`

If you forgot the password:

```bash
kubectl -n monitoring get secret kps-grafana \
  -o jsonpath='{.data.admin-password}' | base64 -d
echo
```

## Exercise 2 — Confirm Prometheus data source

In Grafana UI:

1. Go to **Connections → Data sources**
2. Open **Prometheus**
3. Click **Save & test**

**Expected:** “Successfully queried the Prometheus API.”

## Exercise 3 — Explore built-in dashboards

1. Go to **Dashboards**
2. Open folders like **Kubernetes / Compute Resources**
3. Pick **Cluster** or **Namespace (Pods)**

Explore:

- CPU usage panels
- Memory panels
- Time range selector (top-right)

## Exercise 4 — Create your first dashboard

1. **Dashboards → New → New dashboard → Add visualization**
2. Select Prometheus data source
3. Enter query:

```promql
sum(rate(container_cpu_usage_seconds_total{namespace!=""}[5m])) by (namespace)
```

4. Set visualization to **Time series**
5. Title: `CPU by namespace`
6. **Save dashboard** as `Lab - Beginner CPU`

Add a second panel:

```promql
sum(kube_pod_info) by (namespace)
```

Visualization: **Bar chart** or **Stat**  
Title: `Pod count by namespace`

## Exercise 5 — Import a community dashboard

1. Go to **Dashboards → New → Import**
2. Enter dashboard ID: `315` (Kubernetes cluster monitoring — classic starter)
3. Choose Prometheus data source
4. Import

**Expected:** A full cluster dashboard appears without building panels manually.

## Exercise 6 — Share / export

1. Open your dashboard
2. Click share/export (or **Dashboard settings → JSON Model**)
3. Copy JSON for backup

Export via API (optional):

```bash
# After port-forward is running
curl -s -u admin:LabPass123! http://localhost:3000/api/search | head
```

## Module cleanup

Keep Grafana if continuing to PagerDuty/alerting labs.

To remove later with Prometheus stack:

```bash
helm uninstall kps -n monitoring
kubectl delete ns monitoring
```

## Checkpoint quiz

1. What is a Grafana data source?
2. Name two panel types.
3. Why import dashboards instead of building everything from scratch?
