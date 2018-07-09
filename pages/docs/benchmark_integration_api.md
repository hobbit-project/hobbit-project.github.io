---
title: General Benchmark Integration API
keywords: HOBBIT Documentation
sidebar: main_sidebar
toc: false
permalink: benchmark_integration_api.html
folder: docs
---

## RabbitMQ

Command IDs for well-known commands for [command queue](/command_queue.html) are defined in
[Constants module](https://github.com/hobbit-project/core/blob/master/src/main/java/org/hobbit/core/Constants.java).

**Command ID constant** | **Command ID value** | **Description**
-------- | -------- | ---------------
BENCHMARK_READY_SIGNAL | 2 | Sent by the benchmarked system to indicate that the system is ready.
BENCHMARK_FINISHED_SIGNAL | 11 | Sent by the benchmarked system to indicate that the benchmark is finished.

## Termination


## RDF Model

