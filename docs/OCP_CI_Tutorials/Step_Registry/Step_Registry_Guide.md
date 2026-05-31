# OpenShift CI Step Registry Guide<!-- omit from toc -->

## Table of Contents<!-- omit from toc -->

- [Introduction](#introduction)
- [Step Types](#step-types)
  - [Refs](#refs)
  - [Chains](#chains)
  - [Workflows](#workflows)
- [The interop Folder](#the-interop-folder)
  - [Tooling](#tooling)
  - [CSPI-Maintained Steps Outside `interop/tooling`](#cspi-maintained-steps-outside-interoptooling)
  - [Scenarios](#scenarios)

## Introduction

This document is meant to serve as a general guide to the step registry in OpenShift CI. It will cover:

- The difference between the types of steps.
- The `interop` folder in the step registry.

> **NOTE:**
> 
> This is guide is not meant to replace the official OpenShift CI documentation, but rather act as supplemental documentation for how the CSPI team plans to use these steps. Please see the official OpenShift CI [ steps documentation](https://docs.ci.openshift.org/docs/architecture/ci-operator-internals/steps/) or [step-registry documentation](https://docs.ci.openshift.org/docs/architecture/step-registry/) for additional information.

## Step Types

The step registry is a folder in the [openshift/release](https://github.com/openshift/release) repository that stores steps that can be used during test execution. For the purposes of this document, we will only cover refs, chains, and workflows as those are the steps primarily used by the CSPI team at the time of writing this document.

### Refs

A ref is the most basic step in the step registry, it can be thought of as the building-blocks of other steps. A ref is essentially a BASH script that can be used on it's own or as part of a chain or workflow. 

Please see the [Ref Guide](Step_Registry_Ref_Guide.md) for more information.

### Chains

A chain allows you to string refs and other chains together to be executed in order. Chains are useful to help keep configurations clean and useful. When we create a scenario's chain to follow the "orchestrate, execute, report" structure that CSPI follows, it allows us to see the whole scenario in one configuration file and may prevent some confusion. 

Please see the [Chain Guide](Step_Registry_Chain_Guide.md) for more information.

### Workflows

A workflow can be thought of as a set of [chains](Step_Registry_Chain_Guide.md) and [refs](Step_Registry_Ref_Guide.md). Any of these steps can be defined as either a `pre` step, `test` step, or `post` step.

Workflows can be useful in many situations, but CSPI currently uses them mainly as cluster provisioning and deprovisioning steps. It is possible that our use of workflows will expand as we continue to grow in OpenShift CI.

Please see the [Workflow Guide](Step_Registry_Workflow_Guide.md) for more information.

## The interop Folder

The `ci-operator/step-registry/interop` folder is meant to house all of our scenario-specific steps as well as any re-usable tooling we create in the step registry.

### Tooling

The `ci-operator/step-registry/interop/tooling` folder is meant to hold any re-usable tooling the CSPI team created in the OpenShift CI step registry. These tools are encouraged to be used both in CSPI interop testing and in tests that other teams may be working on. We will own and maintain these tools.

### CSPI-Maintained Steps Outside `interop/tooling`

The CSPI-QE team also maintains re-usable steps elsewhere in the step registry. These are owned by `cspi-qe-ocp-lp` and documented on
[steps.ci.openshift.org](https://steps.ci.openshift.org/):

| Step | Type | Purpose | Official documentation |
|------|------|---------|------------------------|
| `firewatch-ipi-aws-cr` | Workflow | AWS IPI provisioning with Report Portal upload and Jira failure reporting for LP OCP Compat / CR jobs | [`firewatch-ipi-aws-cr`](https://steps.ci.openshift.org/workflow/firewatch-ipi-aws-cr) |
| `mpiit-data-router-reporter` | Ref | Uploads JUnit XML result files from a CI Operator Job run to Report Portal via Data Router, with TFA applied by default | [`mpiit-data-router-reporter`](https://steps.ci.openshift.org/reference/mpiit-data-router-reporter) |
| `firewatch-report-issues` | Ref | Reports CI Operator Job failures to Jira using Firewatch rules | [`firewatch-report-issues`](https://steps.ci.openshift.org/reference/firewatch-report-issues) |

For usage guidance, see the [`firewatch-ipi-aws-cr` workflow section](../Scenario_Development/Scenario_Development_Guide.md#firewatch-ipi-aws-cr) in the Scenario
Development Guide and [Report Portal Upload (Data Router)](../Reporting/Reporting_Guide.md#report-portal-upload-data-router) in the Reporting Guide.

### Scenarios

The `ci-operator/step-registry/interop/{scenario_name}` folders are meant to hold the chains and refs needed to execute specific scenarios. This folder should hold any scenario-specific steps like:

- The Orchestrate chain or ref for a scenario.
- The Execute chain or ref for a scenario.
- Any other scenario-specific steps that must be used during the execution of a scenario.
