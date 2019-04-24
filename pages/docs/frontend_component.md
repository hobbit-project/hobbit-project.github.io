---
title: Frontend Component
keywords: HOBBIT Documentation
sidebar: main_sidebar
toc: false
permalink: frontend_component.html
folder: docs
---

The front end component handles the interaction with the user.
It offers different functionalities to different user groups.
Thus, it contains a user management that allows different roles for authenticated users as well as a guest role for unauthenticated users.

## Roles

* **Guest.** A guest is an unauthenticated user. This type of user is only allowed to read the results of experiments and analysis.
* **Registered user.** This user is allowed to upload system images and start benchmarks for his own systems. 
Additionally, the user can register its systems for challenge tasks.
* **Challenge Organizer.** This user is allowed to organize a challenge, i.e., define challenge tasks with a benchmark and a certain date at which the experiments of the challenge will be executed.

## Visibilities

The aim of the HOBBIT platform is to share benchmarking results. Therefore, nearly all data is publicly visible. However, there are some cases in which the platform does not show all information to all users. In the following, these situations are documented.

### Benchmark visibility

Benchmarks are always treated as public. Thus, they can be seen and used by everybody.

### System visibility

Sometimes, a user would like to benchmark his system without giving other users access to the system. Therefore, visibility of systems in the frontend follows the visibility of their git project in our repository. In the following table, the necessary rights are listed for using a system in a certain situation with a certain [Gitlab visibility](https://docs.gitlab.com/ee/public_access/public_access.html). The letters have the following meaning:

* G = guest
* U = registered user
* R = the user is registered and has at least read access to the gitlab project of the system

Situation \ Project type | public | internal | private
------------ | ------------- | ------------- | -------------
Start experiment | U | U | R |
Register system for a challenge | U | U | R |
See public experiment results | G | G | G |
See private experiment results | U | U | R |

### Visibility in challenges

During the execution of a challenge, [not all experiments are made public](https://hobbit-project.github.io/internal_challenge_workflow.html#storage-of-challenge-data). In the following table, the necessary rights are listed for seeing private experiment results and registrations of systems of a challenge together with the with the [Gitlab visibility](https://docs.gitlab.com/ee/public_access/public_access.html) of the system.

* U = registered user
* R = the user is registered and has at least read access to the gitlab project of the system
* O = challenge organizer

Situation \ Project type | public | internal | private
------------ | ------------- | ------------- | -------------
Register a system for a challenge | U | U | R |
See private experiment results | U | U | R,O |
