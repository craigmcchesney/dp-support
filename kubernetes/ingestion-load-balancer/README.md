# Kubernetes Ingestion Load Balancer

This directory contains Kubernetes manifests that replace the Docker Compose `ingestion-load-balancer` configuration with native Kubernetes load balancing and dynamic scaling.

## Key Differences from Docker Compose

| Component | Docker Compose | Kubernetes |
|-----------|----------------|------------|
| **Load Balancing** | Static Envoy proxy with 2 hardcoded servers | Kubernetes Service with dynamic pod selection |
| **Scaling** | Fixed 2 replicas | HorizontalPodAutoscaler (2-10 replicas based on CPU/memory) |
| **Service Discovery** | Docker network + static hostnames | Kubernetes DNS + service names |
| **Port Exposure** | Envoy on 8080 → 60051 | LoadBalancer service on 8080 → 60051 |

## Files Overview

- **00-namespace.yaml**: Creates isolated namespace `ingestion-lb`
- **01-mongodb-deployment.yaml**: MongoDB single instance with resource limits
- **02-mongodb-service.yaml**: Internal ClusterIP service for MongoDB
- **03-ingestion-deployment.yaml**: Ingestion service with 2 initial replicas and health checks
- **04-ingestion-service.yaml**: LoadBalancer service exposing ingestion pods
- **05-hpa.yaml**: HorizontalPodAutoscaler for dynamic scaling (2-10 replicas)
- **06-deployment-script.yaml**: Step-by-step deployment commands

## Prerequisites

1. **Kubernetes cluster** (minikube, kind, or cloud provider)
2. **kubectl** configured to connect to your cluster
3. **Metrics server** installed for HPA to work:
   ```bash
   kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
   ```
4. **Container image** built and available (see notes below)

## Container Image Notes

The configuration assumes `dp-ingestion-service:latest` is available. You'll need to:

1. **Build the image** from your existing Dockerfile:
   ```bash
   docker build -t dp-ingestion-service:latest -f docker/applications/JavaApp/Dockerfile .
   ```

2. **For cloud clusters**, push to a registry:
   ```bash
   docker tag dp-ingestion-service:latest your-registry/dp-ingestion-service:latest
   docker push your-registry/dp-ingestion-service:latest
   ```
   Then update `03-ingestion-deployment.yaml` with the registry URL.

## Learning Points

### 1. **Namespace Isolation**
Instead of Docker networks, Kubernetes uses namespaces to isolate resources.

### 2. **Services vs Envoy**
- Docker Compose used Envoy for load balancing
- Kubernetes Services provide built-in load balancing with better integration

### 3. **Dynamic Scaling**
- HPA monitors CPU/memory and automatically adds/removes pods
- Scaling policies control how aggressively to scale up/down

### 4. **Health Checks**
- `livenessProbe`: Restart container if unhealthy
- `readinessProbe`: Only send traffic when ready

### 5. **Resource Management**
- `requests`: Guaranteed resources for scheduling
- `limits`: Maximum resources the container can use

## Testing Autoscaling

1. Deploy the configuration
2. Generate load using your benchmark client
3. Watch pods scale automatically:
   ```bash
   kubectl get hpa -n ingestion-lb -w
   kubectl get pods -n ingestion-lb -w
   ```

## Next Steps

- Add persistent storage for MongoDB
- Implement proper monitoring with Prometheus
- Add ingress controller for better external access
- Consider service mesh (Istio) for advanced traffic management