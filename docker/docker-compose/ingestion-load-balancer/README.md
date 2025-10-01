## Overview

This docker-compose file demonstrates using a statically configured Envoy load balancer using a round-robin scheme for dispatching incoming gRPC ingestion requests to a pool of Ingestion Servers.  Because it is just for demonstration, the pool includes 2 statically configured BenchmarkIngestionGrpcServer instances.

## Running

Run the scenario using docker compose as illustrated below:

```
/usr/bin/docker compose -f /home/craigmcc/dp.fork/dp-support/docker/docker-compose/mldp-ecosystem/docker-compose.yml -p mldp-ecosystem up -d
```

This starts the Envoy load balancer and two instances of the benchmark ingestion server.

To see the load balancer in action, run one of the MLDP benchmark clients, e.g.,:

```
/usr/lib/jvm/java-21-openjdk-amd64/bin/java -Ddp.IngestionBenchmark.grpcConnectString=localhost:8080 com.ospreydcs.dp.service.ingest.benchmark.BenchmarkIngestDataStream
```

Note that when running the client, we are overriding the default port 60051 to use port 8080, which is where the Envoy load balancer is listening.  It receives requests on port 8080 and dipatches them to the pool of ingestion servers defined in the envoy.yaml configuration file (which point to the services defined in the docker-compose.yml file via container name and port number).

## Notes

* I had to change the configuration for the local mongodb installation (when not running mongodb from docker) to allow connection from any IP address, which I don't love but it is what is is.  This is necessary so that the ingestion service docker container IPs can connect to mongodb.  This could be solved with a static configuration / firewall etc but I didn't want to go there for this simple scenario.
* I also had to change the ingestion service configuration to use the explicit host IP (-Ddp.MongoClient.dbHost=10.0.1.60) instead of "localhost", which also doesn't scale in a general way.  On some platforms, you can use the special DNS host name "host.docker.internal", but this seems to work on Mac and Windows (platforms with docker desktop) but not on ubuntu.  I also tried using "extra_hosts" in the config, but that also didn't work for me.  I obviously need to investigate further.

```
services:
          your-java-app:
            # ... other configurations
            extra_hosts:
              - "host.docker.internal:$(/sbin/ip route | awk '/default/ { print $3 }')"
            # ...
```
* I also tried using a command like the following to dynamically get the host IP and map it, but I also couldn't seem to make that work.  FWIW, this seems like a good solution, but not sure how it would hold up on different platforms (e.g., depends on /sbin/ip):
```
services:
          your-java-app:
            # ... other configurations
            extra_hosts:
              - "host.docker.internal:$(/sbin/ip route | awk '/default/ { print $3 }')"
            # ...
```