---
title: Frontend
keywords: HOBBIT Documentation
sidebar: main_sidebar
toc: false
permalink: parameters_frontend
folder: docs
---

The frontend is split in two different components: an angular frontend and a Java backend. The Java backend is configured using environment variables. Yet, as angular can not be configured via environment variables, a manual configuration may be necessary. _For any user that just wants to use the default configuration no config changes are necessary!_

The frontend is configured [here](https://github.com/hobbit-project/platform/blob/master/hobbit-gui/gui-client/src/environments/environment.prod.ts). The only important configuration is `backendUrl`. This options specifies where the Java backend is deployed. If the frontend is bundled with the backend, an empty string has to be provided. Yet, if the backend is deployed somewhere else, specify the according url here, e.g. `localhost:8080`.
