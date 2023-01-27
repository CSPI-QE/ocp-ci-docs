# OpenShift CI Scenario Onboarding Guide<!-- omit from toc -->

## Table of Contents<!-- omit from toc -->

- [Overview](#overview)
- [Purpose](#purpose)
- [Onboarding Workflow Diagram](#onboarding-workflow-diagram)
- [Onboarding Phases](#onboarding-phases)
  - [1. Prerequisites](#1-prerequisites)
  - [2. Scenario Kick-off](#2-scenario-kick-off)
  - [3. Develop Scenario](#3-develop-scenario)
  - [4. Maintenance \& Expansion](#4-maintenance--expansion)
  - [5. Celebrate \& Share](#5-celebrate--share)

## Overview

This Onboarding guide is meant to describe a repeatable process that can be followed to create test scenarios for OpenShift integrated products using OpenShift CI. The goal is that we can onboard & expand new scenarios that are easy to understand, maintain and debug.

We must deeply understand the need for each specific scenario that we put through this process and update this process when needed. This is not meant to be followed blindly so that we can onboard scenarios faster. It is meant to teach a generic way to onboard scenarios that is proven to work. It's expected that new scenarios will present new problems and we must develop better solutions than what may be proposed in this document. We need to hear your pain-points and feedback throughout every step of this process. 
> **NOTE: **
> Please communicate over the [#forum-qe-cspi-ocp-ci](https://coreos.slack.com/archives/C047Y0DPEJU) Slack channel

## Purpose

There are many layered products that we've built on top of OpenShift. We need to make sure that all of these products are working with the latest OpenShift builds. In order to effectively test, debug, and maintain all of these product tests and show the results in a consumable report, we need to have some structure in place.

If this onboarding process did not exist we'd be scrambling to put together reports efficiently, find automation bugs, fix automation bugs, find testing gaps, update test scenarios frequently, ..etc.

## Onboarding Workflow Diagram

TBD

## Onboarding Phases

### 1. Prerequisites

In order for onboarding to run smoothly and avoid becoming blocked for long stretches we will be upholding a high standard for achieving the prerequisites needed for this model. We predict that accomplishing these prerequisites will take longer than the onboarding itself.

The result of working through your prerequisites will be the completion of the Prerequisite JIRA ticket assigned to you. All information proving that this scenario is ready to be onboarded will need to be provided and approved by both parties.

See the [Prerequisites Guide](Prerequisites_Guide.md) for a detailed explanation of what is needed and how to achieve it.

### 2. Scenario Kick-off

See the [Kick-off Guide](Kickoff_Guide.md) for the process to follow when officially starting the cross-team collaboration for a scenario.

### 3. Develop Scenario

Here is where most of work will be done for the scenario.

See the [Scenarios Guide](../OCP_CI_Tutorials/Scenarios/Scenarios_Guide.md) for an overview of the work that is needed.

### 4. Maintenance & Expansion

See the [Scenario Maintenance Policy](../Policy/Maintenance/Scenario_Maintenance_Policy.md) for detailed information meant to help you maintain and expand your scenarios testable versions in a consistent and easy fashion.

### 5. Celebrate & Share

See [Celebrate & Share](Celebrate_%26_Share_Guide.md) for fun information meant to help you communicate your success with others so that we can all benefit from what you've created! :)
