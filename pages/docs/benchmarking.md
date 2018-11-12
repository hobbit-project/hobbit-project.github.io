---
title: Benchmarking a System
keywords: HOBBIT Documentation
sidebar: main_sidebar
toc: false
permalink: benchmarking.html
folder: docs
---

{% include note.html content="We have a [video tutorial](https://www.youtube.com/watch?v=QWPujpc9srM) covering the system benchmarking." %}

Being a registered user of the platform and [having uploaded a system](/system_integration.html#3-create-and-push-the-docker-image) which conforms the specification (API) of one of the benchmarks allows the user to benchmark the system.
The benchmarking of a system is done via the *Benchmarks* menu where at first the benchmark is selected to be used for the experiment. The drop down menu displays all possible benchmarks.

![Configuration of a benchmarking experiment. (a) Select the benchmark.](/images/10_Benchmark.png)

Having selected the benchmark, the system to be benchmarked is selected. Only systems uploaded by the user and fitting the API of the chosen benchmark are displayed. Then the benchmark experiment is configured by setting the benchmark specific parameters. These might vary amongst the different benchmarks due to their different nature. Parameters can be e.g. numeric values, string values, dates or even nominal with pre-defined values that can be selected by a drop-down box. Some of the values might also have restrictions.

![Configuration of a benchmarking experiment. (b) Select the system and configure experiment.](/images/12_Benchmark.png)

When the experiment is configured it can be submitted via the *Submit* button. This button is inactive as long as the configuration is not completed. After successful submission a page with the submission details is presented to the user.

![Configuration of a benchmarking experiment. (c) Submission details.](/images/13_Benchmark.png)
