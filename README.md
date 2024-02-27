# Data Platform Overview

This repo is part of the Data Platform project.  The Data Platform consists of services for capturing and providing access to data captured from a particle accelerator facility.  The [data-platform repo](https://github.com/osprey-dcs/data-platform) provides a project overview and links to the various project componnents, as well as an installer for running the latest version.

The service implementations are contained in the [dp-service](https://github.com/osprey-dcs/dp-service) repo, which includes both user-oriented and developer-oriented documentation about the services.

The service APIs are built using [gRPC](https://grpc.io/docs/what-is-grpc/introduction/) for both interface definition and message interchange.  The latest gRPC proto files for the service APIs are contained and documented in the [dp-grpc repo](https://github.com/osprey-dcs/dp-grpc).  Using gRPC, client applications can be built to interact with the Data Platform services using practically any programming language.

The [dp-web-app repo](https://github.com/osprey-dcs/dp-web-app) contains a JavaScript web application that utilizes the Data Platform Query Service to navigate the archive.

This repo, [dp-support](https://github.com/osprey-dcs/dp-support) includes a set of utilities for managing the processes comprising the Data Platform ecosystem.

# dp-support

The Data Platform ecosystem consists of the following components:

- the service implementations in the dp-service repo
- MongoDB, used for persistence by the Data Platform
- (optionally) An Envoy proxy, for sending http1 traffic from the Data Platform web application to the http2-based gRPC service implementations

The dp-support repo includes tools for managing each of these components.  Use of the tools is optional.  They can be used as is (by downloading the installer from the data-platform repo), or can be treated as a starting place for a custom installation.  

The utilities for each component are described in more detail below.

## Quick Start

The data-platform repo includes a [Quick Start section](https://github.com/osprey-dcs/data-platform?tab=readme-ov-file#data-platform-quick-start) for quickly getting the Data Platform up and running, with example usage of the scripts in dp-support to manage the ecosystem.

## MongoDB scripts

MongoDB is used for Data Platform persistence.  The current reference version is 7.0.5.  For details about installing MongoDB as a locally installed package or as a docker container deployment, see the [data-platform repo documentation](https://github.com/osprey-dcs/data-platform).  The tools for managing MongoDB include:

### local MongoDB installation

One set of scripts is used for managing a Linux MongoDB installation using the "systemctl" utility.  These are simple wrapper scripts:

- _mongodb-systemctl-start_: Starts standard MongoDB installation using systemctl.
- _mongodb-systemctl-stop_: Stops MongoDB.
- _mongodb-systemctl-status_: Checks MongoDB status.
- _mongodb-systemctl-enable_: Enables MongoDB auto-start after reboot.
- _mongodb-systemctl-disable_: Disables MongoDB auto-start after reboot.

### Docker MongoDB deployment

The second set of scripts is for managing MongoDB deployed as a Docker container.  The following scripts are included:

- _mongodb-docker-create_: Creates MongoDB Docker container.
- _mongodb-docker-start_: Starts the MongoDB Docker container.
- _mongodb-docker-stop_: Stops the MongoDB Docker container.
- _mongodb-docker-remove_: Removes the MongoDB Docker container.  BEWARE THIS WILL DELETE ALL DATA IN THE DATABASE.
- _mongodb-docker-shell_: Runs mongosh shell against the (running) MongoDB Docker container.

### MongoDB Compass GUI application

Compass is a GUI application for navigating a MongoDB database.  This repo contains a simple wrapper script for starting the application, passing the default MongoDB connection string (MongoDB user/password = "admin") on start up:

```
mongodb://admin:admin@localhost:27017
```

This script will need to be modified as appropriate if the user/password are customized.

- _mongodb-compass-start_: Starts the MongoDB Compass application, passing the default connection string on the command line.

## Data platform server scripts

The bin directory contains scripts for managing the Data Platform server applications, including the Ingestion and Query Services.  These scripts use a set of lower level scripts in the same directory for starting/stopping processes and checking their status: _util-pm-start_, _util-pm-stop_, and _util-pm-status_, respectively.

### Ingestion service scripts

- _server-ingest-start_: Starts the ingestion server application using the util-pm-start script.
- _server-ingest-stop_: Stops the running ingestion server application using the util-pm-stop script.
- _server-ingest-status_: Checks the status of the ingestion server application using util-pm-status.

### Query service scripts

- _server-query-start_: Starts the query server application using the util-pm-start script.
- _server-query-stop_: Stops the running query server application using the util-pm-stop script.
- _server-query-status_: Checks the status of the query server application using util-pm-status.

## Data platform performance benchmarks

The bin directory contains two scripts for running the Data Platform performance benchmarks, which might be useful for testing a new installation.

- _app-run-ingestion-benchmark_: Runs the data platform ingestion service performance benchmark application against the running ingestion server.  Displays an error if the server is not running.  Uploads one minute's data for 4,000 data sources sampled at 1 KHz.
- _app-run-query-benchmark_: Runs the query service performance benchmark application using the server-streaming time-series data query API.  Loads a set of data and exercises the query mechanism against it.
