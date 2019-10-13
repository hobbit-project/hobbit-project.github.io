---
title: Cluster at master.project-hobbit.eu
keywords: HOBBIT Documentation
sidebar: main_sidebar
permalink: master.html
folder: docs
---

There is a hosted instance of HOBBIT Platform available at http://master.project-hobbit.eu.

{% include note.html content="We have a [video tutorial](https://www.youtube.com/watch?v=3oeEyHXVd_4) covering the HOBBIT platform usage." %}

## Hardware of the cluster

| Node        | Feature Highlights | Components |
|-------------|--------------------|------------|
| Master      | Intel(R) Xeon(R) CPU E5-2630 v4 @ 2.20GHz, 10&nbsp;cores; 32&nbsp;GB&nbsp;RAM | HOBBIT components |
| Data        | Intel(R) Xeon(R) CPU E5-2630 v3 @ 2.40GHz, 8&nbsp;cores; 64&nbsp;GB&nbsp;RAM; ~100&nbsp;TB&nbsp;storage | Auxiliary services |
| Workers 1-3 | 2 &times; Intel(R) Xeon(R) CPU E5-2630 v3 @ 2.40GHz, 8&nbsp;cores; 256&nbsp;GB&nbsp;RAM | Benchmark components |
| Workers 4-6 | 2 &times; Intel(R) Xeon(R) CPU E5-2630 v3 @ 2.40GHz, 8&nbsp;cores; 256&nbsp;GB&nbsp;RAM | Benchmarked system components |

## User Registration

Please open http://master.project-hobbit.eu and click on `Register` located on the right hand side.
**Please note** that your user name should be the first part of your mail address. If your mail address is `max.power@example.org` you should use `max.power` as user name.

After the registration you are forwarded to the graphical user interface of the platform. Please open a second browser tab and open http://git.project-hobbit.eu. This Gitlab instance is used to host the data, i.e., the benchmarks and benchmarked systems.
Note that for uploading data into the git project you can either set a password and connect via HTTPS or add an SSH key. Both can be done via `Profile Settings` in the upper right corner of the Gitlab UI. **Please note** it if you add a password in Gitlab got using HTTPS, you should reuse the same password as for the registration since otherwise it will be confusing when you have to use which password. However, adding or editing a `*.ttl` file can be done using the user interface of Gitlab as well.

If you are participating in a challenge as a team, it is sufficient to have one single account for the participating team. The platform does not support the groups/teams with multiple accounts, i.e., it won't be possible to share system images among the accounts.

### Known issues

* Issue: After a login to gitlab, you get the error message 422. Solution: You will have to write a mail to `roeder@informatik.uni-leipzig.de`.
* Issue: If you are using the platform (http://master.project-hobbit.eu) and your session expires because of inactivity (for several hours), the gui might get stuck. Solution: Please reload the complete page to be redirected to the login screen.

## Upload a System

Every system that should be benchmarked on the Hobbit platform needs to be uploaded to it. This uploading comprises several steps. Note that a user profile is necessary to have the right to upload something to the platform.

1. Before uploading something, an adapter for the system needs to be developed, so that the system fits into the platform and is able to interact with the benchmark with which it should be benchmarked. The general development of the adapter is [described here](/system_integration.html).
2. The second step is the creation of a project in the [gitlab instance](http://git.project-hobbit.eu) of the platform. For that you can use the same account as for the platform itself. (The project should have the same name as the system. Please have a look at the [image upload](/system_integration.html) and its hints regarding the project name.)
3. After creating a git project, you need to push the Docker image to the git project as [described here](/system_integration.html).
4. The final step is to put the meta data describing your system into a file called `system.ttl` and push it to the git project. The format of the file is [described here](/system_integration.html).

After uploading a system, [starting an experiment](/benchmarking.html) including the uploaded system and the benchmark for which the adapter has been implemented is recommended to make sure that the adapter is working as expected.

## Upload a Benchmark

Every benchmark that should be run on the Hobbit platform needs to be uploaded to it. This uploading comprises several steps. Note that a user profile is necessary to have the right to upload something to the platform.

1. Before uploading, the benchmark has to be implemented in a way that it fits into the platform and is able to interact with the paltform controller. The general development of a benchmark is [described here](/benchmark_integration.html).
2. The second step is the creation of at least one project in the [gitlab instance](http://git.project-hobbit.eu) of the platform. For that you can use the same account as for the platform itself. (The project should have the same name as the system. Please have a look at the [image upload](/system_integration.html) and its hints regarding the project name.)
3. After creating the git project(s), you need to push the Docker image(s) to the git project(s) as [described here](/system_integration.html).
4. The final step is to put the meta data describing the benchmark into a file called `benchmark.ttl` and push it to one of the git projects. The format of the file is [described here](/benchmark_integration_api.html).
