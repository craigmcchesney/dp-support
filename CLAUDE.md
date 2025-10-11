# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This is dp-support, a utilities repository for managing the Data Platform ecosystem. The Data Platform consists of Java-based microservices for capturing and accessing particle accelerator facility data, along with supporting infrastructure like MongoDB, Envoy proxy, and Apache web server.

## Key Commands

### Service Management
- Start services: `./bin/server-{service}-start` (ingest, query, annotation, ingestion-stream)
- Stop services: `./bin/server-{service}-stop` 
- Check status: `./bin/server-{service}-status`
- Benchmark versions: `./bin/server-benchmark-{service}-start` (uses different ports/databases)

### MongoDB Management
**Local installation (systemctl):**
- `./bin/mongodb-systemctl-start/stop/status/enable/disable`

**Docker deployment:**
- `./bin/mongodb-docker-create/start/stop/remove`
- `./bin/mongodb-docker-shell` - Access mongosh shell
- `./bin/mongodb-compass-start` - Launch GUI with default connection

### Docker Infrastructure
**Envoy proxy:** `./bin/envoy-docker-create/start/stop/remove`
**Apache server:** `./bin/apache-docker-create/start/stop/remove`

### Applications
- `./bin/app-run-desktop-app` - Start Java desktop GUI
- `./bin/app-run-test-data-generator` - Generate sample data (uses fixed date 2023-10-31T15:51:00.000+00:00)
- `./bin/app-run-{ingestion|query}-benchmark` - Run performance benchmarks
- `./bin/app-run-{ingestion|query}-bytes-benchmark` - Byte-based benchmark variants

### Docker Compose Ecosystems
**Full ecosystem:**
```bash
docker compose -f ~/data-platform/docker/docker-compose/mldp-ecosystem/docker-compose.yml -p mldp-ecosystem up -d
~/data-platform/bin/app-run-docker-test-data-generator
```

**Load balancer demo:**
```bash
docker compose -f ~/data-platform/docker/docker-compose/ingestion-load-balancer/docker-compose.yml -p ingestion-load-balancer up -d
~/data-platform/bin/app-run-docker-lb-ingestion-benchmark
```

### Kubernetes Load Balancer (Minikube)
**Deployment:**
```bash
cd kubernetes/ingestion-load-balancer
# Load Docker image into minikube first
minikube image load dp-ingestion-service:latest
# Deploy all components
kubectl apply -f .
```

**Key Commands:**
- `kubectl get svc -n ingestion-lb` - Check service endpoints
- `kubectl get pods -n ingestion-lb` - Check pod status  
- `kubectl get hpa -n ingestion-lb` - Check horizontal pod autoscaler
- `kubectl logs -f deployment/ingestion-service -n ingestion-lb` - Monitor logs
- `kubectl scale deployment ingestion-service --replicas=N -n ingestion-lb` - Manual scaling

**External Access (Minikube):**
- MongoDB: `$(minikube ip):30017`
- Ingestion gRPC: `$(minikube ip):31406` 
- Ingestion HTTP: `$(minikube ip):32607`
- Get minikube IP: `minikube ip`
- Get service URLs: `minikube service ingestion-service -n ingestion-lb --url`

**Monitoring HPA Scaling:**
```bash
kubectl get hpa -n ingestion-lb -w
kubectl get pods -n ingestion-lb -w
kubectl top pods -n ingestion-lb
```

**Cleanup:**
```bash
kubectl delete namespace ingestion-lb
```

## Architecture

### Process Management
The `util-pm-start`, `util-pm-stop`, and `util-pm-status` scripts provide process management using:
- PID files in `$DP_HOME/var/lock/`
- Log files in `$DP_HOME/var/log/`
- Process verification via `ps` command

### Service Components
**Core services (Java-based, use dp-service.jar):**
- Ingestion Service (port 50051) - Data ingestion via gRPC
- Query Service (port 50052) - Data retrieval via gRPC  
- Annotation Service (port 50053) - Metadata management
- Ingestion Stream Service (port 50054) - Streaming ingestion

**Infrastructure:**
- MongoDB (port 27017) - Persistence layer, default credentials admin/admin
- Envoy Proxy (port 8080) - HTTP1 to HTTP2/gRPC translation for web apps
- Apache Server - File serving for export operations

### Docker Configuration
- Single Dockerfile in `docker/applications/JavaApp/Dockerfile` using OpenJDK 21
- JAR files stored in `lib/` directory
- Envoy configurations in `config/envoy.yaml` and `config/envoy.mac.yaml`
- Docker compose setups create bridge networks for inter-service communication

### Performance Benchmarks
Benchmark services use separate ports and databases to avoid conflicts:
- Generate 1 minute of data for 4,000 sources at 1 KHz sampling rate
- Support both regular DataColumns and SerializedDataColumns (bytes) formats
- Load balancer configuration demonstrates round-robin Envoy routing

## Development Notes

### Environment Requirements
- DP_HOME environment variable must be set for service scripts
- Docker CLI access required for containerized deployments
- Java applications use shaded JARs for self-contained deployment

### Data Generation
Sample data uses fixed timestamp starting at 2023-10-31T15:51:00.000+00:00 for consistent testing and development.

### Kubernetes Scaling Limitations & Resource Requirements

**Development Environment Constraints:**
- Minikube performance is 10-100x slower than native Java/Docker for high-throughput data ingestion
- Suitable for learning/configuration but not performance testing with accelerator data volumes
- Recommend minimum 8+ CPU cores and 16+ GB RAM for minikube with particle accelerator workloads
- Consider alternatives: Kind, Docker Desktop K8s, or cloud clusters for better performance

**Production Deployment Requirements:**
- **Memory**: Expect 1-2GB+ per ingestion service pod for accelerator data volumes  
- **CPU**: Scale based on data ingestion rate, not just CPU percentage
- **Network**: Dedicated nodes or high-bandwidth networking for streaming workloads
- **Storage**: Fast SSD storage for MongoDB with high IOPS requirements

**Provider State Sharing Issue:**
- Provider registrations are currently stored in local memory per service instance
- Multiple pod replicas cannot see providers registered in other pods, causing "invalid providerId" errors
- Duplicate key errors occur when load balancer routes retry requests to different pods
- For true horizontal scaling, provider registration state needs to be moved to shared storage (MongoDB) or distributed cache
- Current workaround: Scale to 1 replica or use session affinity for streaming clients

**Recommended Production Architecture:**
- Use sticky sessions or connection affinity for gRPC streaming clients
- Implement idempotent request handling to gracefully handle duplicate data
- Configure HPA based on data ingestion rate metrics, not just CPU/memory
- Use dedicated MongoDB replica set with sufficient resources for write-heavy workloads