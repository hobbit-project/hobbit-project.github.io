---
title: Experiment workflow
keywords: HOBBIT Documentation
sidebar: main_sidebar
toc: false
permalink: experiment_workflow.html
folder: docs
---

## Experiment Workflow

The HOBBIT platform does not demand the implementation of a very detailed workflow. In fact, the platform itself has a small API which has to be implemented by the benchmark and the system for a hand full of necessary messages (the minimal API for the benchmark and the system will be explained in TODO). Apart from that, the real workflow of the benchmarking is up to the benchmark and system developer.

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

Copy the workflow from D2.2.2 including pictures.

