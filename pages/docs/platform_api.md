---
title: General API
keywords: HOBBIT Documentation
sidebar: main_sidebar
toc: false
permalink: platform_api
folder: docs
---

The platform offers a general API for the benchmark and the benchmarked system. Using this API, a component can request the creation of additional components, e.g., a benchmark controller can request the creation of benchmark related components or a benchmarked system can request the creation of additional containers needed by the system. Additionally, created containers can be stopped and the platform controller will send messages regarding the termination of created containers.

For the communication with the platform controller, the [command queue](/command_queue) is used.

## Environment variables

During the creation of docker containers, environment variables can be defined. The following variables are defined by the platform controller:
* `HOBBIT_SESSION_ID` defines the session id of the current experiment the created component is a part of.
* `HOBBIT_CONTAINER_NAME` the name of the container.
Additionally, other environment variables can be added by the component that requested the container creation.
 
If an abstract component class from the `hobbit.core` library is used, `HOBBIT_RABBIT_HOST` is added and contains the name of the RabbitMQ host which is used by the platform.

**Note** that variables with a hyphen in their name, e.g., `may-own-variable`, seem to cause problems in Dockers HTTP API. The container will be created but the variable will not be correctly set.

## Creating a container

Requesting the creation of a container is done by sending the [`Commands.DOCKER_CONTAINER_START`](/command_queue#predefined-command-ids) id (=`0x0C`) on the command queue, followed by a JSON string containing the necessary information. Following the structure of [command queue messages](/command_queue#structure-of-a-message) such a request looks like the following:

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

## Stopping and removing a container

Stopping a container can be done by sending a message to the [command queue](command-queue) containing the `Commands.DOCKER_CONTAINER_STOP` id (=`0x0D`) followed by a `UTF-8` encoded String containing the following JSON data:
```json
{
  "containerName": "container-to-stop"
}
```
where `container-to-stop` should be replaced by the container name that has been received during the creation of the container. Note that this message is won't have a direct response from the platform controller but an "indirect" response as described in the next sub section.

## Container termination

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

