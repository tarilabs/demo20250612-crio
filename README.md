# CRI-O KinD Image Builder

This project provides a GitHub Action to build CRI-O enabled KinD (Kubernetes in Docker) images, following the guide from [rkiselenko.dev](https://rkiselenko.dev/blog/crio-in-kind/).

## Overview

The GitHub Action builds a custom KinD node image with CRI-O container runtime instead of the default containerd. This is useful for testing Kubernetes applications with CRI-O or for environments that specifically require CRI-O.

## Features

- 🤔 Builds KinD base image from source
- 🤔 Creates Kubernetes node image with specified version
- ✅ Installs and configures CRI-O runtime
- ✅ Publishes images to GitHub Container Registry
- ✅ Supports configurable Kubernetes and CRI-O versions
- 🤔 Includes testing and validation steps

## Usage

### Triggering the Build

#### Manual Trigger (Workflow Dispatch)

You can manually trigger the build action from the GitHub Actions tab:

1. Go to the "Actions" tab in your repository
2. Select "Build CRI-O KinD Image" workflow
3. Click "Run workflow"
4. Specify:
   - **Kubernetes version** (e.g., `v1.30.0`)
   - **CRI-O version** (e.g., `v1.30`)

#### Automatic Triggers

The workflow also runs automatically on:
- Push to `main` branch
- Pull requests to `main` branch

### Using the Built Image

Once the GitHub Action completes, the CRI-O enabled KinD image will be available at:

```
ghcr.io/[your-username]/[repository-name]/kindnode-crio:[crio-version]
```

#### Creating a KinD Cluster with CRI-O

1. **Pull the image** (optional, KinD will pull automatically):
   ```bash
   docker pull ghcr.io/[your-username]/[repository-name]/kindnode-crio:v1.30
   ```

2. **Create the cluster**:
   ```bash
   kind create cluster --image ghcr.io/[your-username]/[repository-name]/kindnode-crio:v1.30 --config kind-crio.yaml
   ```

3. **Verify CRI-O is running**:
   ```bash
   kubectl get nodes -o wide
   docker exec -it kind-control-plane crictl version
   docker exec -it kind-control-plane crio version
   ```

### Example: Complete Workflow

```bash
# Clone this repository
git clone https://github.com/[your-username]/[repository-name].git
cd [repository-name]

# Create KinD cluster with CRI-O
kind create cluster --image ghcr.io/[your-username]/[repository-name]/kindnode-crio:v1.30 --config kind-crio.yaml

# Deploy a test application
kubectl apply -f - <<EOF
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-test
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx-test
  template:
    metadata:
      labels:
        app: nginx-test
    spec:
      containers:
      - name: nginx
        image: nginx:alpine
        ports:
        - containerPort: 80
EOF

# Verify the pod is running
kubectl get pods
kubectl logs -l app=nginx-test

# Clean up
kind delete cluster
```

## Files Structure

```
.
├── .github/
│   └── workflows/
│       └── build-crio-kind-image.yml  # Main GitHub Action workflow
├── Dockerfile.crio                    # Dockerfile to install CRI-O
├── kind-crio.yaml                     # KinD cluster configuration for CRI-O
└── README.md                          # This file
```

## Build Process

The GitHub Action follows these steps:

1. **Setup Environment**: Install Go, Docker Buildx, and KinD
2. **Build Base Image**: Clone KinD sources and build the base image
3. **Build Node Image**: Download Kubernetes sources and build the node image
4. **Install CRI-O**: Create a new image with CRI-O installed and configured
5. **Publish**: Tag and push the final image to GitHub Container Registry
6. **Test**: Validate that CRI-O components are working in the image

## Configuration

### Supported Versions

- **Kubernetes**: Any version supported by KinD (typically v1.25+)
- **CRI-O**: Versions available in the official CRI-O package repositories

### Default Versions

- Kubernetes: `v1.30.0`
- CRI-O: `v1.30`

### Customization

You can customize the build by:

1. **Modifying versions**: Change the default versions in the workflow file
2. **Adding packages**: Extend `Dockerfile.crio` to install additional packages
3. **Cluster configuration**: Modify `kind-crio.yaml` for different cluster topologies

## Troubleshooting

### Common Issues

1. **Build fails during Kubernetes compilation**:
   - Ensure the Kubernetes version exists and is compatible with the Go version
   - Check if sufficient disk space and memory are available

2. **CRI-O installation fails**:
   - Verify the CRI-O version is available in the package repository
   - Check if the version format matches the expected pattern

3. **Image won't start**:
   - Ensure all systemd services are properly configured
   - Check container logs for service startup errors

### Debugging

To debug issues with the built image:

```bash
# Run the image interactively
docker run -it --privileged ghcr.io/[your-username]/[repository-name]/kindnode-crio:v1.30 /bin/bash

# Check CRI-O status
systemctl status crio

# Check CRI-O configuration
cat /etc/crio/crio.conf

# Test CRI-O socket
crictl version
```

## Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Test the changes by running the GitHub Action
5. Submit a pull request

## References

- [Original Guide](https://rkiselenko.dev/blog/crio-in-kind/) by Roman Kiselenko
- [KinD Documentation](https://kind.sigs.k8s.io/)
- [CRI-O Documentation](https://cri-o.io/)
- [Kubernetes Documentation](https://kubernetes.io/docs/) 