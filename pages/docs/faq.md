---
title: FAQ
keywords: HOBBIT Documentation
sidebar: main_sidebar
toc: false
permalink: faq
folder: docs
---

* [General questions](#general-questions)
    * [Registration and Login](#registration-and-login)
* [Platform Setup questions](#platform-setup-questions)
* [Implementation questions](#implementation-questions)
    * [Communication](/#communication)
    * [Creation of Docker containers](#creation-of-docker-containers)
    * [Implementing Systems](#implementing-systems)

# General questions

## Registration and Login

#### I always get "Retry later" when accessing git.project-hobbit.eu

You might be logged in as guest. Please go to https://master.project-hobbit.eu/, logout, go back to https://git.project-hobbit.eu/ and login again.

# Platform Setup questions

#### A component with a mounted volume does not work correctly

Depending on the file system on which the platform is being deployed has Docker might need additional privileges to mount directories correctly. This is described in #25 and might not only affect the Virtuoso container.

#### Keycloak JDBC error

Keycloak prints a JDBC error because it can not connect to its database. A typical sign of this problem is the line
```
Caused by: org.h2.jdbc.JdbcSQLException: Feature not supported: "autoServerMode && (readOnly || fileLockMethod == NO || fileLockMethod == SERIALIZED || inMemory)"
```
Please have a look at [#140](https://github.com/hobbit-project/platform/issues/140#issuecomment-341407382).

#### Keycloak login fails with "HTTPS required" message

[Keycloak only allows non-SSL access from IPs from private ranges](https://www.keycloak.org/docs/3.3/server_installation/topics/network/https.html).

If you encounter this error in your local setup, it probably means that
your `docker_gwbridge` network does not use private IP range.
You can check `docker_gwbridge`'s network address with:

`docker network inspect docker_gwbridge |grep Subnet`

To fix this, recreate `docker_gwbridge` network and set any private address for it
(make sure it does not conflict with your other networks):

`docker network rm docker_gwbridge`

`docker network create --subnet=172.18.0.1/24 --gateway=172.18.0.1 --opt com.dockerridge.name=docker_gwbridge --opt com.docker.network.bridge.enable_icc=false docker_gwbridge`

(You may need to leave docker swarm first: `docker swarm leave`.)

#### Connection to RabbitMQ failing

After starting the platform, messages similar to the following are printed:
```
platform-controller_1  | 2017-04-11 16:19:26,146 WARN [org.hobbit.core.components.AbstractComponent] - <Couldn't connect to RabbitMQ with try #0. Next try in 5000ms.>
```
This is no problem. As the content of the message states, the component tried to connect to RabbitMQ. However, in most cases the RabbitMQ broker won't have finished its startup procedure and the the component will retry to connect to it. This becomes a problem if `try #5` fails which means that RabbitMQ has needed more than 1 minute to start (and it most probably crashed).

# Implementation questions

## Communication

#### Do I need to use RabbitMQ for my benchmark?
You need to use RabbitMQ to communicate with the platform. However, the communication between the components of your benchmark or the benchmark and your system can be based on other technologies as well.

## Creation of Docker containers

#### How can I manage my containers to share volumes?
Mounting and sharing volumes is not supported at the moment. Shared data has to be sent via network (e.g., as described in https://github.com/hobbit-project/platform/wiki/Transferring-mimicked-data)

#### How can I configure the ports that are forwarded?
Port forwarding is neither necessary nor helpful inside the platform. All created containers will be located in the same virtual network. They can see all ports of each other and can connect with using the container name as host name.

#### I have set an environmental variable but my created container does not receive it.
Please make sure that you have no hyphen in the name of your variable as explained in the [General hints on components](https://github.com/hobbit-project/platform/wiki/General-hints-on-components#environment-variables).

## Implementing Systems

#### I have a system written in Java. Do I need to use Maven to be able to benchmark it?
No.

First of all, the System Adapter can be an own project separated from your original system project. In that case you can [follow the tutorial](/system_integration) and develop the system adapter using Maven and the following three approaches:
1. You can implement the System Adapter as a parallel container that communicates with your system through some API.
1. The System Adapter wraps your program and starts it as a second process in the same container.
1. The System Adapter executes your system as Java class (i.e., executes it in the same VM). For this solution, you need to include the jar file of your system in the system adapter in your pom file. This blog post explains how you can do that in Maven: http://mark.koli.ch/maven-add-local-jar-dependency-to-classpath


