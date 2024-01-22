---
title: General API
keywords: HOBBIT Documentation
sidebar: main_sidebar
toc: false
permalink: platform_api.html
folder: docs
---

The platform offers two general APIs. An inner and an outer API. The inner API can be used by the benchmark and the benchmark system and offers basic functionalities for them. The outer API summarizes the functions that can be used to integrate the platform into automated processes.

## Outer API

The outer API is a collection of APIs that go beyond the usage of the graphical user interface of the platform's or Kibana's frontend (the latter is only available, if the platform is deployed with ). These APIs might be interesting to automate the benchmarking of systems and are:
* The triple store's SPARQL API
* The platform's HTTP API

The triple store offers a SPARQL endpoint that can be used to access the platform's data. When using the [quick guide](https://hobbit-project.github.io/quick_guide.html) the store's SPARQL endpoint is available at `http://localhost:8890/sparql` for reading data. The endpoint at `http://localhost:8890/sparql-auth` can also be used to alter the data but needs authentication with the credentials that have been defined during the setup of the platform.

The platform's HTTP API is the HTTP service to which HTTP requests of the graphical user interface are sent. It comprises several methods that might be intersting to integrate the HOBBIT platform into an automated process. In the following, we assume that the platform is deployed as described in the [quick guide](https://hobbit-project.github.io/quick_guide.html), i.e., at `http://localhost:8080`. Note that we will skip the authentication that might be necessary in the following.

### Starting an experiment

The HTTP API can be used to request the run of an experiment avoiding the user interface. Such a request should be a POST request like it is sent by the following `curl` command (you can use any other HTTP client):
```
curl --data "@request.txt" -H "Content-Type: application/json" http://localhost:8080/rest/benchmarks
```
It should be noted that we assume that the platform runs on `localhost` and that it does not use a user management. For the latter, the command would have to include the user authentication. The command sends the following JSON that it loads from a `request.txt` file:
```json
{
  "benchmark":"http://w3id.org/dice-research/bbdc/ontology#Benchmark",
  "system":"http://w3id.org/dice-research/bbdc/ontology#MLFlowSystem",
  "configurationParams":[
    {
      "id":"http://w3id.org/dice-research/bbdc/ontology#modelUrl",
      "name":"Model URL",
      "description":"The URL that can be used to access the ML-Flow model that should be benchmarked.",
      "datatype":"xsd:string",
      "value":"http://example.org/some-model"
    }],
  "benchmarkName":"BBDC",
  "systemName":"System 1"
}
```
The request contains the benchmark and system IRIs, the configuration of the benchmark, i.e., a value for each configuration parameter that the benchmark defines, as well as the names of the benchmark and system.

The platform will answer with the following JSON:
```
{
  "id":"1704911859897",
  "timestamp":"2024-01-10T18:37:39.915Z"
}
```
The ID is very important, since it is used together with the namespace `http://w3id.org/hobbit/experiments#` to form the experiment's IRI.

The request above can be additionally extended by adding `maxHardwareConstraints`:
```json
{
  "benchmark":"http://w3id.org/dice-research/bbdc/ontology#Benchmark",
  "system":"http://w3id.org/dice-research/bbdc/ontology#MLFlowSystem",
  "configurationParams":[
    {
      "id":"http://w3id.org/dice-research/bbdc/ontology#modelUrl",
      "name":"Model URL",
      "description":"The URL that can be used to access the ML-Flow model that should be benchmarked.",
      "datatype":"xsd:string",
      "value":"http://example.org/some-model"
    }],
  "benchmarkName":"BBDC",
  "systemName":"System 1",
  "maxHardwareConstraints":
  {
    "iri":"http://example.org/hardware/simple-raspberry-pi",
    "label":"Raspberry Pi",
    "cpuCount":"2",
    "memory":"2147483648"
  }
}
```
The hardware constraint has the same structure as the [HardwareConstraintBean](https://github.com/hobbit-project/platform/blob/develop/hobbit-gui/gui-serverbackend/src/main/java/de/usu/research/hobbit/gui/rest/beans/HardwareConstraintBean.java) in Java. It comprises an IRI, a human readable label and two values, which can be used to restrict the hardware for the system.
1. The `cpuCount` restricts the usage of CPUs to the given number.
2. The `memory` value restricts the RAM of the system's container to the given number of bytes.

Note that:
1. If the limitations exceed the resources available to the hardware nodes, the system will be limited by the available hardware. The platform won't check this.
2. The limitation only holds for the first container of the system. Systems that comprise several containers will have less restrictions!

### Request experiment status

A call to `http://localhost:8080/rest/status` returns the status of the platform's experiment queue as JSON object. The following example shows one running experiment and a second experiment which waits in the queue.
```json
{
  "queuedExperiments":[
    {
      "experimentId":"1699551319764",
      "benchmarkUri":"http://w3id.org/dice-research/bbdc/ontology#Benchmark",
      "benchmarkName":"BBDC",
      "systemUri":"http://w3id.org/dice-research/bbdc/ontology#ExampleSystem",
      "systemName":"System 1",
      "dateOfExecution":0,
      "canBeCanceled":true
    }
],
  "runningExperiment":{
    "experimentId":"1699551319764",
    "benchmarkUri":"http://w3id.org/dice-research/bbdc/ontology#Benchmark",
    "benchmarkName":"BBDC",
    "systemUri":"http://w3id.org/dice-research/bbdc/ontology#ExampleSystem",
    "systemName":"System 1",
    "dateOfExecution":0,
    "canBeCanceled":true,
    "startTimestamp":"2023-11-09T17:35:28Z",
    "latestDateToFinish":"2023-11-09T17:55:42Z",
    "status":"Benchmark and system are initializing."
  }
}
```

## Inner API

The inner API can be used by the benchmark and the benchmarked system. Using this API, a component can request the creation of additional components, e.g., a benchmark controller can request the creation of benchmark related components or a benchmarked system can request the creation of additional containers needed by the system. Additionally, created containers can be stopped and the platform controller will send messages regarding the termination of created containers.

For the communication with the platform controller, the [command queue](https://hobbit-project.github.io/command_queue.html) is used.

### Environment variables

During the creation of docker containers, environment variables can be defined. The following variables are defined by the platform controller:
* `HOBBIT_SESSION_ID` defines the session id of the current experiment the created component is a part of.
* `HOBBIT_CONTAINER_NAME` the name of the container.
Additionally, other environment variables can be added by the component that requested the container creation.
 
If an abstract component class from the `hobbit.core` library is used, `HOBBIT_RABBIT_HOST` is added and contains the name of the RabbitMQ host which is used by the platform.

**Note** that variables with a hyphen in their name, e.g., `may-own-variable`, seem to cause problems in Dockers HTTP API. The container will be created but the variable will not be correctly set.

### Creating a container

Requesting the creation of a container is done by sending the [`Commands.DOCKER_CONTAINER_START`](https://hobbit-project.github.io/command_queue.html#predefined-command-ids) id (=`0x0C`) on the command queue, followed by a JSON string containing the necessary information. Following the structure of [command queue messages](https://hobbit-project.github.io/command_queue.html#structure-of-a-message) such a request looks like the following:

| start byte | length | meaning |
|---|---|---|
| 0 | 4 | hobbit session id length as 32 bit integer (s) |
| 4 | s | hobbit session id |
| s + 4 | 1 | `0x0C` (the Id of the `Commands.DOCKER_CONTAINER_START` command) |
| s + 5 | c | `UTF-8` encoded JSON String (with length c) |

The JSON String should have the following structure:
```json
{
  "image": "image-to-run",
  "type": "system|benchmark",
  "parent":"parent-container-name",
  "environmentVariables": [
    "key1=value1",
    "key2=value2"
  ]
}
```
Where `image-to-run` is the complete name of the image that should be run (including the repository URL), `type` contains the information whether it belongs to the benchmark or the benchmarked system and `parent-container-name` is the name of the container that sends the request (i.e., the value of the `HOBBIT_CONTAINER_NAME` environment variable). With the `environmentVariables` array, the creating container can submit additional information, e.g., parameters, to the newly created container.

**Note** that a different queue for the response ans well as a correlation ID should be mentioned in the head of the request message since the response will be sent as message to the other queue with the same correlation ID. The response is either the name of the created container as UTF-8 encoded string or an empty response if an error occured.

The Hobbit platform organizes the containers as a tree. The container that requests the creation of another container serves as parent of this newly created container. Note that a parent container should always make sure that its chield containers terminate before the parent terminates. If a parent terminates and its children are still running, the platform will force them to stop.

All created components are part of the same network. In this network, the container names serve as the host names of the containers and no port forwarding is necessary.

### Stopping and removing a container

Stopping a container can be done by sending a message to the [command queue](command-queue) containing the `Commands.DOCKER_CONTAINER_STOP` id (=`0x0D`) followed by a `UTF-8` encoded String containing the following JSON data:
```json
{
  "containerName": "container-to-stop"
}
```
where `container-to-stop` should be replaced by the container name that has been received during the creation of the container. Note that this message is won't have a direct response from the platform controller but an "indirect" response as described in the next sub section.

### Container termination

When a container terminates, the platform controller sends a broadcast message to the [command queue](command-queue) containing the container name and its exit code.

In most cases, a container exits with an exit code `0` if it was running successfully. However, please note that the exit code `137` might be caused by the termination of the container using the stop command described above, i.e., not every exit code that is not equal to `0` might indicate a problem.

#### General message structure

The message that is received in these cases has the following structure:

| start byte | length | meaning |
|---|---|---|
| 0 | 4 | `9` (length of the String `"BROADCAST"`) |
| 4 | 9 | `"BROADCAST"` |
| 13 | 1 | `0x10` (the Id of the `Commands.DOCKER_CONTAINER_TERMINATED` command) |
| 14 | 4 | length of the container name (c) |
| 18 | c | container name |
| 18 + c | 1 | exit code |

#### Receiving the message using our Abstract Java classes

If our abstract Java classes are used, receiving this messages can be eased by adding the following lines of code to the component:
```java

    public MyComponent() {
        // We have to add the broadcast command header to receive messages about
        // terminated containers
        addCommandHeaderId(Constants.HOBBIT_SESSION_ID_FOR_BROADCASTS);
    }

    @Override
    public void receiveCommand(byte command, byte[] data) {
        if (command == Commands.DOCKER_CONTAINER_TERMINATED) {
            ByteBuffer buffer = ByteBuffer.wrap(data);
            String containerName = RabbitMQUtils.readString(buffer);
            int exitCode = buffer.get();
            // check whether it is one of your containers and react accordingly
        }
        super.receiveCommand(command, data);
    }
```

