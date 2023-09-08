# (WIP) OpenShift CI Scenario Kick-off Guide<!-- omit from toc -->

## Table of Contents<!-- omit from toc -->

- [Overview](#overview)
- [Jira Tracking (epics \& tasks)](#jira-tracking)
- [Introduction of Resources](#introduction-of-resources)
- [Communication Channels](#communication-channels)
- [Discuss Private vs Public](#discuss-private-vs-public)
- [Discuss RACI Chart](#discuss-raci-chart)
- [Discuss timelines](#discuss-timelines)

## Overview

This kick-off guide exists so that we don't forget to cover an important topic when we first meet to start engaging on this work. We should only be at this step if all perquisites have been met and both parties have signed off on the official plan.

Please take notes during this kick-off and record them in the kick-off Jira task once created.

## Jira Tracking

Each scenario being onboarded will be tracked under its own Epic. Each Epic will consist of the tasks needed to accomplish the onboarding.

Follow below to generate the Jira tickets for your scenario

```BASH
Coming soon
```

Assign the Epic and tasks to the appropriate resources as you uncover them.

## Introduction of Resources

Schedule a kick-off meeting to introduce everyone that has stake in the scenario being tested.

Examples of important things to discuss:

 - How many engineers are available (or needed) to work on this?
 - How much time can be dedicated?
 - Are there any PTOs upcoming?
 - Are there any product releases upcoming?

## Communication Channels

For fast CSPI OCP CI specific questions communicate over Slack at [#forum-qe-cspi-ocp-ci](https://coreos.slack.com/archives/C047Y0DPEJU)

For updates and history tracking please communicate over the JIRA tickets that were created for the specific scenario onboarding that you are working on. See section [Jira Tracking](#jira-tracking)

## Discuss Private vs Public

### 1. Run tests directly on test cluster: OpenShift CI runs upstream, this means any artifact, log, error, console output, ..etc will be made public.

With this in mind you need to discuss what is right for the scenario being tested.

 - Can you test upstream?
 - Do you follow good security practices in your tests that won't expose secrets in output pass or fail?

If this product cannot be tested upstream we need to discuss the exact reason why. Then we must document this reason in the Kick-off Jira ticket. This will be referenced when building the scenario foundation to determine which path to follow.

We can either run everything publicly like OpenShift CI was built to do or we can navigate the path of running and storing output privately.

### 2. Run tests through a pod on test cluster: logs and console output are kept in private, artifacts only show error message.

**The CSPI QE team reccommends the public testing approach.**

## Discuss RACI Chart

There is a shared responsibility for the maintenance of these OpenShift CI scenarios. See the [RACI Chart](RACI_Chart.md) to understand the roles & responsibilities for this onboarding.

## Discuss timelines

After you've discussed all of the above you are ready to discuss timelines.
