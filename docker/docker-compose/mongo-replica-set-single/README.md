## Overview

This docker-compose file runs MongoDB in a single-node replica set configuration.  It is useful for testing where a minimal replica set is required.

## Running

Run the scenario using the provided shell script:

```
dp-support/docker/docker-compose/mongo-replica-set-single/start-mongo-replica-set-single.sh
```

Stop the database as shown below:

```
docker compose -f dp-support/docker/docker-compose/mongo-replica-set-single/docker-compose.yml down -v
```

## Database Connection URI

Use the following URI to connect to the database from a client application or Mongo Compass UI:

```
mongodb://admin:admin@localhost:27017/?replicaSet=rs0&authSource=admin
```

## Useful Commands

Check that the "mongo" container is running:
```
docker ps | grep mongo
```

Check docker logs for the mongo container:
```
docker logs mongo
```

Send hello command via mongo shell to running container to check topology and stats:
```
docker exec mongo mongosh -u admin -p admin --authenticationDatabase admin --eval "db.hello()"
```