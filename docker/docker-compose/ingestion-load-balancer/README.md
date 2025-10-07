## Overview

This docker-compose file demonstrates using a statically configured Envoy load balancer using a round-robin scheme for dispatching incoming gRPC ingestion requests to a pool of MLDP Ingestion Servers.  Because it is just for demonstration, the pool includes 2 statically configured BenchmarkIngestionGrpcServer instances.

## Running

Run the scenario using docker compose as illustrated below:

```
/usr/bin/docker compose -f dp-support/docker/docker-compose/mldp-ecosystem/docker-compose.yml -p mldp-ecosystem up -d
```

This starts the Envoy load balancer and two instances of the benchmark ingestion server.

To see the load balancer in action, run one of the ingestion benchmark clients against the load balancer ingestion servers.

You can run a client outside the docker environment by pointing at port 8080, e.g.,:
```
/usr/lib/jvm/java-21-openjdk-amd64/bin/java -Ddp.IngestionBenchmark.grpcConnectString=localhost:8080 com.ospreydcs.dp.service.ingest.benchmark.BenchmarkIngestDataStream
```

There is also a script for executing the ingestion benchmark client inside the docker environment against the docker load balancer configuration.  The script is in dp-support/bin:

```
app-docker-run-lb-ingestion-benchmark
```

Note that when running the client, we are overriding the default port 60051 to use port 8080, which is where the Envoy load balancer is listening.  It receives requests on port 8080 and dipatches them to the pool of ingestion servers defined in the envoy.yaml configuration file (which point to the services defined in the docker-compose.yml file via container name and port number).

## Notes

* The envoy configs must use the docker ingestion service names from docker-compose.yml in the envoy config for routing traffic to those services.  I had to remove the "container_name" attributes from the docker compose file to make this work correctly.
