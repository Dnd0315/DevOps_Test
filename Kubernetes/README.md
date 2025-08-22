# Kubernetes Resources - Decisions and Assumptions

## Overview

This document outlines the key decisions and assumptions made during the design and implementation of the Kubernetes resources for this project. It also presents future considerations for the project's evolution.

## Description and Assumptions

- This backend is a stateless application.[Kubernetes Stateless Deployment Documentation](https://kubernetes.io/docs/tasks/run-application/run-stateless-application-deployment/)
- The application uses the public "hashicorp/http-echo" image.
- All resources are deployed to the `test` namespace unless otherwise specified.
- Ingress controllers are pre-installed and configured by the cluster administrator.
- Resource requests and limits are set based on usual estimated application usage.([Kubernetes Resources Documentation](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/))



## Key Decisions

- **Deployment Strategy:** Used `Deployment` resources for stateless workloads to enable rolling updates and easy scaling.
- **Service Exposure:** Internal communication uses `ClusterIP` services; external access is provided via `Ingress` and `LoadBalancer` services as needed.
- **Resource Management:** Set conservative CPU and memory requests/limits to ensure fair scheduling and avoid resource contention.
- **Health Checks:** Configured `livenessProbe` and `readinessProbe` for all pods to ensure proper lifecycle management.
- **Auto-scaling:** Enabled Horizontal Pod Autoscaler (HPA) where applicable, based on CPU utilization, with `maxReplicas` set to 10 and `minReplicas` set to 3.
- **Disruption Management:** Enabled Pod Disruption Budget with `minAvailable` set to 2.

## Future Considerations

- **Configuration Management:** Application configuration is injected using `ConfigMap` and `Secret` resources.
- **RBAC:** Applied least-privilege principles for service accounts and roles.
- **Logging & Monitoring:** Assumed integration with cluster-level logging and monitoring solutions (e.g., Prometheus, ELK).
- **Resource Management:** Review and adjust resource requests/limits based on real usage metrics.
- **Network:**Implement network policies for enhanced security.
- **CI/CD:**Integrate with CI/CD pipelines for automated deployments.
- **Vulnerabilities:**Regularly update images and dependencies to address vulnerabilities.

---
This README should be updated as the Kubernetes resources evolve.