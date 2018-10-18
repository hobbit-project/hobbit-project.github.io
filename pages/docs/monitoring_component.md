---
title: Resource Monitoring
keywords: HOBBIT Documentation
sidebar: main_sidebar
toc: false
permalink: monitoring_component.html
folder: docs
---

Resource monitoring in docker swarm is performed with [Prometheus](https://prometheus.io/).
The following exporters are used:
* [Node exporter](https://github.com/prometheus/node_exporter)
for gathering general hardware information
* [cAdvisor](https://github.com/google/cadvisor)
for gathering information about running docker containers

Example configuration file for Prometheus is at [config/prometheus/example.conf](https://github.com/hobbit-project/platform/blob/develop/config/prometheus/prometheus.conf).

Since metrics are only updated periodically,
a delay may be needed to be sure that the metric has been updated
when requesting information about running system.
