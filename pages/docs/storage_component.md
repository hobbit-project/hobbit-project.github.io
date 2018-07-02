---
title: Storage Component
keywords: HOBBIT Documentation
sidebar: main_sidebar
toc: false
permalink: storage_component.html
folder: docs
---

The storage component contains the experiment results.
It comprises two containers---a Virtuoso triple store that uses the **Hobbit** ontology to describe the results and a Java program that handles the communication between the message bus and the triple store. This communication can be encrypted to make sure that no invasiv benchmark or system image can change the data in the storage.
The storage component offers a public SPARQL Endpoint with read-only access.
