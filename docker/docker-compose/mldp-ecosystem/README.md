## Overview

This docker-compose file runs the full MLDP ecosystem, including the mongodb server and the 4 MLDP services: ingestion, query, annotation, and ingestion stream.

## Running

Run the scenario using docker compose as illustrated below:

```

```

Once the ecosystem is running, any MLDP client can be used to interact with the services.  For example, to generate some sample data, run the TestDataGenerator:
```
```

Note that 

## Notes

* I had to change the configuration for the local mongodb installation (when not running mongodb from docker) to allow connection from any IP address, which I don't love but it is what is is.  This is necessary so that the ingestion service docker container IPs can connect to mongodb.  This could be solved with a static configuration / firewall etc but I didn't want to go there for this simple scenario.  I edited /etc/mongod.conf as follows:
```
bindIp: 0.0.0.0
```

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
docker run --add-host host.docker.internal:$(/sbin/ip route | awk '/default/ { print $3 }') your-java-app-image
```