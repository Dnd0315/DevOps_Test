# Deployment with GitHub Actions

This document explains how to use GitHub Actions to automate the deployment of our backend Kubernetes application.

## Prerequisites

- A GitHub repository with workflows defined in `.github/workflows/`  
- A **self-hosted** GitHub Actions runner on macOS configured and running  
- Administrative access to configure required GitHub secrets

## Installing the Self-Hosted Runner

1. **Download the runner**  
   Navigate to **Settings > Actions > Runners** in your GitHub repository, click **Add runner**, select macOS, and download the package.

2. **Configure the runner**  
   Extract the package, open a terminal in its folder, and run:  
   ```
   ./config.sh --url https://github.com/OWNER/REPO --token YOUR_TOKEN
   ```
   Replace `OWNER/REPO` and `YOUR_TOKEN` accordingly.

3. **Start the runner**  
   Run interactively:  
   ```
   ./run.sh
   ```

4. **Optional: Install as a service**  
   To run the runner automatically on system boot:  
   ```
   sudo ./svc.sh install
   sudo ./svc.sh start
   ```

5. **macOS permission workaround**  
   The runner may fail when writing to `/Users/runner`. Fix with:  
   ```
   sudo mkdir -p /Users/runner
   sudo chown $(whoami) /Users/runner
   chmod 700 /Users/runner
   ```

## Using a Python Virtual Environment in GitHub Actions

Due to Homebrew's macOS Python system restrictions, the workflow uses a Python virtual environment:

- Creates a Python virtual environment  
- Installs `ansible` and `kubernetes` packages inside it  
- Runs the playbook with the venv’s Python interpreter ensuring dependencies are available

## Workflow Triggering

- **Automatic:** On push or pull requests to `main` branch  
- **Manual:** From the GitHub **Actions** tab, select the workflow, and click **Run workflow**

## Required Variables and Secrets

- Configure secrets (Kubernetes access credentials, tokens, etc.) under **Settings > Secrets and variables > Actions**  
- Environment variables such as `KUBECONFIG` as needed for Ansible
