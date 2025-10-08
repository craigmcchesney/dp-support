## Overview

This docker-compose file runs the full MLDP ecosystem, including the mongodb server and the 4 MLDP services: ingestion, query, annotation, and ingestion stream.

## Running

Run the scenario using docker compose as illustrated below:

```
/usr/bin/docker compose -f dp-support/docker/docker-compose/mldp-ecosystem/docker-compose.yml -p mldp-ecosystem up -d
```

Once the ecosystem is running, any MLDP client can be used to interact with the services.  For example, to generate some sample data, run the TestDataGenerator.

You can run the TestDataGenerator outside the docker environment with the default settings, e.g.,
```
/usr/lib/jvm/java-21-openjdk-amd64/bin/java com.ospreydcs.dp.service.ingest.utility.TestDataGenerator
```

This connects to the ingestion service running on "localhost" port 50051.

There is also a script for running the test data generator inside the docker environment against the docker service ecosystem.  The script is in dp-support/bin:

```
app-run-docker-test-data-generator
```

Note that when running the docker client, we are overriding the ingestion service hostname to use the docker service name, e.g., "eco-ingestion-server", also on port 50051.
