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

It can be seen that the benchmarking of a system has three different phases. First, the system and the benchmark are created and initialized. Second, the benchmark starts to benchmark the system and stores its responses in the evaluation storage. Third, the system as well as other parts are terminated and the data in the evaluation storage is used to evaluate the system responses.

[Experiment workflow](/experiment_workflow.html) explains that in more detail.
