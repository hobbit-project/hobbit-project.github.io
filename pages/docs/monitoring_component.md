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
* [Node exporter](https://github.com/prometheus/node_exporter) for gathering general hardware information
* [cAdvisor](https://github.com/google/cadvisor) for gathering information about running docker containers
