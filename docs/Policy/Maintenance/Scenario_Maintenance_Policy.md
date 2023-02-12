# OpenShift CI Scenario Maintenance Policy<!-- omit from toc -->

## Table of Contents<!-- omit from toc -->

- [Overview](#overview)
- [Scenario Expansions](#scenario-expansions)
  - [Updating the Scenario Config](#updating-the-scenario-config)
- [Scenario Deprecations](#scenario-deprecations)
- [OpenShift Deployment](#openshift-deployment)
- [Layered Product Deployment](#layered-product-deployment)
- [Test Execution](#test-execution)
- [Triggering Cadence](#triggering-cadence)
- [Reporting Tests](#reporting-tests)
- [Skipping Scenarios](#skipping-scenarios)

## Overview

You'll see in this [RACI Chart](../../Onboarding/RACI_Chart.md) that there is a shared responsibility for the maintenance of these OpenShift CI scenarios. This document is meant to describe the tasks included in maintaining a scenario and state who should be responsible.

We define scenario maintenance as:
> Any task needed to ensure that the scenario executes correctly when automatically triggered.

Scenario maintenance tasks include:

1. Scenario Expansions (PQE)
2. Scenario Deprecations (PQE)
3. OpenShift Deployment (Workflow OWNERS)
4. Layered Product Deployment (PQE)
5. Test Execution (PQE)
6. Triggering Cadence (CSPI-QE)
7. Reporting Tests (CSPI-QE)
8. Skipping Scenarios (CSPI-QE)

## Scenario Expansions

> This is the responsibility of the PQE team.

**Currently under evaluation**

A scenario expansion can be defined as preparing your test scenario to execute tests against a newer layered product release than what was previously configured. For example, if the scenario executes tests for `ACM2.6 on OCP4.12` then an expansion would mean to extend the config for this scenario to now include testing for `ACM2.7 on OCP4.12`

### Updating the Scenario Config

Here we have the `tests"` section of a scenario config for the ACM scenario.

```yaml
tests:
- as: acm-interop-aws
  cron: 0 1 * * 1
  steps:
    cluster_profile: aws-cspi-qe
    env:
      BASE_DOMAIN: aws.interop.ccitredhat.com
      SUB_CHANNEL: release-2.6
      SUB_INSTALL_NAMESPACE: open-cluster-management
      SUB_PACKAGE: advanced-cluster-management
      SUB_SOURCE: redhat-operators
      SUB_TARGET_NAMESPACES: open-cluster-management
    test:
    - chain: interop-acm
    workflow: ipi-aws
```

This is showing us that the layered product being installed is being controlled by the `SUB_CHANNEL` env variable. This will install the `ACM2.6` version of the operator from the operatorhub.

If our goal is to expand the scenario so that we also test `ACM2.7` then we just need to create a new test stanza with an updated value for `SUB_CHANNEL`. So now we will have.

```yaml
tests:
- as: acm2.6-interop-aws
  cron: 0 1 * * 1
  steps:
    cluster_profile: aws-cspi-qe
    env:
      BASE_DOMAIN: aws.interop.ccitredhat.com
      SUB_CHANNEL: release-2.6
      SUB_INSTALL_NAMESPACE: open-cluster-management
      SUB_PACKAGE: advanced-cluster-management
      SUB_SOURCE: redhat-operators
      SUB_TARGET_NAMESPACES: open-cluster-management
    test:
    - chain: interop-acm
    workflow: ipi-aws
- as: acm2.7-interop-aws
  cron: 0 1 * * 1
  steps:
    cluster_profile: aws-cspi-qe
    env:
      BASE_DOMAIN: aws.interop.ccitredhat.com
      SUB_CHANNEL: release-2.7
      SUB_INSTALL_NAMESPACE: open-cluster-management
      SUB_PACKAGE: advanced-cluster-management
      SUB_SOURCE: redhat-operators
      SUB_TARGET_NAMESPACES: open-cluster-management
    test:
    - chain: interop-acm
    workflow: ipi-aws
```

This will lead to two OCP clusters being provisioned in parallel (one will be the base for testing `ACM2.6` and the other will test `ACM2.7`).

> **IMPORTANT:**
>
> This simple process for expansions can be followed so long as the scenario chain, for example `chain: interop-acm` and PQE's test suites are capable of running tests for a specific product version. We need to be diligent when checking for this. We cannot have tests running against a 2.7 version that only work for a 2.6 version of the product.
>
> Also we must ensure that a rehearsal job is run and passes based on the new expansion code.

## Scenario Deprecations

> This is the responsibility of the PQE team.

**Currently under evaluation**

When a layered product version is no longer supported on the OpenShift release that it is being tested on we must remove the older version of the layered product from the scenario config file. Using the example above, a scenario deprecation will just involve removing the test stanza for the older version that needs to be deprecated.

 For example if we had

 ```yaml
tests:
- as: acm2.5-interop-aws
  cron: 0 1 * * 1
  steps:
    cluster_profile: aws-cspi-qe
    env:
      BASE_DOMAIN: aws.interop.ccitredhat.com
      SUB_CHANNEL: release-2.5
      SUB_INSTALL_NAMESPACE: open-cluster-management
      SUB_PACKAGE: advanced-cluster-management
      SUB_SOURCE: redhat-operators
      SUB_TARGET_NAMESPACES: open-cluster-management
    test:
    - chain: interop-acm
    workflow: ipi-aws
- as: acm2.6-interop-aws
  cron: 0 1 * * 1
  steps:
    cluster_profile: aws-cspi-qe
    env:
      BASE_DOMAIN: aws.interop.ccitredhat.com
      SUB_CHANNEL: release-2.6
      SUB_INSTALL_NAMESPACE: open-cluster-management
      SUB_PACKAGE: advanced-cluster-management
      SUB_SOURCE: redhat-operators
      SUB_TARGET_NAMESPACES: open-cluster-management
    test:
    - chain: interop-acm
    workflow: ipi-aws
```

And we wanted to deprecate testing `ACM2.5` then we will just remove the following test stanza

```yaml
- as: acm2.5-interop-aws
  cron: 0 1 * * 1
  steps:
    cluster_profile: aws-cspi-qe
    env:
      BASE_DOMAIN: aws.interop.ccitredhat.com
      SUB_CHANNEL: release-2.5
      SUB_INSTALL_NAMESPACE: open-cluster-management
      SUB_PACKAGE: advanced-cluster-management
      SUB_SOURCE: redhat-operators
      SUB_TARGET_NAMESPACES: open-cluster-management
    test:
    - chain: interop-acm
    workflow: ipi-aws
```

While scenario expansions must have a passing rehearsal job for the PR to be merged, a deprecation does not.

## OpenShift Deployment

>This responsibility is abstracted away from both the CSPI-QE team and PQE teams to the workflow OWNERS. 

This responsibility falls on the owner of the workflow that is being used to deploy the OpenShift environment needed by the OpenShift Layered Product Scenario. Each workflow will most likely have a different owner. If you need support for a problem in a workflow you'll need to find the workflow that you are using in the [release repo's step-registry](https://github.com/openshift/release/tree/master/ci-operator/step-registry) and contact the users in the OWNERS file.

## Layered Product Deployment

>This is the responsibility of the PQE team.

The model being followed by this [layered product onboarding](../../Onboarding/Onboarding_Guide.md) is built on the idea that automation that is already created and being maintained should not be recreated by a different team attempting to achieve the same result. Therefore the onboarding of the scenario will ensure that the layered product deployment relies on automation built by that product's QE team. If there are failures in the OCP CI scenario for the layered product deployment then it will need to be fixed at the source, which will be the layered product QE team's repositories.

- If a product is able to be installed using the steps existing in the [operatorhub-subscribe ref](https://github.com/openshift/release/tree/master/ci-operator/step-registry/operatorhub/subscribe) than we can make use of that instead. 

> **NOTE:**
>
> Most likely there will be more to the product deployment then just the main operator install.

## Test Execution

>This is the responsibility of the PQE team.

Similar to the layered product deployment the code being used to setup and execute the tests will be coming from the product QE's test repositories. If there are tests failing for any reason the product QE team will need to fix the problem in their test repo.

## Triggering Cadence

>This is the responsibility of the CSPI-QE team.

If there is ever a change to the cadence of testing we must update the cron configuration for all scenarios. The current model is to trigger using the latest pre-GA OpenShift build on a weekly cadence each Monday morning.

See the [Triggering Guide](../../OCP_CI_Tutorials/Triggering/Triggering_Guide.md) for more information.

## Reporting Tests

>This is the responsibility of the CSPI-QE team.

This is part of the value that the CSPI-QE team provides. We will be responsible for consolidating reports for many layered product scenarios consumable by PM, leadership, engineering & QE. It will be the responsibility of the CSPI-QE team to ensure that the test results being generated by the layered products tests are reported quickly & accurately to all the appropriate places.

See the [Reporting Guide](../../OCP_CI_Tutorials/Scenarios/Reporting_Guide.md) for information into how reporting is done.

## Skipping Scenarios

>This is the responsibility of the CSPI-QE team.

TBD
