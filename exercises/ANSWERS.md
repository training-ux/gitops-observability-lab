# Checkpoint answers (spoilers)

## Helm

1. A **chart** is the package/template; a **release** is an installed instance of that chart.
2. `helm template`
3. `helm rollback <release> <revision>`

## Argo CD

1. It keeps the cluster matched to the desired state stored in Git (reduces config drift / snowflake clusters).
2. Argo CD automatically corrects manual cluster changes back to Git.
3. In Git (repo path or Helm chart source).

## Prometheus

1. Pull (scrape) by default.
2. Per-second average increase of the counter over 5 minutes.
3. Declares which Services Prometheus should scrape when using the operator.

## Grafana

1. A backend Grafana queries (e.g. Prometheus).
2. Time series, stat, table, bar chart, gauge (any two).
3. Faster setup and community best practices.

## PagerDuty

1. Secret key that authorizes Alertmanager (or other tools) to create incidents on a service.
2. Alertmanager.
3. It creates constant noise; great for testing, bad for real on-call.
