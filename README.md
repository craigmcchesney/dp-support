# overview
This repo includes support for installing and managing the Data Platform and surrounding ecosystem.

The Data Platform provides tools for managing the data captured in an experimental research facility, such as a particle accelerator.  The data are used within control systems and analytics applications, and facilitate the creation of machine learning models for those applications.

The Data Platform is agnostic to the source and acquisition of the data.  A project goal is to manage data captured from the [EPICS "Experimental Physics and Industrial Control System"]([url](https://epics-controls.org/)), that use of EPICS is not required.  The Data Platform APIs are generic and can be used from essentially all programming languages and any type of application.

# data platform components
The core Data Platform contains two primary components, an Ingestion Service and a Query Service.  Each of those services provides a gRPC-based API that can be used directly to build client applications in a variety of programming languages.  Alternatively, we plan to provide higher level libraries for building client applications using languages like Java, Python, and C++.

# technology stack and ecosystem
A primary technology used building the Data Platform is the [gRPC open-source high-performance remote procedure call (RPC) framework]([url](https://grpc.io/)).  As described on [Wikipedia]([url](https://en.wikipedia.org/wiki/GRPC)), "this framework was originally developed by Google for use in connecting microservices.  It uses HTTP/2 for transport, protocol buffers as the interface description languages, and provides features such as authentication and bidirectional streaming.  It generates cross-platform client and server bindings for many languages."

# status

# todo and roadmap

# repos

# installation

## prerequisites
### mongodb
### mongoexpress

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
