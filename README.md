# GitOps & Observability Lab

Beginner-friendly exercises for **Helm**, **Argo CD**, **Prometheus**, **Grafana**, and **PagerDuty**.

Each module has:

1. Short concept notes
2. Copy-paste commands
3. Simple exercises with expected outcomes

## Learning path

| Order | Module | What you learn |
| --- | --- | --- |
| 1 | [01-helm](./01-helm) | Package and install Kubernetes apps |
| 2 | [02-argocd](./02-argocd) | GitOps continuous delivery |
| 3 | [03-prometheus](./03-prometheus) | Metrics scraping and PromQL basics |
| 4 | [04-grafana](./04-grafana) | Dashboards and visualization |
| 5 | [05-pagerduty](./05-pagerduty) | Alerting and on-call routing |

## Prerequisites

- A Kubernetes cluster (`minikube`, `kind`, or EKS)
- `kubectl`, `helm`, and `git` installed
- Optional: Argo CD CLI (`argocd`)

Check your tools:

```bash
kubectl version --client
helm version
kubectl get nodes
```

## Quick start (kind example)

```bash
# Create a local cluster (optional)
kind create cluster --name gitops-lab

# Confirm cluster access
kubectl cluster-info
kubectl get ns
```

## Suggested pace

- **Day 1:** Helm + Argo CD
- **Day 2:** Prometheus + Grafana
- **Day 3:** PagerDuty integration + full flow review

## Cheatsheets

- [Helm commands](./cheatsheets/helm.md)
- [kubectl essentials](./cheatsheets/kubectl.md)
- [PromQL basics](./cheatsheets/promql.md)

## Related project

Production-style stack example: [observability-eks](https://github.com/training-ux/observability-eks)

## License

MIT — use freely for training and demos.
