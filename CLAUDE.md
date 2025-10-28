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

### Kubernetes Data Platform Ecosystem (Minikube)

**Prerequisites:**
```bash
minikube start
./docker/applications/build-dp-images.sh
```

**Deployment:**
```bash
cd kubernetes/dp-ecosystem
# Follow deployment-script.yaml for complete step-by-step deployment
kubectl apply -f .
```

**Docker Image Build:**
- Build script: `./docker/applications/build-dp-images.sh`
- Uses same Dockerfile as docker-compose configurations (`docker/applications/JavaApp/Dockerfile`)
- Creates images: `dp-ingestion-service`, `dp-query-service`, `dp-annotation-service`, `dp-ingestion-stream-service`
- All services use shared `dp-service.jar` with different main classes

**Service Architecture:**
- **Ingestion Service** (port 50051) - Data ingestion via gRPC
- **Query Service** (port 50052) - Data retrieval via gRPC  
- **Annotation Service** (port 50053) - Metadata management
- **Ingestion Stream Service** (port 50054) - Streaming ingestion
- **MongoDB** (port 27017) - Shared persistence layer

**Key Commands:**
- `kubectl get all -n dp-ecosystem` - Check all resources
- `kubectl get svc -n dp-ecosystem` - Check service endpoints
- `kubectl get pods -n dp-ecosystem` - Check pod status  
- `kubectl get hpa -n dp-ecosystem` - Check horizontal pod autoscaler
- `kubectl logs -f deployment/{service-name} -n dp-ecosystem` - Monitor logs

**External Access (Minikube):**
- MongoDB: `$(minikube ip):30017`
- Ingestion gRPC: `$(minikube ip):31406` | HTTP: `$(minikube ip):32607`
- Query gRPC: `$(minikube ip):31407` | HTTP: `$(minikube ip):32608`  
- Annotation gRPC: `$(minikube ip):31408` | HTTP: `$(minikube ip):32609`
- Ingestion Stream gRPC: `$(minikube ip):31409` | HTTP: `$(minikube ip):32610`

**Horizontal Pod Autoscaling:**
- **Ingestion**: 2-10 replicas (high-throughput workload)
- **Query**: 2-8 replicas  
- **Annotation**: 2-6 replicas (lighter workload)
- **Ingestion Stream**: 2-8 replicas (streaming workload)

**Monitoring:**
```bash
kubectl get hpa -n dp-ecosystem -w
kubectl get pods -n dp-ecosystem -w
kubectl top pods -n dp-ecosystem
```

**Manual Scaling:**
```bash
kubectl scale deployment ingestion-service --replicas=N -n dp-ecosystem
kubectl scale deployment query-service --replicas=N -n dp-ecosystem
kubectl scale deployment annotation-service --replicas=N -n dp-ecosystem
kubectl scale deployment ingestion-stream-service --replicas=N -n dp-ecosystem
```

**Cleanup:**
```bash
kubectl delete namespace dp-ecosystem
```

**Performance Expectations:**
- **Target workload**: 4000 sources × 1000 Hz = 4M samples/sec
- **Minikube results**: 11min for 1min data (single pod), 20min (dual pod)
- **Production requirement**: Real-time (1:1 ratio) with sub-second response times
- **Resource scaling**: Performance degrades in constrained environments (minikube)

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
- **Kubernetes Integration**: Same Docker images used for both docker-compose and Kubernetes deployments
  - Build script: `./docker/applications/build-dp-images.sh` creates K8s-tagged images
  - Identical base image, different tags for service identification in Kubernetes
  - All services share `dp-service.jar` with different main class entry points

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