# kubectl cheatsheet (lab essentials)

```bash
# Context / cluster
kubectl config current-context
kubectl get nodes
kubectl cluster-info

# Namespaces & workloads
kubectl get ns
kubectl get pods -A
kubectl get deploy,svc,ing -n <ns>
kubectl describe pod <pod> -n <ns>
kubectl logs <pod> -n <ns> --tail=100
kubectl logs -f deploy/<name> -n <ns>

# Apply / delete
kubectl apply -f file.yaml
kubectl delete -f file.yaml
kubectl delete ns <ns>

# Debug
kubectl get events -n <ns> --sort-by=.lastTimestamp
kubectl port-forward svc/<name> 8080:80 -n <ns>
kubectl exec -it <pod> -n <ns> -- /bin/sh
```
