---
title: Platform Controller Component
keywords: HOBBIT Documentation
sidebar: main_sidebar
toc: false
permalink: platform_controller_component
folder: docs
---

The platform controller is the central component of the **Hobbit** platform coordinating the interaction of other components if needed.
This mainly includes the handling of requests that come from the front end component, starting and stopping of benchmarks, observing of cluster health status and triggering of the analysis component.

## Persistent Status
The internal status of the platform controller is stored in a Redis database.
This enables the shut down or restart of a platform controller without loosing current status, e.g., benchmarks that have already been configured and pushed into the execution queue.

## Queue

Experiments submitted by users obtain a unique **Hobbit** ID and are put into the execution queue managed by the platform controller.
This queue is a "First in, First out" queue, i.e., experiments are executed in the same order in which they have been added.
However, the platform controller guarantees, that the experiments of a certain challenge are executed on a certain date.
Thus, the order of the experiments change as soon as the execution date of a challenge has come.
In this case, the experiments of the challenge are executed first.

## Benchmark Execution
The platform controller contains a queue of experiments, i.e., benchmark and system combinations.
If there is no running experiment and the queue is not empty, the platform controller initiates the execution of an experiment in the following way:
1. The platform controller makes sure that a benchmark can be started. This includes a check to make sure that the system/cluster is healthy.
1. The platform controller loads the maximum runtime the benchmark should get.
1. The platform controller spawns the benchmarked system. This includes a check whether the benchmark defines parameters that should be forwarded to the system.
1. The platform controller spawns the benchmark controller.
1. The system as well as the benchmark controller load all the data that they need and make sure that they are working properly. Afterwards they send a message to the platform controller using the command queue to indicate that they are ready. The platform controller is waiting for these messages.
1. After both the system and the benchmark controller are ready, the platform controller sends a start signal to the benchmark controller. With this signal, the platform controller stops observing the system container. It is assumed that the benchmark controller takes care of sending a termination signal to the system.
1. The platform controller receives the results from the benchmark controller.
1. The results are extended with additional information about the hardware before they are send to the storage.

The platform controller observes the state of the benchmark controller.
If the experiment takes more time than a configured maximum, the platform controller terminates the benchmark controller and all the containers that belong to it.
Since the platform controller manages the creation of containers it has a list of created containers that belong to the currently running benchmark.
This list is used to make sure that all resources are freed and the cluster is ready for the next experiment.

## Challenge management

The platform controller manages several actions necessary to support the workflow of challenges. Firstly, it makes sure that if a challenge is closed every system is benchmarked for the challenge task it has been registered for.

Secondly, it takes care of the publication of challenge results. The platform offers the definition of a distinct publication date for a challenge. Before this date is reached, the complete set of results is only visible for the organizer of a challenge. When the date is reached, the results are made public. This mechanism ensures that the challenge organizer can already gather the winners of the single task challenges for announcing them during a ceremony while nobody else can get the complete list of results.

Thirdly, the platform controller takes care of executing repeatable challenges. These challenges are special in the way that there is no one single date at which the registration of systems closes, but there are several dates at which the registered systems are benchmarked.
This allows the challenge organizer to run its challenge over several months with intermediate results and a public leaderboard.

## Interaction with Analysis Component
The platform controller triggers the analysis component sending the URI of an terminated experiment to inform the analysis component that new results for this combination are available.

## Docker Container Creation
The platform controller is the only component that has direct access to the docker daemon.
If another component would like to start a docker container, it has to send a request to the platform controller containing the image name and parameters.
Thus, the platform controller offers the central control of commands that are sent to the docker daemon which increases the security of the system.

## Resource usage gathering

As a part of the API offered by the platform controller the resource usage information of the currently benchmarked system can be queried. This enables the benchmarks to measure the systems efficiency not only based on the response times but also based on the resource statistics. This statistics contains the sum of a) CPU time, b) main memory and c) disk space that the containers of the system are consuming.
