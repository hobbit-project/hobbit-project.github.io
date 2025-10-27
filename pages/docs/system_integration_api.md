---
title: Integrating a System
keywords: HOBBIT Documentation
sidebar: main_sidebar
toc: false
permalink: system_integration_api.html
folder: docs
---

This article describes the general API the platform offers for a benchmarked system. Note that in addition to this API, the benchmark might define an own API for the communication with the system.

The API of the platform for benchmarked systems comprises three parts.
1. There are some details that should be taken into account when defining the system container image.
2. The platform controller offers some functionalities which can be used by the system to create new containers and stop them.
3. Every system needs to be described in a meta data file which gives the platform the necessary information about the system.

## System container

There is one main container that will represent the complete system (from the platforms point of view). Although the system can comprise several containers, the platform will observe this main container and react on its termination - regardless whether other system containers still exist or not.

### Container start

A benchmarked system might comprise more than one container. However, it needs to have exactly __one main container__. This container will be created by the platform controller including the following environmental variables:

| Variable name | Meaning |
|---|---|
| `HOBBIT_RABBIT_HOST` | The DNS name of the RabbitMQ instance which is used for communicating to the platform controller, e.g, via the [command queue](command_queue). |
| `HOBBIT_SESSION_ID` | The ID of the current experiment. It is necessary for the usage of the [command queue](command_queue). |
| `SYSTEM_PARAMETERS_MODEL` | The system instance model from the meta data file serialized as `JSON-LD` string. |

After the container started, the platform assumes that it will connect itself to the command queue.

### Communication

The platform expects the system container to send the [`SYSTEM_READY_SIGNAL`](https://hobbit-project.github.io/command_queue.html#predefined-command-ids) via the command queue after the system is initialized. Note that the benchmarking is not started as long as this message has not been received. It should also be mentioned that from that moment on, the benchmark is allowed to bombard the system with messages. So the system should be fully up and running when sending this message.

#### Benchmark API Communication

Apart from that, the system container will have to implement the benchmark API. If the benchmark is relying on [our proposed workflow](https://hobbit-project.github.io/experiment_workflow.html), the system adapter should listen to the command queue for the [`TASK_GENERATION_FINISHED`](https://hobbit-project.github.io/command_queue.html#predefined-command-ids) message. If this message is received, the benchmark informs the system that there won't be any new tasks generated. They system should process all remaining tasks and terminate.

### Container termination

The systems main container is observed by the platform. If it terminates, the platform is submitting its exit code to the benchmark. Note that in most cases, an exit code `== 0` means that a system terminated as expected while a different exit code can be interpreted as a crashing system. The exact handling of this case is left to the benchmark.

If the system terminates _before_ during its initialization (or more precise: before the platform started the benchmark) the experiment will be aborted with an error code stating that the system crashed.

## Platform controller API

The platform controller offers an API to the system which offers
* the creation of new containers and
* the termination of previously started containers.

The API is described [here](https://hobbit-project.github.io/platform_api.html) in detail.

## System meta data file

The system meta data file comprises meta data about the uploaded system that is needed by the Hobbit platform.
The file contains the meta data as [RDF triples](https://www.w3.org/2001/sw/wiki/RDF) in the
[Turtle format](https://www.w3.org/TR/turtle/) and needs to be uploaded to the git instance of the platform, e.g., into the root directory of the project which hosts the system Docker image. This article gives guidance how this meta data file can be created for a system called `"MyOwnSystem"`.

### Simple file

The simplest file defines only the information necessary to get the system running. First, we have to collect the necessary data:
* The system needs a __unique__ identifier, i.e., a URI. In our example, we choose `http://www.example.org/exampleSystem/MyOwnSystem`.
* The system needs a name (`"MyOwnSystem"`) and a short description (`"This is my own example system..."`).
* The name of the uploaded docker image is needed (`"git.project-hobbit.eu:4567/maxpower/mysystemadapter"`).
* The URI of the benchmark API, the system implements (`http://benchmark.org/MyNewBenchmark/BenchmarkApi`, should be provided by the benchmark description page).

Our example file has the following content

```turtle
@prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#> .
@prefix hobbit: <http://w3id.org/hobbit/vocab#> .

<http://www.example.org/exampleSystem/MyOwnSystem> a  hobbit:SystemInstance;
	rdfs:label	"MyOwnSystem"@en;
	rdfs:comment	"This is my own system defined in a simple way"@en;
	hobbit:imageName "git.project-hobbit.eu:4567/maxpower/mysystemadapter";
	hobbit:implementsAPI <http://benchmark.org/MyNewBenchmark/BenchmarkApi> .
```

### Using parameters for a system

Some systems offer one or more parameters which can be used to adapt the system for certain scenarios.
The HOBBIT platform can analyse the influence of a parameter on the systems performance using its analysis component.
To enable this analysis, it is necessary to define the parameters as well as the single parameterizations of a system in the meta data file.
To this end, a `hobbit:System` has to be defined that is connected to `hobbit:FeatureParameter` instances.
The single parameterised versions of the system are defined as `hobbit:SystemInstance` objects with a single value for each parameter.

```turtle
@prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#> .
@prefix hobbit: <http://w3id.org/hobbit/vocab#> .
@prefix xsd: <http://www.w3.org/2001/XMLSchema#> .

<http://www.example.org/exampleSystem/System> a hobbit:System ;
    rdfs:label "MySystem"@en;
    rdfs:comment "This is my own system defined in a simple way"@en;
    hobbit:hasParameter <http://www.example.org/exampleSystem/System#threshold> .

<http://www.example.org/exampleSystem/System#threshold> a hobbit:FeatureParameter ;
    rdfs:label "Threshold"@en ;
    rdfs:comment "A threshold parameter for our System."@en ;
    rdfs:range  xsd:float .

<http://www.example.org/exampleSystem/MyOwnSystem> a  hobbit:SystemInstance;
    rdfs:label  "MySystem (0.7)"@en;
    rdfs:comment    "This is my own system with a threshould of 0.7"@en;
    hobbit:imageName "git.project-hobbit.eu:4567/maxpower/mysystemadapter";
    hobbit:implementsAPI <http://benchmark.org/MyNewBenchmark/BenchmarkApi> ;
    hobbit:instanceOf <http://www.example.org/exampleSystem/System> ;
    <http://www.example.org/exampleSystem/System#threshold> "0.7"^^xsd:float .

<http://www.example.org/exampleSystem/MyOwnSystem2> a  hobbit:SystemInstance;
    rdfs:label  "MySystem (0.6)"@en;
    rdfs:comment    "This is my own system with a threshould of 0.6"@en;
    hobbit:imageName "git.project-hobbit.eu:4567/maxpower/mysystemadapter";
    hobbit:implementsAPI <http://benchmark.org/MyNewBenchmark/BenchmarkApi> ;
    hobbit:instanceOf <http://www.example.org/exampleSystem/System> ;
    <http://www.example.org/exampleSystem/System#threshold> "0.6"^^xsd:float .
```

In the example, `"MySystem"` is defined as a `hobbit:System` with a threshold parameter attached to it using the `hobbit:hasParameter` property.
The threshould parameter is an instance of `hobbit:FeatureParameter`, has a label as well as a description and defines the range of its values as floating point number.
The two instances `"MySystem (0.6)"` and `"MySystem (0.7)"` are connected to the system definition using the `hobbit:instanceOf` property and define a certain value for the threshold parameter.
This example results in two system instances listed as systems for benchmarking in the platforms front end and the knowledge about the threshold as a feature that can be used in the analysis component.

To ease the usage of parameters, the meta data of a system instance is given to its Docker container when it is started, i.e., the user does not have to define a single Docker image for every parameterization but can read the parameters from an environmental variable at runtime.

**Note** that a single system.ttl file can contain multiple systems and system instances.

## Integration with ENEXA

Modules of the [ENEXA project](https://enexa.eu/) do not need any adaptation. The HOBBIT platform implements the [ENEXA service](https://enexa.eu/documentation/service_des.html) and, hence, supports the interactions that the ENEXA module may expect. 

Most benchmark implementations won't be aware that they interact with an ENEXA module. Hence, we recommend to implement a light-weight system adapter that translates between requests of the benchmark and the ENEXA module. An example can be found [here](https://github.com/EnexaProject/tentris-odin-hobbit-benchmark) where the system adapter creates the evaluated ENEXA module using the typical ENEXA service methods and forwards requests and responses between the benchmark and the module.
