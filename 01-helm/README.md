# Module 01 — Helm for beginners

Helm is the package manager for Kubernetes. A **chart** is a folder of templates + default values.

## Concepts (60 seconds)

| Term | Meaning |
| --- | --- |
| Chart | App package (`Chart.yaml`, `templates/`, `values.yaml`) |
| Release | An installed instance of a chart |
| Values | Config you pass at install/upgrade time |
| Repo | Place that hosts charts (`bitnami`, `prometheus-community`, etc.) |

## Lab setup

```bash
# Create a practice namespace
kubectl create namespace helm-lab

# Add a public chart repo
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
helm search repo nginx
```

## Exercise 1 — Install NGINX with Helm

```bash
helm install web bitnami/nginx -n helm-lab

# Check the release
helm list -n helm-lab
kubectl get all -n helm-lab
```

**Expected:** A release named `web` and pods/services in `helm-lab`.

### Cleanup this exercise (optional)

```bash
helm uninstall web -n helm-lab
```

## Exercise 2 — Override values

```bash
helm install web2 bitnami/nginx -n helm-lab \
  --set service.type=ClusterIP \
  --set replicaCount=2

kubectl get deploy,svc -n helm-lab
```

**Expected:** Deployment shows `2/2` ready replicas.

## Exercise 3 — Upgrade and rollback

```bash
# Upgrade replica count
helm upgrade web2 bitnami/nginx -n helm-lab --set replicaCount=3
helm history web2 -n helm-lab

# Rollback to previous revision
helm rollback web2 1 -n helm-lab
helm history web2 -n helm-lab
```

**Expected:** History shows multiple revisions; rollback restores an older one.

## Exercise 4 — Use a values file

Create `values-dev.yaml`:

```yaml
replicaCount: 1
service:
  type: ClusterIP
```

Install with the file:

```bash
helm install web3 bitnami/nginx -n helm-lab -f values-dev.yaml
helm get values web3 -n helm-lab
```

## Exercise 5 — Create a tiny custom chart

```bash
mkdir -p charts && cd charts
helm create hello-helm
cd hello-helm

# Preview rendered manifests (no install)
helm template demo . 

# Install your chart
helm install demo . -n helm-lab
kubectl get pods -n helm-lab -l app.kubernetes.io/instance=demo
```

**Challenge:** Edit `values.yaml`, change `replicaCount` to `2`, then:

```bash
helm upgrade demo . -n helm-lab
```

## Useful commands

```bash
helm list -A
helm status <release> -n <ns>
helm get manifest <release> -n <ns>
helm uninstall <release> -n <ns>
```

## Module cleanup

```bash
helm uninstall web2 web3 demo -n helm-lab 2>/dev/null || true
kubectl delete namespace helm-lab
```

## Checkpoint quiz

1. What is the difference between a **chart** and a **release**?
2. Which command previews YAML without installing?
3. How do you undo a bad upgrade?
