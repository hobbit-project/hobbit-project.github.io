---
title: Java SDK
keywords: tutorials
sidebar: main_sidebar
toc: false
permalink: java_sdk.html
folder: docs
---

[HOBBIT Java SDK](https://github.com/hobbit-project/java-sdk)
is a standalone software library
targeted to make the design and development
of HOBBIT-compatible components easier.

The SDK helps platform users with the following tasks:

- design systems for benchmarking and debug them within a particular benchmark;
- design a benchmark and debug the interactions between its components;
- upload results (docker images of your components) to the online platform.

### Creating a benchmark

Make sure you have all [HOBBIT requirements](/requirements.html) met.

Clone [Java SDK Example kit](https://github.com/hobbit-project/java-sdk-example/)
which contains a template for a benchmark.

Implement your data generator in [DataGenerator::generateData](https://github.com/hobbit-project/java-sdk-example/blob/master/src/main/java/org/hobbit/sdk/examples/examplebenchmark/benchmark/DataGenerator.java)
and your task generator in [TaskGenerator::generateTask](https://github.com/hobbit-project/java-sdk-example/blob/master/src/main/java/org/hobbit/sdk/examples/examplebenchmark/benchmark/TaskGenerator.java).

Test the benchmark as a Java code:
`mvn -Dtest=ExampleBenchmarkTest#checkHealth test`.

Configure project name in
[Constants](https://github.com/hobbit-project/java-sdk-example/blob/master/src/main/java/org/hobbit/sdk/examples/examplebenchmark/Constants.java)
if needed.

Package the benchmark as a jar file:
`mvn -DskipTests package`.

Build the benchmark Docker image:
`mvn -Dtest=ExampleBenchmarkTest#checkHealthDockerized test`.

Now, you can test the benchmark using Docker containers:
`mvn -Dtest=ExampleSystemTest#checkHealth test`.

After that, the benchmark image is ready and can be `docker push`-ed
to remote repository.

As usual, you'll need to add [a `.ttl` file](/benchmark_integration_api.html) describing your benchmark.

For more details, consult
[HOBBIT Java SDK Example kit README](https://github.com/hobbit-project/java-sdk-example).
