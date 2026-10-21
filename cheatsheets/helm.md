# Helm cheatsheet

```bash
# Repos
helm repo add <name> <url>
helm repo update
helm search repo <keyword>

# Install / upgrade
helm install <release> <chart> -n <ns> --create-namespace
helm upgrade --install <release> <chart> -n <ns> -f values.yaml
helm upgrade <release> <chart> -n <ns> --set key=value

# Inspect
helm list -A
helm status <release> -n <ns>
helm get values <release> -n <ns>
helm get manifest <release> -n <ns>
helm history <release> -n <ns>

# Rollback / delete
helm rollback <release> <revision> -n <ns>
helm uninstall <release> -n <ns>

# Develop
helm create <chart-name>
helm template <release> <chart>
helm lint <chart>
```
