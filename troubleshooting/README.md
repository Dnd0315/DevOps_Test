# Troubleshooting & Optimization 

## CrashLoopBackOff
- kubectl describe pod pod_name -n namespace : inspect Events for image pull issues or failing probes. Works on Deployment also.
- kubectl logs pod_name -n namespace: verify container command/args and readiness/liveness endpoints.
- Validate resource limits and check pod status for OOMKilled events.
- Reproduce locally to isolate image issues:
  - docker run --rm -p 5678:5678 hashicorp/http-echo -text="hello world"

## Performance

### Metrics to check:
- kubectl top pods -n namespace and describe HPA: see throttling, autoscaling behavior.
- kubectl describe ingress backend-ingress -n namespace: check 5xx/timeout events for /api.
- Inspect ingress controller metrics/logs for namespace  upstream timeouts on /api.

### Targeted optimizations for /api
- Capacity and autoscaling
- Raise minReplicas for backend to match baseline peak (avoid cold start during spikes).
- Right-size CPU/memory requests to reduce CPU throttling; ensure limits aren’t too tight.

