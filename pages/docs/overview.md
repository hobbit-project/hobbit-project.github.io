---
title: Platform Overview
keywords: HOBBIT Documentation
sidebar: main_sidebar
toc: false
permalink: overview.html
folder: docs
---

The platform is designed to benchmark Linked Data systems on a cluster. It comprises several components that are designed as [Docker containers](https://www.docker.com/what-docker). The following figure shows an overview over the (simplified) architecture of the platform.

![component diagram](/images/Components_diagram.png)

The different colors in the figure show the different parts of the platform. The blue components are the platform components that offer its main functionality. The orange components belong to a benchmark and are only instantiated if the benchmark is running. The grey component is the system that is benchmarked by the orange benchmark. Note that even if the overview picture shows the system as a single component, it might comprise multiple distributed components.

Internally, the platform is written in Java and its communication is based on [RabbitMQ](http://www.rabbitmq.com/). However, the benchmarks and benchmarked systems can be written in any language as long as a) they are able to communicate with the platform and b) they are available as Docker containers.

## Workflow of a benchmark

A typical workflow of a benchmark is shown in the following sequence diagram. Note that the workflow of single benchmarks can differ from this since the platform has only some basic requirements for a benchmark to be executed.

![sequence diagram](/images/Sequence_diagram.png)

It can be seen that the benchmarking of a system has three different phases. First, the system and the benchmark are created and initialized. Second, the benchmark starts to benchmark the system and stores its responses in the evaluation storage. Third, the system as well as other parts are terminated and the data in the evaluation storage is used to evaluate the system responses. The following steps explain the sequence diagram in more detail.

1. The platform controller generates the benchmark controller and the system.
    * The System Adapter loads all the data that it needs and makes sure that it is working properly. After that it sends a message on the command queue to indicate that it is ready.
    * The benchmark controller creates the data generators, the task generators as well as the evaluation storage.
        * 
        * The single components initialize them self and send a message on the command queue that they are ready.
        * The benchmark controller collects these messages and sends a message on the command queue that it is ready.
2. As soon as the platform controller received the messages from the benchmarked system and the benchmark controller, it sends a start message to the benchmark controller to start the benchmarking.
    * The benchmark controller starts its components, i.e., the data and task generators.
        * The data generators create the data that is needed for the benchmarking and send it to the task generators and/or the benchmarked system.
        * The task generators generate the tasks that should be solved by the system and send them to the system. The correct result for the task is stored in the evaluation storage.
        * The System receives the data from the data generator and the tasks from the task generator. Every task has a unique task ID and the system should generate a single response for every task. This response is send to the evaluation storage together with the task ID it belongs to.
    * The benchmark components terminate when a preconfigured amount of data/tasks have been created. The system receives a message, that all task generators terminated and has to terminate as soon as it has processed all remaining tasks that might be in its incoming queue.
3. After the data and task generators as well as the system terminated, the benchmark controller starts the result evaluation.
    * The benchmark controller creates its evaluation module.
    * The evaluation module iterates over the tasks stored in the evaluation module and evaluates the system responses by comparing them with the expected responses.
    * After the evaluation module has send the results to the benchmark controller, the benchmark and its components terminate.
    * The result is enhanced with additional information by the platform controller before it is stored in the platform storage.
