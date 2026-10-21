# PromQL cheatsheet (beginner)

```promql
# Instant vector
up
http_requests_total

# Rate of increase per second
rate(http_requests_total[5m])

# Aggregate
sum(rate(http_requests_total[5m]))
sum by (namespace) (rate(container_cpu_usage_seconds_total[5m]))

# Filters
up{job="kubernetes-pods"}
kube_pod_info{namespace="default"}

# Comparisons (for alerts)
rate(http_requests_total[5m]) > 10
```

## Reading tips

- `[5m]` = look at the last 5 minutes
- `rate()` needs a range vector (`metric[5m]`)
- `sum by (label)` groups results
