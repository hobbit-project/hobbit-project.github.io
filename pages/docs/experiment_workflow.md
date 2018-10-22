---
title: Experiment workflow
keywords: HOBBIT Documentation
sidebar: main_sidebar
toc: false
permalink: experiment_workflow
folder: docs
---

## Experiment Workflow

The HOBBIT platform does not demand the implementation of a very detailed workflow. In fact, the platform itself has a small API which has to be implemented by the benchmark and the system for a hand full of necessary messages (the minimal API for the benchmark and the system will be explained in [How to integrate a Benchmark](/benchmark_integration)). Apart from that, the real workflow of the benchmarking is up to the benchmark and system developer.

However, since we want to support the development of benchmarks and system adapters, we defined a second, more fine grained workflow which is supported by our abstract Java class implementations.

### Minimal Workflow

In the context of the HOBBIT platform, an experiment is defined as the benchmarking of an experiment with a single benchmark configuration. Therefor it receives a URI and comprises a single system instance, a single benchmark, a certain configuration for this benchmark and a set of values of the benchmarks key performance indicators (KPIs).

The general workflow of an experiment can be easily summarized in the following steps.

1. The platform prepares the experiment. This includes the pulling of the Docker containers that have been defined in the benchmarks and systems meta data.
2. The Docker containers of the benchmark and the system are created. Both containers should initialize themselfs. As soon as they are ready for the benchmarking phase, they are sending a message to the platform.
3. The benchmarking is started as soon as both parts are ready. The platform will wait for the benchmark to finish its benchmarking phase and transfer the results of the benchmarking.
4. The results are stored and the platform is cleaning up before the next experiment is executed.

This very general workflow is assumed by the platform and has to be implemented by every benchmark or system that should be integrated into the platform. Note that the platform does not define how the benchmark and the system have to communicate with each other.

### Recommended Workflow

<!-- https://docs.google.com/drawings/d/1wj22SfFHUxg_Az76p-Xuhcfq7RTpH1JlkHsjf54nSag/edit -->

![sequence diagram](/images/Sequence_diagram.svg)

Figure shows a sequence diagram containing the steps as well as the type of communication that is used.
However, the orchestration of the single benchmark components is part of the benchmark and might be different.

1. The platform controller makes sure that a benchmark can be started. This includes a check to make sure that all nodes of the cluster are available.
1. The platform controller spawns the components of benchmarked system.
    * The system initializes itself and makes sure that it is working properly.
    * It sends a message to the platform controller to indicate that it is ready.
1. The platform controller spawns the benchmark controller.
    * The benchmark controller spawns the data and task generators as well as the evaluation storage.
    * The benchmark controller sends a message to the platform controller to indicate that it is ready.
1. The platform controller waits until the system as well as the benchmark controller are ready.
1. The platform controller sends a start signal to the benchmark controller which starts the data generators.
1. The data generators start their algorithms to create the input data.
    * The data is sent to the system and to the task generators.
    * The task generators generate the tasks and send it to the system.
    * The systems response is sent to the evaluation storage.
1. The task generators send the expected result in the evaluation storage.
1. After the data and task generators finished their work the benchmarking phase ends and the generators as well as the benchmarked system terminate.
1. After that the terminated components are discarded and the benchmark controller spawns the evaluation module.
1. The evaluation module loads results from the evaluation storage. This is done by requesting the results pairs, i.e., expected result and actual result received from the system for a single task, from the storage. The evaluation module uses these pairs to evaluate the system's performance and calculate the Key Performance Indicators (KPIs). The results of this evaluation are returned to the benchmark controller before the evaluation module terminates.
1. The benchmark controller sends the signal to the evaluation storage to terminate.
1. The benchmark controller sends the evaluation results to the platform controller and terminates.
1. After the benchmark controller has finished its work, the platform controller can add additional information to the result, e.g., the configuration of the hardware, and store the result. After that, a new evaluation could be started.
1. The platform controller sends the URI of the experiment results to the analysis component.
1. The analysis component reads the evaluation results from the storage, processes them and stores additional information in the storage.
