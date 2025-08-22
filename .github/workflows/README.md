# Deployment with GitHub Actions

This document explains how to use GitHub Actions to automate the deployment of our backend Kubernetes application.


## Prerequisites

- A GitHub repository configured with GitHub Actions workflows in the `.github/workflows/` directory  
- A **self-hosted** GitHub Actions runner configured and active if you want to run jobs on a specific machine (in this case, macOS)  
- Administrative access to configure necessary GitHub secrets

***

## Installing the Self-Hosted Runner

To run GitHub Actions jobs on your own hardware or server, you need to install and configure a self-hosted runner:

### Step 1: Download the Runner

- Go to your GitHub repository on the web  
- Navigate to **Settings > Actions > Runners**  
- Click **Add runner** to register a new runner  
- Select your operating system (Linux, macOS, Windows)  
- Download the runner package from the link provided

### Step 2: Configure the Runner

- Extract the downloaded package to a directory on your machine  
- Open a terminal and navigate to that directory  
- Run the configuration script with the URL of your repository and the generated token:

```bash
# Example for Linux/macOS
./config.sh --url https://github.com/OWNER/REPO --token YOUR_TOKEN
```

- Replace `OWNER/REPO` with your GitHub repository path  
- Replace `YOUR_TOKEN` with the one shown on GitHub during runner setup (valid for a short time only)

### Step 3: Start the Runner

- From the runner directory, start the runner process interactively with:

```bash
./run.sh
```

- The runner will wait and listen for job requests from GitHub Actions

### Step 4 (Optional): Install as a Service

For continuous availability without manual start, install the runner as a service:

```bash
sudo ./svc.sh install
sudo ./svc.sh start
```

This will launch the runner automatically on system boot.

### Step 5: macOS Permission Workaround

On macOS, the runner’s Python setup step may attempt to write to `/Users/runner` and fail due to permissions. To fix this:

```bash
sudo mkdir -p /Users/runner
sudo chown $(whoami) /Users/runner
chmod 700 /Users/runner
```

This creates the folder and grants write permissions to your user, allowing the runner to function normally.

***

## Using a Python Virtual Environment in GitHub Actions

Because Homebrew-managed Python on macOS uses an externally managed environment preventing system-wide pip installs, this workflow uses a Python virtual environment (venv) to isolate dependencies.

The workflow:

- Creates a Python virtual environment  
- Installs `ansible` and `kubernetes` libraries into the venv  
- Runs the playbook using the Python interpreter from the venv to ensure all dependencies are found

***

## Workflow Triggering

### Automatic

- On each push or pull request to the `main` branch (or other configured branches)  
- On scheduled events defined in the GitHub Actions workflow file  

### Manual

- From the **Actions** tab of the GitHub repository  
- Select the desired workflow  
- Click **Run workflow** to trigger manually (requires `workflow_dispatch` in the YAML file)  

***

## Required Variables and Secrets

- Configure secrets in GitHub (e.g., Kubernetes cluster access, tokens, credentials) under **Settings > Secrets and variables > Actions**  
- Environment variables for Ansible such as `KUBECONFIG` or others depending on your setup

***

