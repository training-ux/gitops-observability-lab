# Module 02 — Argo CD for beginners

Argo CD watches Git and keeps your cluster matching the desired state (**GitOps**).

## Concepts (60 seconds)

| Term | Meaning |
| --- | --- |
| Application | Argo CD object that syncs one Git path/chart into a namespace |
| Sync | Apply Git state to the cluster |
| Self-heal | Auto-correct cluster drift back to Git |
| App of Apps | One root app that creates child apps |

## Exercise 1 — Install Argo CD

```bash
kubectl create namespace argocd

helm repo add argo https://argoproj.github.io/argo-helm
helm repo update

helm upgrade --install argocd argo/argo-cd \
  -n argocd \
  --set server.service.type=LoadBalancer \
  --wait
```

Get the admin password:

```bash
kubectl -n argocd get secret argocd-initial-admin-secret \
  -o jsonpath='{.data.password}' | base64 -d
echo
```

Find the UI address:

```bash
kubectl -n argocd get svc argocd-server
# Login: username = admin , password = value from previous command
```

Local-only access alternative:

```bash
kubectl -n argocd port-forward svc/argocd-server 8080:443
# Open https://localhost:8080
```

## Exercise 2 — Deploy a sample app from Git

Create `guestbook-app.yaml`:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: guestbook
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/argoproj/argocd-example-apps.git
    targetRevision: HEAD
    path: guestbook
  destination:
    server: https://kubernetes.default.svc
    namespace: guestbook
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
      - CreateNamespace=true
```

Apply it:

```bash
kubectl apply -f guestbook-app.yaml
kubectl -n argocd get applications
kubectl get pods -n guestbook
```

**Expected:** Application `guestbook` becomes `Synced` / `Healthy`.

## Exercise 3 — Watch GitOps sync

```bash
# See app details
kubectl -n argocd get app guestbook -o yaml | less

# Force a refresh
kubectl -n argocd patch app guestbook --type merge \
  -p '{"metadata":{"annotations":{"argocd.argoproj.io/refresh":"hard"}}}'
```

Break and heal (self-heal demo):

```bash
# Manually scale (creates drift)
kubectl -n guestbook scale deploy/guestbook-ui --replicas=3

# Wait ~1-3 minutes (or refresh) — Argo CD should restore desired replicas
kubectl -n guestbook get deploy guestbook-ui
```

## Exercise 4 — Helm chart via Argo CD

Create `nginx-argocd-app.yaml`:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: nginx-helm
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://charts.bitnami.com/bitnami
    chart: nginx
    targetRevision: 19.0.0
    helm:
      values: |
        service:
          type: ClusterIP
        replicaCount: 1
  destination:
    server: https://kubernetes.default.svc
    namespace: nginx-gitops
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
      - CreateNamespace=true
```

```bash
kubectl apply -f nginx-argocd-app.yaml
kubectl -n argocd get app nginx-helm
kubectl get all -n nginx-gitops
```

## Useful CLI commands (optional)

```bash
# Install CLI: https://argo-cd.readthedocs.io/en/stable/cli_installation/
argocd login localhost:8080 --username admin --password '<password>' --insecure
argocd app list
argocd app get guestbook
argocd app sync guestbook
```

## Module cleanup

```bash
kubectl -n argocd delete app guestbook nginx-helm --ignore-not-found
# Optional full remove:
# helm uninstall argocd -n argocd
# kubectl delete ns argocd guestbook nginx-gitops
```

## Checkpoint quiz

1. What problem does GitOps solve?
2. What does `selfHeal: true` do?
3. Where does Argo CD store the desired state?
