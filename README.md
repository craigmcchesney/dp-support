# overview
This repo includes support for installing and managing the Data Platform and surrounding ecosystem.

The Data Platform provides tools for managing the data captured in an experimental research facility, such as a particle accelerator.  The data are used within control systems and analytics applications, and facilitate the creation of machine learning models for those applications.

The Data Platform is agnostic to the source and acquisition of the data.  A project goal is to manage data captured from the [EPICS "Experimental Physics and Industrial Control System"](https://epics-controls.org/), that use of EPICS is not required.  The Data Platform APIs are generic and can be used from essentially all programming languages and any type of application.

# performance
A key requirement of the Data Platform is ingesting data at rates suitable for use in an environment such as a particle accelerator.  One baseline performance goal is to ingest data from 4,000 sources sampling at a rate of 1 KHz, or 4 million samples per second.

# data platform components
The core Data Platform contains two primary components, an Ingestion Service and a Query Service.  Each of those services provides a gRPC-based API that can be used directly to build client applications in a variety of programming languages.  Alternatively, we plan to provide higher level libraries for building client applications using languages like Java, Python, and C++.

# technology stack and ecosystem
A primary technology used building the Data Platform is the [gRPC open-source high-performance remote procedure call (RPC) framework](https://grpc.io/).  As described on [Wikipedia](https://en.wikipedia.org/wiki/GRPC), "this framework was originally developed by Google for use in connecting microservices.  It uses HTTP/2 for transport, protocol buffers as the interface description languages, and provides features such as authentication and bidirectional streaming.  It generates cross-platform client and server bindings for many languages."

The other primary technology element is [MongoDB](https://www.mongodb.com/).  MongoDB is an open source document / NoSQL database management system.  Instead of using tables like a traditional relational database, it manages data in JSON-like documents.  The Ingestion and Query Services utilize MongoDB to store and retrieve data in fulfillment of client API requests.

# status
- A prototype implementation was built focusing on the creation of a general API supporting [ingestion](https://github.com/osprey-dcs/datastore) and [query](https://github.com/osprey-dcs/datastore-service) of heterogeneous data types including scalar, array / table, structure, and image.  Service implementations were created using Java for both the Ingestion and Query Services, as well as libraries for building client applications.  The prototype technology stack included both [InfluxDB](https://www.influxdata.com/) (for time series data) and MongoDB (for metadata).  This prototype did not meet the project goal for ingestion performance.
- A prototype web application was created using JavaScript React.js and using the gRPC query API.
- Performance benchmark applications were developed and executed to evaluate candidate technologies for use in the Data Platform implementation in light of the project performance goal stated above.  Benchmarks focused on gRPC for API communication; InfluxDB, MongoDB and MariaDB for database storage; and writing JSON and HDF5 files to disk.  The benchmark results showed that it was likely we could build service implementations meeting our performance requirements by using gRPC for communication and [MongoDB for storing "buckets" of time series data](https://dev.to/hpgrahsl/a-slightly-closer-look-at-mongodb-5-0-time-series-collections-part-1-32m6).
- An initial implementation of the Ingestion Service providing a gRPC API and using MongoDB for storing time-series data was built.  It is accompanied by a performance benchmark application that is used at each stage of development to measure performance relative to the project goal.  The initial implementation exceeds our goal by a comfortable margin, but this will continue to be a focus as the project evolves.


# todo and roadmap
- The next step in development is to build an initial implementation of the Query Service that provides a gRPC API and uses the MongoDB schema created by the Ingestion Service to fulfill client query requests.
- The initial implementation of the Ingestion Service supports scalar data types including float, integer, string, and boolean data.  Additional work is required to support arrays/tables, structures, and images.
- Add an API for checking the status of individual ingestion requests, and identifying problems in ingested data.
- Add an API for registering data providers.
- Build libraries for developing client applications.  This might include support for building applications with a rolling time window or retrieving data at a fixed interval.
- Build a new web version of the web application.
- The initial Ingestion Service implementation is optimized to handle data with a regular sampling interval.  We need to add support for ingestion data with irregular sampling (or do we?).
- Evaluate the requirements for updating metadata, and make changes to Ingestion Service implementation and database schema as appropriate.
- Perform load testing and address issues that arise.
- Potential directions include migration of time-series data from MongoDB to HDF5 files, storing protobuf data directly in MongoDB or data files, and horizontal scaling using an approach such as [Kubernetes](https://kubernetes.io/).

# repos
- [dp-grpc](https://github.com/osprey-dcs/dp-grpc) - Includes the gRPC API definition for the Ingestion and Query Services (in "proto" files).
- [dp-common](https://github.com/osprey-dcs/dp-common) - Includes features in common to both the Ingestion and Query Services, such as the configuration mechanism.
- [dp-ingest](https://github.com/osprey-dcs/dp-ingest) - Includes the initial implementation of the Ingestion Service, as well as the performance benchmark application.
- [dp-support](https://github.com/osprey-dcs/dp-support) - (this repo) Includes tools for installing, configuring, and managing the Data Platform ecosystem.

# installation

## prerequisites
The primary prerequisite for installing the Data Platform is installation of MongoDB.  Mongo-express is a web portal for navigating a MongoDB platform, and can be extremely useful during development and testing, and it's installation is optional.

### mongodb
MongoDB installation is required to use the Data Platform.  Installation will vary by platform and instructions for doing so should be fairly easy to find.  For installing on Ubuntu Linux 22.04 and similar platforms, I've found [these instructions](https://tecadmin.net/how-to-install-mongodb-on-ubuntu-22-04/) to be helpful.

It is also possible (and relatively simple) to run MongoDB from a docker container.  While probably not appropriate for a production installation or system under heavy load, this approach might be useful for development, evaluation, and other applications.  The [official site includes instructions for doing so](https://www.mongodb.com/compatibility/docker).

### mongoexpress

This tool is a bit quirky, and probably not appropriate for a production MongoDB installation.  There is some documentation on the [github repo](https://github.com/mongo-express/mongo-express), but I encountered some odd issues in installation, namely that I couldn't get the latest version working.  It is also necessary to tweak the config file (/usr/local/lib/node_modules/mongo-express/config.js by default) to get the installation working with your MongoDB install.

Here is a note about installing a previous version:

> Installation according to the instructions on the github readme (link 1) failed, using "npm install -g mongo-express".  (Link 2) recommended installing an older version, which seems to solve the problem. "npm i -g mongo-express@1.0.0"

Regarding configuration, if user/password authentication is enabled on MongoDB, this needs to be reflected in the mongo-express configuration.  Adding the connectionString explicitly at the top of the config.js file (including the username and password) seems to be the easiest thing to do, e.g.,

let mongo = {
  <snip>
  connectionString: 'mongodb://user:password@localhost:27017/',
  host: '127.0.0.1',
  port: '27017',
  dbName: '',
};

I also like to enable "admin" mode, which shows everything in the database.  This is done in the section shown below, by setting "admin: true":

module.exports = {
  mongodb: {
    <snip>
    // set admin to true if you want to turn on admin features
    // if admin is true, the auth list below will be ignored
    // if admin is true, you will need to enter an admin username/password below (if it is needed)
    admin: true,


## development installation
## production installation

# using the data platform

## ingestion service API
## query service API

## service configuration
### default config file
### overriding config file
### overriding individual configuration properties

## running data platform services

# appendix and technical details

## mongodb schema

## benchmark approach and results
