---
title: Organizing a Challenge
keywords: HOBBIT Documentation
sidebar: main_sidebar
toc: false
permalink: challenge_organization.html
folder: docs
---

## About challenges

A Challenge is a collection of one or more challenge task. Each challenge task uses exactly one benchmark with a certain configuration to benchmark systems that participate in the challenge. Challenges have some features that separate them from other experiments that can be executed on the platform:
* The experiments of a challenge are executed on a particular date. From that time on, the experiments of the challenge have a higher priority than other experiments.
* Challenge results have a publication date. Before this date is reached, the results are not publicly visible. Participants can see only the results of their system while the challenge owner can see all results.

Note that you need to have a special role to be able to create a challenge. If you want to use the master HOBBIT instance for a certain challenge, please contact the [hobbit project](http://project-hobbit.eu) team.

## Workflow overview

1. Create a challenge
   * If the challenge can be already be visible, you can tick the checkbox. Otherwise, the challenge won't be visible for normal users.
2. Create challenge tasks
   * The creation of the tasks work like the definition of an experiment but without choosing a system.
3. Participant registration
   * After the creation and
   * if the challenge is visible, users can register their system for one or more challenge tasks if the system fits to the benchmark of the challenge task
4. Close the challenge
   * Closing the challenge is necessary to fix the registrations and the challenge task configuration. Note that from that point on, no additional user will be able to register a system for the challenge.
   * Note that participants can still update their system images.
5. Execution date reached
   * The single experiments are executed.
   * The challenge organiser can see the results in the challenge view. The participants can see only their result.
6. Publication date is reached
   * The results are publicly available.

## How to organize a challenge

A challenge organizer can create challenges. In order to do so the challenge organizer needs to be logged in. When selecting the *Challenges* menu item the challenge organizer, other than a conventional user, additionally sees the *Add Challenge* button.

![Challenge page header for a user with the challenge organizer role.](/images/51_Challenge.png)

To add a challenge the user presses the *Add Challenge* button and is re-directed to the page providing a form to add information on the challenge. The challenge organizer is required to enter a challenge name, some description, an execution date and some publishing date.
While the first date marks the point in time at which the platform will start to execute the experiments of the challenge, the second date is the day from which on the results will be published, i.e., they are made visible to everybody.
Additionally, a challenge homepage can be entered which will be rendered as link for the other users.

Additionally the challenge organizer can indicate if a challenge is *Visible for everyone*. Not setting this flag can be used, e.g., in order to safe a challenge and re-edit it late on. In this case it will not be visible to anyone else.

When all parameters are filled out, the challenge organizer is required to save the challenge (press the *Save* button) before adding tasks to the challenge.

![New challenge form page.](/images/52_Challenge.png)

This will redirect the challenge organizer to the challenge list page. By clicking on the newly created challenge in the list the challenge organizer can change the challenge details and add tasks. In order to add tasks the challenge organizer presses the *Add Task* button.

![Filled challenge form without tasks.](/images/54_Challenge.png)

The challenge organizer is then redirected to the challenge task detail page. Here the challenge organizer can name the task and give some more details via the description. Additionally the organizer is required to select and configure a benchmark that implements the described task. This works exactly like the creation of a single experiment described above but without choosing a particular system for benchmarking.

![Edit challenge task details form page.](/images/55_Challenge.png)

After having added the challenge tasks to the challenge the challenge needs to be saved again. In order to allow people to register it needs to be *Visible for everyone* by setting this flag before saving the challenge.

The *Close Challenge* button closes a challenge, i.e., no more registrations are possible and the challenge is waiting for its execution date to arrive for running the experiments. It should only be pressed when the registration for the challenge is over. After closing the evaluation of the registered systems starts as soon as the execution date is reached. When pressing the button a dialogue comes up asking for closing the challenge to ensure this does not happen accidentally.

![Dialog for closing a challenge.](/images/58_Challenge.png)
