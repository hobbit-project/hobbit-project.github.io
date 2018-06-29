---
title: Platform Config File
keywords: HOBBIT Documentation
sidebar: main_sidebar
toc: false
permalink: parameters_config.html
folder: docs
---

The controller can be configured in more detail using the `config.yaml` file that can be mounted to the platform controller container as described in the [Volumes section](https://github.com/hobbit-project/platform/wiki/Configuration-parameters#volumes).

## Benchmark-specific timeouts
For single benchmarks, the timouts can be configured in the following way
```yaml
timeouts:
  'http://example.org/ExampleBenchmark': # benchmark URL
    benchmark: 2400000 # in ms
    challenge: 2400000 # in ms
```
For every benchmark, there can be two timeouts. One for the benchmark when it is executed in the "normal" way and a second timeout for the benchmark when it is used for a challenge.
