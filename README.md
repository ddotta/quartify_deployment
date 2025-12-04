# Quartify Deployment

 [Version française](README_FR.md)

Helm/Kubernetes configuration to deploy the Quartify web application on SSP Cloud (Onyxia).

## Description

This repository contains the deployment configuration for [Quartify](https://github.com/ddotta/quartify), an R package that automatically converts R scripts into Quarto markdown documents (.qmd).

**Docker Image**: [ddottaagr/quartify:latest](https://hub.docker.com/r/ddottaagr/quartify)

## Application Features

The Quartify web application offers:

-  Interactive Shiny interface (EN/FR)
-  R  Quarto (.qmd) conversion
-  **HTML rendering with Quarto** (Quarto is installed in the Docker image)
-  Support for callouts, tabsets, Mermaid diagrams
-  Source line numbers for traceability
-  Customizable Quarto themes (25+ themes)
-  Download generated .qmd and .html files

## Deployment on SSP Cloud

### Prerequisites

- Access to [SSP Cloud](https://datalab.sspcloud.fr/)
- Active user account

### Option 1: Via Web Interface (Recommended)

1. Log in to https://datalab.sspcloud.fr/
2. Go to **My Lab**  **Service Catalog**
3. Search for "Shiny" or create a custom service
4. In the configuration:
   - **Repository**: `https://github.com/ddotta/quartify_deployment`
   - **Chart**: `.` (root of the repository)
   - **Values file**: `values-sspcloud.yaml`
5. Click **Launch**
6. Once the service is started, click on the provided URL

### Option 2: Via Helm CLI

```bash
# Clone the repository
git clone https://github.com/ddotta/quartify_deployment.git
cd quartify_deployment

# Install with Helm
helm install quartify . -f values-sspcloud.yaml

# Get the service URL
kubectl get ingress
```

### Option 3: Via kubectl

```bash
# Apply manifests directly
kubectl apply -f templates/
```

## Configuration

### Main Files

- **`Chart.yaml`**: Helm chart metadata
- **`values.yaml`**: Default configuration
- **`values-sspcloud.yaml`**: SSP Cloud specific configuration
- **`templates/`**: Kubernetes manifests (Deployment, Service, Ingress)

### Customization

To modify the configuration, edit `values-sspcloud.yaml`:

```yaml
image:
  repository: ddottaagr/quartify
  tag: latest
  pullPolicy: Always

resources:
  limits:
    memory: "2Gi"
    cpu: "2000m"
  requests:
    memory: "1Gi"
    cpu: "1000m"

service:
  type: ClusterIP
  port: 3838

ingress:
  enabled: true
  hosts:
    - host: quartify.lab.sspcloud.fr
```

## Repository Structure

```
quartify_deployment/
 Chart.yaml                 # Helm metadata
 values.yaml                # Default configuration
 values-sspcloud.yaml       # SSP Cloud configuration
 templates/
    deployment.yaml        # Kubernetes Deployment
    service.yaml          # Kubernetes Service
    ingress.yaml          # Ingress for exposure
    _helpers.tpl          # Helm template helpers
 README.md                  # This file
 README_FR.md              # French version
```

## Docker Image

The `ddottaagr/quartify` Docker image contains:

- **R 4.4.1** (rocker/shiny)
- **Shiny Server**
- **Quarto 1.4.549**  (enables HTML rendering)
- **quartify package** (installed from GitHub)
- All required R dependencies

### Building the Image

The image is automatically built via GitHub Actions on every push to the `main` branch of the [quartify](https://github.com/ddotta/quartify) repository.

To build manually:

```bash
# Clone the main repository
git clone https://github.com/ddotta/quartify.git
cd quartify

# Build the image
docker build -t ddottaagr/quartify:latest .

# Test locally
docker run -p 3838:3838 ddottaagr/quartify:latest
```

## Troubleshooting

### Service Won't Start

```bash
# Check pod logs
kubectl get pods
kubectl logs <pod-name>

# Check events
kubectl describe pod <pod-name>
```

### Resource Issues

If the pod is in `OOMKilled` state (out of memory), increase limits in `values-sspcloud.yaml`:

```yaml
resources:
  limits:
    memory: "4Gi"  # Increase to 4GB
```

### Application Not Responding

Verify the port is correct:

```bash
# Service should be on port 3838
kubectl get service quartify
```

## Updates

To update the application with a new image version:

```bash
# Update the deployment
helm upgrade quartify . -f values-sspcloud.yaml

# Or force restart
kubectl rollout restart deployment/quartify
```

## Useful Links

-  **Main Repository**: https://github.com/ddotta/quartify
-  **Docker Image**: https://hub.docker.com/r/ddottaagr/quartify
-  **Documentation**: https://ddotta.github.io/quartify/
-  **SSP Cloud**: https://datalab.sspcloud.fr/

## Support

For any questions or issues:

- Open an [issue on GitHub](https://github.com/ddotta/quartify/issues)
- Consult the [complete documentation](https://ddotta.github.io/quartify/)

## License

This deployment project follows the same license as the main Quartify project (MIT).
