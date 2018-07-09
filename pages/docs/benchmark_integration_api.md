---
title: General Benchmark Integration API
keywords: HOBBIT Documentation
sidebar: main_sidebar
toc: false
permalink: benchmark_integration_api.html
folder: docs
---

## RabbitMQ

Command queue structure and command IDs are described at [command queue](/command_queue.html).

## Termination

To indicate that your benchmark is finished,

* call `sendResultModel` method
if you're extending [AbstractBenchmarkController](https://github.com/hobbit-project/core/blob/master/src/main/java/org/hobbit/core/components/AbstractBenchmarkController.java);
* otherwise, send [BENCHMARK_FINISHED_SIGNAL](/command_queue.html).

## RDF Model

The benchmark meta data file (`benchmark.ttl`) comprises meta data about the benchmark that is needed by the Hobbit platform. The file contains the meta data as RDF triples in the Turtle format and needs to be uploaded to the git instance of the platform.

Benchmark description in `benchmark.ttl`
should include an instance of [hobbit:Benchmark](http://w3id.org/hobbit/vocab#Benchmark)
describing your benchmark, docker image it uses, related parameters (if any) and KPIs.

Detailed information can be found in [Hobbit ontology](https://github.com/hobbit-project/ontology/blob/master/ontology.ttl).
