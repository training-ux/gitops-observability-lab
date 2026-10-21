# Module 05 — PagerDuty for beginners

PagerDuty routes alerts to people on-call (SMS, phone, app). In this lab you connect **Alertmanager → PagerDuty**.

## Concepts (60 seconds)

| Term | Meaning |
| --- | --- |
| Service | App/system you monitor in PagerDuty |
| Integration key | Secret that Alertmanager uses to open incidents |
| Escalation policy | Who gets notified, and in what order |
| Incident | An alert that needs a human response |

## Before you start

1. Create a free PagerDuty account: https://www.pagerduty.com/
2. Create a **Service** (e.g. `gitops-lab`)
3. Add integration type **Events API v2**
4. Copy the **Integration Key** (also called routing key)

Export it locally (do not commit this):

```bash
export PD_ROUTING_KEY='YOUR_INTEGRATION_KEY_HERE'
echo "Key length: ${#PD_ROUTING_KEY}"
```

## Architecture for this lab

```text
Prometheus rule fires
        ↓
Alertmanager
        ↓
PagerDuty Events API
        ↓
On-call notification
```

## Exercise 1 — Create Alertmanager PagerDuty config

Create `alertmanager-pagerduty.yaml`:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: alertmanager-pagerduty
  namespace: monitoring
type: Opaque
stringData:
  routingKey: "REPLACE_ME"
```

Apply with your key:

```bash
kubectl -n monitoring create secret generic alertmanager-pagerduty \
  --from-literal=routingKey="$PD_ROUTING_KEY" \
  --dry-run=client -o yaml | kubectl apply -f -
```

## Exercise 2 — Patch kube-prometheus-stack for PagerDuty

Create `pagerduty-values.yaml`:

```yaml
alertmanager:
  enabled: true
  config:
    global:
      resolve_timeout: 5m
    route:
      receiver: pagerduty-lab
      group_by: ["alertname", "namespace"]
      group_wait: 30s
      group_interval: 5m
      repeat_interval: 4h
    receivers:
      - name: pagerduty-lab
        pagerduty_configs:
          - routing_key_file: /etc/alertmanager/secrets/alertmanager-pagerduty/routingKey
            description: '{{ .CommonAnnotations.summary }}'
            severity: '{{ .CommonLabels.severity }}'
            client: kube-prometheus-lab
            client_url: 'http://localhost:9093'
```

Upgrade the stack (assumes release name `kps` from Module 03):

```bash
helm upgrade kps prometheus-community/kube-prometheus-stack \
  -n monitoring \
  -f pagerduty-values.yaml \
  --set alertmanager.alertmanagerSpec.secrets[0]=alertmanager-pagerduty \
  --reuse-values
```

Wait for Alertmanager to roll:

```bash
kubectl -n monitoring get pods | grep alertmanager
kubectl -n monitoring logs -l app.kubernetes.io/name=alertmanager --tail=50
```

## Exercise 3 — Create a simple firing alert rule

```bash
cat <<'EOF' | kubectl apply -f -
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: lab-always-firing
  namespace: monitoring
  labels:
    release: kps
spec:
  groups:
    - name: lab.rules
      rules:
        - alert: LabTestAlert
          expr: vector(1)
          for: 1m
          labels:
            severity: warning
          annotations:
            summary: "Lab test alert from Prometheus"
            description: "This alert always fires so beginners can test PagerDuty."
EOF
```

Watch alerts:

```bash
# Prometheus UI Alerts page, or:
kubectl -n monitoring port-forward svc/kps-kube-prometheus-stack-prometheus 9090:9090
# Open http://localhost:9090/alerts
```

**Expected in ~1–3 minutes:**  
`LabTestAlert` is **FIRING**, and a PagerDuty incident appears for your service.

## Exercise 4 — Acknowledge and resolve in PagerDuty

In PagerDuty UI:

1. Open the new incident
2. Click **Acknowledge**
3. Delete/disable the test rule:

```bash
kubectl -n monitoring delete prometheusrule lab-always-firing
```

4. Wait for resolve (or manually resolve in PagerDuty)

## Exercise 5 — Send a manual test event (no cluster)

Useful when debugging the key itself:

```bash
curl -X POST https://events.pagerduty.com/v2/enqueue \
  -H 'Content-Type: application/json' \
  -d "{
    \"routing_key\": \"$PD_ROUTING_KEY\",
    \"event_action\": \"trigger\",
    \"payload\": {
      \"summary\": \"Manual lab test from curl\",
      \"severity\": \"warning\",
      \"source\": \"gitops-observability-lab\"
    }
  }"
```

**Expected:** JSON response with `"status":"success"` and a new incident.

## Safety tips for beginners

- Never commit routing keys to Git
- Use a dedicated **lab** PagerDuty service
- Delete always-firing rules when done
- Start with `warning`, not `critical`, during training

## Module cleanup

```bash
kubectl -n monitoring delete prometheusrule lab-always-firing --ignore-not-found
kubectl -n monitoring delete secret alertmanager-pagerduty --ignore-not-found
```

## End-to-end checkpoint

You now have the full beginner loop:

1. **Helm** packages apps
2. **Argo CD** syncs desired state from Git
3. **Prometheus** scrapes metrics and evaluates rules
4. **Grafana** visualizes health
5. **PagerDuty** pages a human when it matters

## Checkpoint quiz

1. What is a PagerDuty integration/routing key?
2. Which component sends alerts to PagerDuty in this lab?
3. Why is an always-firing rule useful for training but dangerous in production?
