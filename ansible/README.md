# How to run the Ansible playbook (using the dev inventory)

## Prerequisites
- A reachable Kubernetes cluster (Kind/K3s/Docker Desktop/EKS/etc.) and a working kubectl context
- Python 3.9+ and Ansible installed
- Ansible Kubernetes collection installed: kubernetes.core
- Kubeconfig available via $KUBECONFIG or default kubectl context

## Install requirements
```bash
# (optional) create and activate a virtualenv
python3 -m venv .venv && source .venv/bin/activate

# install Ansible
pip install "ansible>=8.0.0"

#install kubernetes python dependencies
pip install kubernetes

# install Kubernetes collection for Ansible
ansible-galaxy collection install kubernetes.core
```

## Run the playbook (dev)
```bash
ansible-playbook -i ansible/inventories/dev/hosts.ini ansible/playbook.yaml
```

## What this does
- Renders Kubernetes manifests from templates with dev variables
- Applies Deployment, Service, HPA, PDB, and Ingress idempotently
- Waits until the Deployment is available

## Check results
```bash
# list resources in dev namespace (adjust if you changed k8s_namespace)
kubectl get deploy,svc,hpa,pdb,ingress -n dev
```
