---
title: Integrating Your Own Code
keywords: HOBBIT Documentation
sidebar: main_sidebar
toc: false
permalink: system_integration.html
folder: docs
---

This tutorial shows how a system can be prepared to be benchmarked within the HOBBIT platform. The tutorial mainly covers all necessary steps including the development of a System Adapter in Java. Note that the adapter can be implemented in other languages as well. However, for Java we are offering a library which eases the implementation.

A system that should be benchmark within the HOBBIT platform needs to fulfil the following requirements:
1. Be encapsulated in a Docker image
1. Implement the API of the benchmark. This is something that can not be covered by this tutorial since it is depending on the benchmark.
1. Implement the API of the platform. This mainly includes
    1. Sending a message to the platform controller that your system is ready to be benchmarked your container has started,
    1. Sending requests and handling the responses correctly if your system does not comprise only one container but needs more than one.

For the requirements 2 and 3, a System Adapter can be developed which takes care of that. So your system does not have to be changed just to be benchmarked.

In this tutorial, we will present two scenarios for that. First, the system adapter will be implemented as a wrapper which runs together with the benchmarked system in the same container. The second approach is a separated System Adapter which runs in another container in parallel to the benchmarked system.

## 1. Set up the System Adapter project
TODO

## 2. Writing the System Adapter
### 2.a. The System Adapter as wrapper
TODO

### 2.b. The System Adapter separated from the system
TODO

## 3. Writing the system.ttl file

The system meta data file comprises meta data about the uploaded system that is needed by the Hobbit platform.
The file contains the meta data as [RDF triples](https://www.w3.org/2001/sw/wiki/RDF) in the
[Turtle format](https://www.w3.org/TR/turtle/) and needs to be uploaded to the git instance of the platform, e.g., into the root directory of the project which hosts the system Docker image. This article gives guidance how this meta
data file can be created for a system called `"MyOwnSystem"`.

### Simple file

The simplest file defines only those information that are necessary to get the system running. First, we have to collect the necessary data:
* The system needs a unique identifier, i.e., a URI. In our example, we choose `http://www.example.org/exampleSystem/MyOwnSystem`.
* The system needs a name (`"MyOwnSystem"`) and a short description (`"This is my own example system..."`).
* The name of the uploaded docker image is needed (`"git.project-hobbit.eu:4567/maxpower/mysystem"`).
* The URI of the benchmark API, the system implements (`http://benchmark.org/MyNewBenchmark/BenchmarkApi`, should be provided by the benchmark description page).

Our example file has the following content

```turtle
@prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#> .
@prefix hobbit: <http://w3id.org/hobbit/vocab#> .

<http://www.example.org/exampleSystem/MyOwnSystem> a  hobbit:SystemInstance;
	rdfs:label	"MyOwnSystem"@en;
	rdfs:comment	"This is my own system defined in a simple way"@en;
	hobbit:imageName "git.project-hobbit.eu:4567/maxpower/mysystem";
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
    hobbit:imageName "git.project-hobbit.eu:4567/maxpower/mysystem";
    hobbit:implementsAPI <http://benchmark.org/MyNewBenchmark/BenchmarkApi> ;
    hobbit:instanceOf <http://www.example.org/exampleSystem/System> ;
    <http://www.example.org/exampleSystem/System#threshold> "0.7"^^xsd:float .

<http://www.example.org/exampleSystem/MyOwnSystem2> a  hobbit:SystemInstance;
    rdfs:label  "MySystem (0.6)"@en;
    rdfs:comment    "This is my own system with a threshould of 0.6"@en;
    hobbit:imageName "git.project-hobbit.eu:4567/maxpower/mysystem";
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
