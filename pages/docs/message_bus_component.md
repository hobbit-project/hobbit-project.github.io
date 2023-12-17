---
title: Message Bus Component
keywords: HOBBIT Documentation
sidebar: main_sidebar
toc: false
permalink: message_bus_component.html
folder: docs
---

## Overview

This component contains the message bus system.
There are several messaging systems available from which we chose RabbitMQ.
Table shows the different queues that are used by the platform components.
We will use the following queue types supported by RabbitMQ:
* **Simple.** A simple queue has a single sending component and a single receiving consumer.
* **Bus.** Every component connected to this queue receives all messages sent by one of the other connected components.
* **RPC.** The queue has one single receiving consumer that handles incoming requests, e.g., a SPARQL query, and sends a response containing the result on a second parallel queue back to the emitter of the request.

Queues that are used exclusively by the benchmarking components or the benchmarked system are not listed in the table, since their usage depends on a particular benchmark.
However, if the benchmark relies on RabbitMQ, the benchmark implementation has to add the ID of the experiment to the execution queue in order to be executed in parallel with other submitted benchmarks.

**Name** | **Type** | **Description**
-------- | -------- | ---------------
`hobbit.command` | Bus | Broadcasts commands to all connected components.
`hobbit.storage` | RPC | The storage component accepts reading and writing SPARQL queries. The response contains the query result.
`hobbit.frontend-controller` | RPC | Used by the front end to interact with the controller.
`hobbit.controller-analysis` | Simple | Used by the controller to send the URIs of finished experiments to the analysis component which uses them for further analysis.

The `hobbit.command` queue is used to connect the loosely coupled components and orchestrate their activities.
Since the platform should be able to support the execution of more than one experiment in parallel, we will use a simple addressing scheme to be able to distinguish between platform components and benchmark components of different experiments.
Table shows the structure of a command message.
Based on the **Hobbit** ID at the beginning of the message, a benchmark component can decide whether the command belongs to its experiment.
The message contains a byte that encodes the command that is sent.
Based on this command a component that belongs to the addressed experiment decides whether it has to react to this message.
Additional data can be appended as well if necessary.

**Bytes** | **Type** | **Description**
--------- | -------- | ---------------
0..3 | int | Length `h` of the **Hobbit** ID.
4..`h + 3` | String | **Hobbit** ID of the experiment this command belongs to.
`h + 4` | byte | ID of the command.
> `h + 4` | byte[] | Additional data (optional)

## Communication scenarios

The platform offers three scenarios for a RabbitMQ-based communication between containers of a single experiment.

1.  A single broker for everything

    The simplest solution is to use a single RabbitMQ broker for the complete communication, i.e., the communication of the platform components and the communication of the components of an experiment. This approach needs the least amount of resources. It can be easily configured by setting the `HOBBIT_RABBIT_EXPERIMENTS_HOST` environmental variable of the platform controller to the name of the platform's RabbitMQ instance. However, in larger setups, it is might be better to separate the platform and experiment communication into two different brokers.

2.  A permanent broker for experiments

    In the second scenario, the platform operates with two brokers – one for the platform internal communication and the second for experiments. To run the platform with this configuration, a second RabbitMQ broker can be added to the docker-compose file of the platform. It's name is used as value for the `HOBBIT_RABBIT_EXPERIMENTS_HOST` environmental variable of the platform controller.

3.  A new broker for each experiment

    It is possible that faulty implementations of benchmarks or systems can spam the RabbitMQ broker with messages and leave it in a bad state which makes it harder to operate correctly for the next experiment. For such cases, the platform offers the creation of a new broker for each experiment. To this end, the image name (and version) of the RabbitMQ broker should be given to the platform controller by defining it's `HOBBIT_RABBIT_IMAGE` environmental variable.
