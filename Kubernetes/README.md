# Backend on Kubernetes — Complete README

## Overview

This repository contains a simple backend application (hashicorp/http-echo) deployed on Kubernetes with production-minded primitives: Deployment, Service, Ingress, Horizontal Pod Autoscaler (HPA), and PodDisruptionBudget (PDB). It also includes guidance for monitoring (ServiceMonitor via Prometheus Operator).
This document outlines the key decisions and assumptions made during the design and implementation of the Kubernetes resources for this project. It also presents future considerations for the project's evolution.

## Description and Assumptions

- This backend is a stateless application.[Kubernetes Stateless Deployment Documentation](https://kubernetes.io/docs/tasks/run-application/run-stateless-application-deployment/)
- The application uses the public "hashicorp/http-echo" image.
- All resources are deployed to the `test` namespace unless otherwise specified.
- Resource requests and limits are set based on usual estimated application usage.([Kubernetes Resources Documentation](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/))

## Prerequisites

- A working Kubernetes cluster (Docker Desktop, Kind, k3d, Minikube, EKS/GKE/AKS)
- kubectl installed and configured for the target cluster
- For Ingress testing: an Ingress controller (NGINX recommended below)
- For HPA: Metrics Server installed
- For ServiceMonitor: kube-prometheus-stack (Prometheus Operator CRDs)

## Quick Start: Deploy with kubectl

1) Apply the Namespace
```bash
kubectl apply -f backend-namespace.yaml
# Or: kubectl create namespace test
```

2) Apply application resources
```bash
# Deployment + Service
kubectl apply -f backend-deployment.yaml
kubectl apply -f backend-service.yaml

# HPA (requires Metrics Server)
kubectl apply -f backend-hpa.yaml

# PDB
kubectl apply -f k8s/backend-pdb.yaml

# Ingress (requires an Ingress controller)
kubectl apply -f k8s/backend-ingress.yaml
```

3) Verify rollout
```bash
kubectl get deploy,svc,hpa,pdb,ingress -n test
kubectl rollout status deployment/backend -n test
kubectl describe svc/backend-svc -n test
```

4) Test the application

- Directly via Service (port-forward):
```bash
kubectl -n dev port-forward svc/backend-svc 18080:80
curl http://localhost:18080/api
```

- Via Ingress (after installing an Ingress controller, see next section):
```bash
# If 8080 is busy, pick 8081/8888 instead
kubectl -n ingress-nginx port-forward svc/ingress-nginx-controller 8081:80
curl http://localhost:8081/api

# If the Ingress uses a host (e.g., api.local)
echo "127.0.0.1 api.local" | sudo tee -a /etc/hosts
curl -H "Host: api.local" http://localhost:8081/api
```

## Install an Ingress Controller (NGINX)

1) Install NGINX Ingress Controller
```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/cloud/deploy.yaml
```

2) Wait for controller readiness
```bash
kubectl -n ingress-nginx get pods
# Wait until ingress-nginx-controller is Running
```

3) Expose controller locally (choose a free port)
```bash
kubectl -n ingress-nginx port-forward svc/ingress-nginx-controller 8081:80
# Test:
curl -i http://localhost:8081/api
# Or with Host header if your Ingress defines spec.rules.host:
curl -i -H "Host: api.local" http://localhost:8081/api
```

Notes:
- Ensure your Ingress includes spec.ingressClassName: nginx.
- Ingress and Service must be in the same namespace.

## Install Metrics Server (for HPA)

If HPA shows “unknown” metrics or doesn’t scale:
```bash
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
kubectl get apiservices | grep metrics
```

## Optional Monitoring: Prometheus Operator (kube-prometheus-stack)

ServiceMonitor is a Custom Resource. To use it, install kube-prometheus-stack which provides Prometheus Operator and CRDs.

1) Install Helm (macOS: brew install helm)

2) Add repo and install the stack
```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
kubectl create namespace monitoring
helm install monitoring prometheus-community/kube-prometheus-stack -n monitoring
```

3) Docker Desktop note: node-exporter may crash due to mount propagation. Easiest fix is to disable node-exporter:
```bash
helm upgrade monitoring prometheus-community/kube-prometheus-stack -n monitoring -f - <<EOF
nodeExporter:
  enabled: false
EOF
```

4) Confirm CRDs
```bash
kubectl get crd | grep servicemonitors.monitoring.coreos.com
```

5) Apply the ServiceMonitor (adjust label “release” to your Helm release, e.g., monitoring)
```bash
kubectl apply -f k8s/backend-servicemonitor.yaml
```

6) Check Prometheus targets
```bash
kubectl -n monitoring port-forward svc/monitoring-kube-prometheus-prometheus 9090:9090
# Open http://localhost:9090 > Status > Targets
```

## Cleanup

```bash
kubectl delete -f backend-ingress.yaml --ignore-not-found
kubectl delete -f backend-pdb.yaml --ignore-not-found
kubectl delete -f backend-hpa.yaml --ignore-not-found
kubectl delete -f backend-service.yaml --ignore-not-found
kubectl delete -f backend-deployment.yaml --ignore-not-found
kubectl delete -f backend-namespace.yaml --ignore-not-found
```

---