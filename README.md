# Basic Flask App

**Updated:** 21 November 2026  
**Purpose:** Learn how to run a Flask application locally, package it as a Docker image, publish it to Docker Hub or Azure Container Registry (ACR), and deploy it to Azure Kubernetes Service (AKS).

## Diagrams

### Application architecture

![Flask application architecture](docs/architecture.mmd)

Source: [architecture diagram](docs/architecture.mmd).

### Build and deployment flow

![Container build and deployment flow](docs/deployment-flow.mmd)

Source: [deployment flow diagram](docs/deployment-flow.mmd).

## Concepts

| Term | Meaning |
| --- | --- |
| Flask | A lightweight Python web framework. |
| Docker | A platform for building and running container images. |
| Container | A running instance of an image, with the application and its dependencies. |
| Kubernetes | An open-source system for deploying and managing containerized applications. |
| ACR | Azure Container Registry, a private registry for storing container images. |
| AKS | Azure Kubernetes Service, a managed Kubernetes service on Azure. |

Typical workflow:

```text
Application source → Dockerfile → Docker image → Container
                                               ↓
                                      Registry (Docker Hub or ACR)
                                               ↓
                                           AKS cluster
```

## Prerequisites

- Git
- Python 3 and pip (for running locally)
- Docker Desktop, installed and running (for containers)
- Docker Hub account (if using Docker Hub)
- Azure CLI and an Azure subscription (for ACR and AKS)
- `kubectl` (the AKS setup can install it through `az aks install-cli`)

Install Docker Desktop on Windows using the [official guide](https://docs.docker.com/desktop/setup/install/windows-install/). Create or sign in to a [Docker Hub](https://hub.docker.com/) account if publishing there. In VS Code, the Docker and Dev Containers extensions are optional conveniences.

Check installations:

```bash
git --version
python --version
docker --version
az version
kubectl version --client
```

## 1. Get the application

```bash
git clone https://github.com/atulkamble/flask-app.git
cd flask-app
```

## 2. Run locally

### macOS and Linux

```bash
python3 -m venv .venv
source .venv/bin/activate
python -m pip install -r requirements.txt
python app.py
```

### Windows PowerShell

If Git or Python is not installed, install them using your preferred method. With Chocolatey, for example:

```powershell
choco install git -y
choco install python -y
```

Then open a new PowerShell window and run:

```powershell
git clone https://github.com/atulkamble/flask-app.git
cd flask-app
python --version
pip --version
python -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install -r requirements.txt
python app.py
```

Open <http://localhost:5000> in a browser. Stop the development server with `Ctrl+C`.

## 3. Build and run with Docker

Make sure Docker Desktop is running, then build the image from the repository root:

```bash
docker build -t pythonapp:local .
docker run --rm -d --name flask-app -p 5000:5000 pythonapp:local
```

Open <http://localhost:5000>. Stop and remove the container with:

```bash
docker stop flask-app
```

The Flask application must listen on `0.0.0.0` inside the container for the published port to be reachable. For example:

```python
app.run(host="0.0.0.0", port=5000)
```

### Publish to Docker Hub (optional)

Replace `<dockerhub-user>` with your Docker Hub username. Sign in interactively so Docker can prompt for credentials; do not put a password in this README or a shell command.

```bash
docker login
docker buildx build --platform linux/amd64,linux/arm64 \
  -t <dockerhub-user>/pythonapp:latest --push .
```

Pull and run the published image:

```bash
docker pull <dockerhub-user>/pythonapp:latest
docker run --rm -d --name flask-app -p 5000:5000 <dockerhub-user>/pythonapp:latest
```

## 4. Publish to Azure Container Registry (ACR)

Sign in and select the intended subscription. Use your actual registry name; the name must be globally unique and contain only lowercase letters and numbers. Replace placeholders below with values from your Azure environment.

```bash
az login
az account list --output table
az account set --subscription <subscription-id-or-name>
```

Create an ACR if you do not already have one:

```bash
az group create --name containers --location eastus
az acr create --resource-group containers --name <acr-name> --sku Basic
```

Build and push directly to ACR using Azure CLI (no Docker registry password is needed for this flow):

```bash
az acr login --name <acr-name>
docker buildx build --platform linux/amd64,linux/arm64 \
  -t <acr-name>.azurecr.io/cloudnautic/pythonapp:latest --push .
```

Check that the image was published:

```bash
az acr repository list --name <acr-name> --output table
az acr repository show-tags --name <acr-name> --repository cloudnautic/pythonapp --output table
```

If you use `docker login` with ACR instead, use `az acr login --name <acr-name>` or configure a scoped token/service principal. Avoid enabling the ACR admin account for routine use. Never commit registry passwords, tokens, or service-principal secrets to source control. If a real credential was previously pasted into a public or shared place, revoke or rotate it immediately.

## 5. Create an AKS cluster and connect

The following creates a basic two-node cluster. Azure resources can incur charges; review the configuration and clean up the resources when finished. Set the correct subscription before creating resources.

```bash
az group create --name containers --location eastus
az aks create \
  --resource-group containers \
  --name mycluster \
  --node-count 2 \
  --enable-managed-identity \
  --generate-ssh-keys
```

Install `kubectl` if needed, then fetch cluster credentials:

```bash
az aks install-cli
az aks get-credentials --resource-group containers --name mycluster --overwrite-existing
kubectl get nodes
```

## 6. Deploy to AKS

This repository should contain a `k8s/` directory with valid Kubernetes manifests. Confirm the actual filenames before applying them:

```bash
ls k8s
```

Update the Deployment manifest to reference the image pushed to your registry, for example:

```yaml
image: <acr-name>.azurecr.io/cloudnautic/pythonapp:latest
```

For AKS to pull a private ACR image, attach the registry to the cluster. This grants the AKS kubelet identity pull access:

```bash
az aks update --resource-group containers --name mycluster --attach-acr <acr-name>
```

Apply the manifests (adjust names to match the files in `k8s/`):

```bash
kubectl apply -f k8s/deployment.yml
kubectl apply -f k8s/service.yml
```

Check rollout and service status:

```bash
kubectl get deployments
kubectl get pods -o wide
kubectl get services
kubectl rollout status deployment/<deployment-name>
```

When the Service is `LoadBalancer`, wait for an external IP:

```bash
kubectl get service <service-name> --watch
```

Open `http://<external-ip>:<service-port>/` in a browser. The port is the Service's `port` value, which may differ from the container's `targetPort`.

## Troubleshooting

- **`docker` cannot connect:** Start Docker Desktop and wait until its engine is ready.
- **Browser cannot reach the container:** Confirm the Flask app binds to `0.0.0.0` and that the Docker port mapping matches the app and container ports.
- **`ImagePullBackOff` on AKS:** Verify the image name and tag, confirm the image exists in ACR, and run `az aks update --attach-acr` for the cluster and registry.
- **No external IP:** Confirm the Service type is `LoadBalancer`, then allow time for Azure to provision the public IP.
- **Manifest not found:** Check filenames and capitalization in `k8s/`; use the exact paths in `kubectl apply -f`.

## Clean up Azure resources

Delete the resource group to remove the AKS cluster, registry, and other resources created in it. This is destructive and removes everything in that group, so verify its contents first:

```bash
az group show --name containers --output table
az group delete --name containers --yes --no-wait
```

## Security note

Do not store credentials, access keys, subscription IDs, or other secrets in this README, source files, or Git history. Use Azure managed identity, scoped credentials, and secret storage. Rotate any credential that has been exposed.
