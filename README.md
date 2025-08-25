# Kubernetes nginx with KIND

This project demonstrates how to deploy nginx on a Kubernetes cluster using KIND (Kubernetes in Docker). The setup includes a multi-node cluster with nginx deployed as both individual pods and a deployment with service exposure.

## Prerequisites

- Docker installed and running
- kubectl installed
- KIND installed

### Install KIND

```bash
# On macOS
brew install kind

# On Linux
curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.20.0/kind-linux-amd64
chmod +x ./kind
sudo mv ./kind /usr/local/bin/kind

# On Windows (using chocolatey)
choco install kind
```

## Project Structure

```
k8s/
├── README.md
├── config.yml              # KIND cluster configuration
└── nginx/
    ├── namespace.yml        # nginx namespace
    ├── pod.yml             # Individual nginx pod
    ├── deployment.yml      # nginx deployment (3 replicas)
    └── service.yml         # NodePort service for nginx
```

## Quick Start

### 1. Create KIND Cluster

```bash
# Create cluster with custom configuration
kind create cluster --config=config.yml --name nginx-cluster

# Verify cluster is running
kubectl cluster-info --context kind-nginx-cluster
```

### 2. Deploy nginx

```bash
# Create namespace
kubectl apply -f nginx/namespace.yml

# Deploy nginx pod
kubectl apply -f nginx/pod.yml

# Deploy nginx deployment (3 replicas)
kubectl apply -f nginx/deployment.yml

# Create service to expose nginx
kubectl apply -f nginx/service.yml
```

### 3. Verify Deployment

```bash
# Check namespace
kubectl get namespaces

# Check pods in nginx namespace
kubectl get pods -n nginx

# Check deployment
kubectl get deployment -n nginx

# Check service
kubectl get service -n nginx

# Get detailed service information
kubectl describe service nginx-service -n nginx
```

### 4. Access nginx

```bash
# Get the NodePort assigned to the service
kubectl get service nginx-service -n nginx

# Get node IP (for KIND, this will be localhost)
kubectl get nodes -o wide

# Access nginx using NodePort (replace <NODE_PORT> with actual port)
curl http://localhost:<NODE_PORT>
```

### 5. Port Forward (Alternative Access Method)

```bash
# Forward local port to service
kubectl port-forward service/nginx-service 8080:80 -n nginx

# Access nginx at http://localhost:8080
```

## Cluster Configuration

The KIND cluster is configured with:
- 1 control-plane node
- 2 worker nodes
- Kubernetes version 1.31.2

## nginx Configuration

- **Namespace**: `nginx`
- **Deployment**: 3 replicas of nginx:latest
- **Service**: NodePort type exposing port 80
- **Pod**: Single nginx pod for testing

## Useful Commands

### Cluster Management

```bash
# List KIND clusters
kind get clusters

# Delete cluster
kind delete cluster --name nginx-cluster

# Load local Docker image into KIND cluster
kind load docker-image nginx:latest --name nginx-cluster
```

### Debugging

```bash
# Get pod logs
kubectl logs -n nginx <pod-name>

# Execute into pod
kubectl exec -it -n nginx <pod-name> -- /bin/bash

# Describe resources
kubectl describe pod -n nginx <pod-name>
kubectl describe deployment -n nginx nginx-deployment
kubectl describe service -n nginx nginx-service
```

### Scaling

```bash
# Scale deployment
kubectl scale deployment nginx-deployment --replicas=5 -n nginx

# Check scaling status
kubectl get deployment nginx-deployment -n nginx -w
```

## Cleanup

```bash
# Delete all nginx resources
kubectl delete -f nginx/

# Delete KIND cluster
kind delete cluster --name nginx-cluster
```

## Troubleshooting

### Common Issues

1. **Port already in use**: If you get port binding errors, make sure no other services are using the same ports.
    For example:
    ```bash
    sudo lsof -i :8080
    ```
    
2. **Image pull errors**: KIND uses local Docker images. If using custom images, load them into KIND:
   ```bash
   kind load docker-image <image-name> --name nginx-cluster
   ```

3. **Service not accessible**: Check if the NodePort service is correctly configured and the port is within the valid range (30000-32767 for NodePort).

### Verification Steps

```bash
# Check cluster status
kubectl get nodes

# Check all resources in nginx namespace
kubectl get all -n nginx

# Check events for debugging
kubectl get events -n nginx --sort-by='.lastTimestamp'
```

## Notes

- This setup uses `nginx:latest` image. For production, use specific version tags.
- The NodePort service exposes nginx on a random high port. Use LoadBalancer or Ingress for production.
- KIND is intended for development and testing, not production use.

---
🧑🏻‍💻 Rakshit Malik
