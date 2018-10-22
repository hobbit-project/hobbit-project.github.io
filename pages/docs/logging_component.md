---
title: Logging Component
keywords: HOBBIT Documentation
sidebar: main_sidebar
toc: false
permalink: logging_component
folder: docs
---

The logging comprises of three single components---Logstash, Elasticsearch and Kibana.
While Logstash is used to collect the log message from the single components, Elasticsearch is used to store them inside a fulltext index.
Kibana offers the front end for accessing this index.

## Logging inside Components

Single components should write log messages to a standard output.
Docker containers will be configured to send the standard outputs to the Logstash instance.

We encourage the usage of a facade, e.g., [slf4j](http://www.slf4j.org/), in the program code.
This offers the advantage that a certain implementation, e.g., log4j, can be chosen without influence on the already written code.
Additionally, all components should share a single pattern for the log messages to ease the analysis of log messages and the configuration of Kibana.
