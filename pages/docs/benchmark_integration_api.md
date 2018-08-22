---
title: General Benchmark Integration API
keywords: HOBBIT Documentation
sidebar: main_sidebar
toc: false
permalink: benchmark_integration_api.html
folder: docs
---

This article describes the general API the platform offers for a benchmark. Note that in addition to this API, the benchmark might define an own API for the communication with the system.

The API of the platform for a benchmark comprises three parts.
1. There are some details that should be taken into account when defining the benchmark container image.
2. The platform controller offers some functionalities which can be used by the benchmark to create new containers and stop them.
3. Every benchmark needs to be described in a meta data file which gives the platform the necessary information about the benchmark.

## Benchmark controller container

There is one main container that will represent the benchmark (from the platforms point of view). Although the benchmark can comprise several containers, the platform will observe this main container and react on its termination - regardless whether other containers still exist or not.

### Container start

A benchmark controller might comprise more than one container. However, it needs to have exactly __one main container__. This container will be created by the platform controller including the following environmental variables:

|- Variable name -|- Meaning -|
|---|---|
| `HOBBIT_RABBIT_HOST` | The DNS name of the RabbitMQ instance which is used for communicating to the platform controller, e.g, via the [command queue](command_queue). |
| `HOBBIT_SESSION_ID` | The ID of the current experiment. It is necessary for the usage of the [command queue](command_queue). |
| `BENCHMARK_PARAMETERS_MODEL_KEY` | The benchmark model from the meta data file serialized as `JSON-LD` string. |
| `HOBBIT_EXPERIMENT_URI` | The URI of the experiment which should be used to generate the result model. |

After the container started, the platform assumes that it will connect itself to the command queue.

#### Benchmark parameters

As described below, a benchmark can define parameters which can be configured before starting an experiment. These parameters are submitted to the benchmark controller container as [RDF](https://www.w3.org/2001/sw/wiki/RDF) model within the `BENCHMARK_PARAMETERS_MODEL_KEY` environmental variable. For reading the parameters, it is necessary to understand the content of the model.

The parameters will be defined as properties in the benchmark meta data file which is described below. Therefore, every parameter has an own URI and a defined value range. The parameter model contains triples like
```turtle
<http://w3id.org/hobbit/experiments#New> <parameter-URI> "parameter value"^^<parameter-value-datatype-URI> .
```
(Note that for simplicity, we have chosen the more human-readable `turtle` serialization for this example. However, the parameter model will be serialized in `JSON-LD`. It is recommended to use an RDF library which handles the (de-)serialization and eases the development.)

Note that for enumerations, i.e., for parameters that have a fixed set of possible values, these values will be given as URIs as in the following example:
```turtle
<http://w3id.org/hobbit/experiments#New> <parameter-URI> <value-URI> .
```

### Communication

The communication between the benchmark controller and the platform is mainly based on the  [command queue](/command_queue.html) and predefined command IDs.

#### Benchmark initialization

The platform expects the benchmark controller to send the [`BENCHMARK_READY_SIGNAL`](https://hobbit-project.github.io/command_queue.html#predefined-command-ids) via the command queue after the benchmark is initialized. Note that the benchmarking is not started as long as this message has not been received.

#### Benchmark start

Note that the benchmark will have to wait for the platform controller to send the [`START_BENCHMARK_SIGNAL`](https://hobbit-project.github.io/command_queue.html#predefined-command-ids) via the command queue. Only after receiving this signal, the benchmark controller is allowed to start benchmarking the system. Note that this message contains the container name of the systems main container, i.e., from that point in time on, the benchmark itself is responsible to react if the system terminates (see the [container termination](https://hobbit-project.github.io/platform_api.html#container-termination) section for information about receiving the information about a terminated container). The start message has the following structure:

| start byte | length | meaning |
|---|---|---|
| 0 | 4 | hobbit session id length as 32 bit integer (s) |
| 4 | s | hobbit session id |
| s + 4 | 1 | `0x11` (the Id of the `Commands.START_BENCHMARK_SIGNAL` command) |
| s + 5 | c | `UTF-8` encoded String containing the container name of the system container (with length c) |

Note that since the benchmark and system containers are within the same virtual network, the systems container name can be used to directly communicate with it, e.g., via HTTP (note that this would be part of the benchmark API and has to be negotiated between the benchmark and the system developer).

#### Benchmark result

After benchmarking the system, the benchmark is expected to create an RDF model containing the results of the benchmarking. The triples are expected to have the following structure
```turtle
<experiment-URI> <kpi-property-URI> "result literal"^^<result-datatype-URI> .
```
(Note that for simplicity, we have chosen the more human-readable `turtle` serialization for this example. However, the result model will be serialized in `JSON-LD`. It is recommended to use an RDF library which handles the serialization and eases the development.)

The experiment URI is available as environmental variable. The URI of the KPI the benchmark is measuring as well as their datatypes are defined in the benchmarks meta data file. See the section for the file below, for details.

The result model should be serialized as `JSON-LD` String and submitted to the platform via the command queue using the following message:

| start byte | length | meaning |
|---|---|---|
| 0 | 4 | hobbit session id length as 32 bit integer (s) |
| 4 | s | hobbit session id |
| s + 4 | 1 | `0x0B` (the Id of the `Commands.BENCHMARK_FINISHED_SIGNAL` command) |
| s + 5 | c | `UTF-8` encoded String containing the result model serialized as `JSON-LD` (with length c) |

### Container termination

The benchmark controller container is observed by the platform. If it terminates, the platform is checking its exit code. Note that in most cases, an exit code `== 0` means that a benchmark terminated as expected while a different exit code will be interpreted as a crashing benchmark leading to an abortion of the experiment and storing an error in the result database.

## Platform controller API

The platform controller offers an API to the system which offers
* the creation of new containers and
* the termination of previously started containers.

The API is described [here](https://hobbit-project.github.io/platform_api.html) in detail.

## Benchmark meta data file

The benchmark meta data file comprises meta data about the uploaded benchmark that is needed by the Hobbit platform. 
The file contains the meta data as [RDF triples](https://www.w3.org/2001/sw/wiki/RDF) in the 
[Turtle format](https://www.w3.org/TR/turtle/) and needs to be uploaded to the git instance of the platform, e.g., into the root directory of one of the projects that host one of the benchmark components, e.g., the benchmark controller. This section gives guidance how this meta data file can be created for a benchmark called `"MyBenchmark"`.

First, we have to collect the necessary data:
* The benchmark needs a unique identifier, i.e., a URI. In our example, we choose `http://www.example.org/exampleBenchmark/MyOwnBenchmark`. 
* The benchmark needs a name (`"MyBenchmark"`) and a short description (`"This is my own example benchmark..."`).
* The name of the uploaded benchmark controller Docker image is needed (`"git.project-hobbit.eu:4567/maxpower/mybenchmarkcontroller"`).
* The URI of the benchmark API.
* The version number of the benchmark implementation.
* The parameters of the benchmark, their ranges and their default values.
* The KPIs measured by the benchmark and their ranges.

Our example file has the following content

```turtle
@prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#> .
@prefix hobbit: <http://w3id.org/hobbit/vocab#> .
@prefix xsd: <http://www.w3.org/2001/XMLSchema#> .

<http://www.example.org/exampleBenchmark/MyOwnBenchmark> a hobbit:Benchmark;
	rdfs:label "MyBenchmark"@en;
	rdfs:comment	"This is my own example benchmark..."@en;
	hobbit:imageName	"git.project-hobbit.eu:4567/maxpower/mybenchmarkcontroller";
	hobbit:version "v1.1"@en;
	hobbit:measuresKPI <http://www.example.org/exampleBenchmark/precision> ;
	hobbit:measuresKPI <http://www.example.org/exampleBenchmark/recall> ;
	hobbit:measuresKPI <http://www.example.org/exampleBenchmark/fmeasure> ;
	hobbit:hasAPI	<http://www.example.org/exampleBenchmark/API>;
	hobbit:hasParameter <http://www.example.org/exampleBenchmark/size>;
	hobbit:hasParameter <http://www.example.org/exampleBenchmark/queryScenario>;
	hobbit:hasParameter <http://www.example.org/exampleBenchmark/amountOfNullExamples> .
```
It can be seen that apart from the label, the description, a version and the image name our benchmark has an API, several KPIs and parameters assigned to it which will be explained in the following.

**Note** that a single `benchmark.ttl` file can contain multiple benchmarks.

### API
Inside the meta data files, the API is a URI that works as a simple identifier to be able to map systems to benchmarks. A benchmark can only be executed together with a system if both share the same API.

```turtle
<http://www.example.org/exampleBenchmark/API> a hobbit:API .
```

### KPIs
A Key Performance Indicator (KPI) is a value that shows the performance of the benchmarked system. It should have a label, a description and a range. In our example we have the three KPIs precision, recall and F-measure.

```turtle
<http://www.example.org/exampleBenchmark/precision> a hobbit:KPI;
	rdfs:label "Precision"@en;
	rdfs:comment "Precision = TP / (TP + FP)"@en;
	rdfs:range xsd:float .

<http://www.example.org/exampleBenchmark/recall> a hobbit:KPI;
	rdfs:label "Recall"@en;
	rdfs:comment "Recall = TP / (TP + FN)"@en;
	rdfs:range xsd:float .

<http://www.example.org/exampleBenchmark/fmeasure> a hobbit:KPI;
	rdfs:label "F-measure"@en;
	rdfs:comment "F-measure is the harmonic mean of precision and recall."@en;
	rdfs:range xsd:float .
```

### Parameters
There are two (overlapping) groups of parameters. configurable parameters can be configured by the user when starting a benchmark. Feature parameters are used for deeper analysis.

Every parameter should have a label, a description and a value range. Configurable parameters should have a default value that can be used by non-expert users to get the benchmark running.

```turtle
<http://www.example.org/exampleBenchmark/size> a hobbit:ConfigurableParameter, hobbit:FeatureParameter;
	rdfs:label "Dataset size"@en;
	rdfs:comment "The size of the generated dataset counted in triples"@en;
	rdfs:range xsd:unsignedInt;
        hobbit:defaultValue "10000"^^xsd:unsignedInt .
```

Apart from parameters with simple values, HOBBIT supports parameters that you have to choose from a given set of values.
To use this feature, the parameter needs to have a class as `rdfs:range` and the single choosable values need to be instances of that class.
```turtle
<http://www.example.org/exampleBenchmark/graphPattern> a hobbit:ConfigurableParameter, hobbit:FeatureParameter;
	rdfs:label "Graph pattern"@en;
	rdfs:comment "The graph pattern used to generate the test data"@en;
	rdfs:range <http://www.example.org/exampleBenchmark/GraphPattern>;
        hobbit:defaultValue <http://www.example.org/exampleBenchmark/Star> .

<http://www.example.org/exampleBenchmark/GraphPattern> a rdfs:Class, owl:Class .

<http://www.example.org/exampleBenchmark/Star> a <http://www.example.org/exampleBenchmark/GraphPattern>;
	rdfs:label "Star graph pattern"@en;
	rdfs:comment "The graph has a central node and all other nodes have only one connection to this central node"@en .

<http://www.example.org/exampleBenchmark/Grid> a <http://www.example.org/exampleBenchmark/GraphPattern>;
	rdfs:label "Grid graph pattern"@en;
	rdfs:comment "A 2-dimensional grid of nodes"@en .

<http://www.example.org/exampleBenchmark/Clique> a <http://www.example.org/exampleBenchmark/GraphPattern>;
	rdfs:label "Clique graph pattern"@en;
	rdfs:comment "A fully connected graph"@en .
```
In the example, the benchmark offers different graph patterns from which the user will have to choose one.

In some cases, the benchmark might define a parameter that shouldn't be set by the user but is still interesting for later analysis, e.g., if the benchmark is based on data that is generated on the fly.
This can be achieved by defining a parameter only as `hobbit:FeatureParameter` and not as configurable.
```turtle
<http://www.example.org/exampleBenchmark/amountOfBlanknodes> a hobbit:FeatureParameter;
	rdfs:label "Amount of blank nodes"@en;
	rdfs:comment "The amount of blank nodes that have been created during the graph generation."@en;
	rdfs:range xsd:float .
```
In our example, the benchmark would generate a graph for benchmarking a system. After the generation and the evaluation of the systems results, the benchmark could add the amount of blank nodes that have been created to the result. Note that this is not a KPI since it is not bound to the performance of the system.

### The complete example

In the following, you can find the complete example described in the section above.

```turtle
@prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#> .
@prefix hobbit: <http://w3id.org/hobbit/vocab#> .
@prefix xsd: <http://www.w3.org/2001/XMLSchema#> .

<http://www.example.org/exampleBenchmark/MyOwnBenchmark> a hobbit:Benchmark;
	rdfs:label "MyBenchmark"@en;
	rdfs:comment	"This is my own example benchmark..."@en;
	hobbit:imageName	"git.project-hobbit.eu:4567/maxpower/mybenchmarkcontroller";
	hobbit:version "v1.1"@en;
	hobbit:measuresKPI <http://www.example.org/exampleBenchmark/precision> ;
	hobbit:measuresKPI <http://www.example.org/exampleBenchmark/recall> ;
	hobbit:measuresKPI <http://www.example.org/exampleBenchmark/fmeasure> ;
	hobbit:hasAPI	<http://www.example.org/exampleBenchmark/API>;
	hobbit:hasParameter <http://www.example.org/exampleBenchmark/size>;
	hobbit:hasParameter <http://www.example.org/exampleBenchmark/queryScenario>;
	hobbit:hasParameter <http://www.example.org/exampleBenchmark/amountOfNullExamples> .

<http://www.example.org/exampleBenchmark/API> a hobbit:API .

<http://www.example.org/exampleBenchmark/precision> a hobbit:KPI;
	rdfs:label "Precision"@en;
	rdfs:comment "Precision = TP / (TP + FP)"@en;
	rdfs:range xsd:float .

<http://www.example.org/exampleBenchmark/recall> a hobbit:KPI;
	rdfs:label "Recall"@en;
	rdfs:comment "Recall = TP / (TP + FN)"@en;
	rdfs:range xsd:float .

<http://www.example.org/exampleBenchmark/fmeasure> a hobbit:KPI;
	rdfs:label "F-measure"@en;
	rdfs:comment "F-measure is the harmonic mean of precision and recall."@en;
	rdfs:range xsd:float .

<http://www.example.org/exampleBenchmark/size> a hobbit:ConfigurableParameter, hobbit:FeatureParameter;
	rdfs:label "Dataset size"@en;
	rdfs:comment "The size of the generated dataset counted in triples"@en;
	rdfs:range xsd:unsignedInt;
        hobbit:defaultValue "10000"^^xsd:unsignedInt .

<http://www.example.org/exampleBenchmark/graphPattern> a hobbit:ConfigurableParameter, hobbit:FeatureParameter;
	rdfs:label "Graph pattern"@en;
	rdfs:comment "The graph pattern used to generate the test data"@en;
	rdfs:range <http://www.example.org/exampleBenchmark/GraphPattern>;
        hobbit:defaultValue <http://www.example.org/exampleBenchmark/Star> .

<http://www.example.org/exampleBenchmark/GraphPattern> a rdfs:Class, owl:Class .

<http://www.example.org/exampleBenchmark/Star> a <http://www.example.org/exampleBenchmark/GraphPattern>;
	rdfs:label "Star graph pattern"@en;
	rdfs:comment "The graph has a central node and all other nodes have only one connection to this central node"@en .

<http://www.example.org/exampleBenchmark/Star> a <http://www.example.org/exampleBenchmark/GraphPattern>;
	rdfs:label "Grid graph pattern"@en;
	rdfs:comment "A 2-dimensional grid of nodes"@en .

<http://www.example.org/exampleBenchmark/Star> a <http://www.example.org/exampleBenchmark/GraphPattern>;
	rdfs:label "Clique graph pattern"@en;
	rdfs:comment "A fully connected graph"@en .

<http://www.example.org/exampleBenchmark/amountOfBlanknodes> a hobbit:FeatureParameter;
	rdfs:label "Amount of blank nodes"@en;
	rdfs:comment "The amount of blank nodes that have been created during the graph generation."@en;
	rdfs:range xsd:float .
```
