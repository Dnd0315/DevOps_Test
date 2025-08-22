# Running the Ansible Playbook (dev inventory)

## Prerequisites

- Reachable Kubernetes cluster and `kubectl` configured  
- Python 3.9+ and Ansible installed  
- Ansible Kubernetes collection (`kubernetes.core`) installed  
- kubeconfig available via environment or default context

## Install Requirements

```bash
# Optional: create and activate a virtualenv
python3 -m venv .venv && source .venv/bin/activate

# Install Ansible
pip install "ansible>=8.0.0"

# Install Kubernetes Python client
pip install kubernetes

# Install Ansible Kubernetes collection
ansible-galaxy collection install kubernetes.core
```

## Run the Playbook

```bash
ansible-playbook -i ansible/inventories/dev/hosts.ini ansible/playbook.yaml
```

## What This Does

- Renders Kubernetes manifests with dev variables  
- Applies Deployment, Service, HPA, PDB, and Ingress idempotently  
- Waits until deployment is available

## Verify Resources

```bash
kubectl get deploy,svc,hpa,pdb,ingress -n dev
```

