# OpenShift CI Scenario Maintenance Policy<!-- omit from toc -->

## Table of Contents<!-- omit from toc -->

- [Overview](#overview)
- [Scenario Expansions](#scenario-expansions)
  - [Automatic Scenario Expansions (Preferred)](#automatic-scenario-expansions-preferred)
  - [Manual Scenario Expansions](#manual-scenario-expansions)
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

A scenario expansion can be defined as preparing your test scenario to execute tests against a newer layered product release than what was previously configured. For example, if the scenario executes tests for `oadp1.1 on OCP4.14` then an expansion would mean to update the config for this scenario to now test against `oadp1.2 on OCP4.14`

> **IMPORTANT:**
>
> As of right now the CSPI-QE team who is responsible for this interop testing program will ONLY be testing the lastest GA version of the layered product. 
> 
> This means that moving to the latest GA version will be dropping test coverage for the previous version.

### Automatic Scenario Expansions (Preferred)

This automated expansion is made possible only when the `!default` channel is used to install your operator.

Requirements

- Product is an operator
- install-operators config uses the `!default` channel
- base_images do not need to be updated to test newest layered product version
- Other scenario vars specific to the test ref do not need to be updated to test the newest layered product version.

Here's a good example to show to give you some things to think about

[3scale scenario config file](https://github.com/openshift/release/blob/0678141fe8d952b170b71e0217235536300e2499/ci-operator/config/3scale-qe/3scale-deploy/3scale-qe-3scale-deploy-main__3scale-amp-ocp4.15-lp-interop.yaml#L57)

Here we use the `!default` channel to determine which version of the operator we want to install using the [install-operators ref](https://steps.ci.openshift.org/reference/install-operators).

```yaml
      OPERATORS: |
        [
            {"name": "3scale-operator", "source": "redhat-operators", "channel": "!default", "install_namespace": "threescale", "target_namespaces": "threescale", "operator_group":"threescale-operator-group"}
        ]
```

This means the product is an operator, and uses the default channel.

Now let's take a look at the [base_images](https://github.com/openshift/release/blob/0678141fe8d952b170b71e0217235536300e2499/ci-operator/config/3scale-qe/3scale-deploy/3scale-qe-3scale-deploy-main__3scale-amp-ocp4.15-lp-interop.yaml#L1)

```yaml
base_images:
  cli:
    name: "4.15"
    namespace: ocp
    tag: cli
  test-image:
    name: 3scale-interop-tests
    namespace: ci
    tag: v2.13
```

Here we see the cli image is related to ocp 4.15 that is fine here and won't effect the run if we test against a newer layered product image.

But now we see the `test-image` has a version specific tag. This means that we have mirrored an image into the internal registry with code specific to `v2.13` this now could be a problem. Ideally we would have used a test image with a `latest` tag so that we know we are always using test code that is up to date with the latest GA version of the product.

We now have the risk that the automatic expansion will take place (i.e. the `!default` channel is updated to be a newer version of the product) when this happens the test suite image labeled `v2.13` will be used to test the product. this could be a problem if the product is now `v2.14`. An investigation from PQE would be needed here to see if they needed to update their image once the default channel changes.

### Manual Scenario Expansions

Here we have the `tests:` section of a scenario config for the ACM scenario.

```yaml
tests:
- as: oadp-interop-aws
  cron: 0 6 * * 1
  steps:
    cluster_profile: aws-cspi-qe
    env:
      BASE_DOMAIN: aws.interop.ccitredhat.com
      OPERATORS: |
        [
          {"name": "redhat-oadp-operator", "source": "redhat-operators", "channel": "stable-1.1", "install_namespace": "openshift-adp", "target_namespaces": "openshift-adp", "operator_group":"oadp-operator-group"},
        ]
    test:
    - ref: install-operators
    workflow: ipi-aws
```

This is showing us that the layered product is being installed using the channel `stable-1.1` when the install-operator ref is run.

If our goal is to expand the scenario so that we update to test `oadp 1.2` then we need to replace the channel `stable-1.1` with `stable-1.2`. So now we will have.

```yaml
tests:
- as: oadp-interop-aws
  cron: 0 6 * * 1
  steps:
    cluster_profile: aws-cspi-qe
    env:
      BASE_DOMAIN: aws.interop.ccitredhat.com
      OPERATORS: |
        [
          {"name": "redhat-oadp-operator", "source": "redhat-operators", "channel": "stable-1.1", "install_namespace": "openshift-adp", "target_namespaces": "openshift-adp", "operator_group":"oadp-operator-group"},
        ]
    test:
    - ref: install-operators
    workflow: ipi-aws
```

This example has been taken from the [oadp scenario config here](https://github.com/openshift/release/blob/dc564ebcb5b723f91ffb81f55285d4b5ca7ee5fc/ci-operator/config/oadp-qe/oadp-qe-automation/oadp-qe-oadp-qe-automation-main__oadp1.2-ocp4.15-lp-interop.yaml)

We've now updated the layered product version that will be installed but now we need to check to make sure the right version of the tests will be used.

Here's the base_image section of the config

```shell
base_images:
  mtc-python-client:
    name: mtc-python-client
    namespace: mtc-qe
    tag: main
  oadp-apps-deployer:
    name: oadp-apps-deployer
    namespace: oadp-qe
    tag: main
  oadp-e2e-qe:
    name: oadp-e2e-qe
    namespace: oadp-qe
    tag: release-v1.1
```

We see 3 images. 2 of them use the tag `main` which will be fine so long as the main tag will always represent the image meant for the latest GA version. The 3rd image we see uses the tag release-v1.1, if we just change the install version to `stable-1.2` like we did above and do not change this image there is likely to be problems.

So in this case we would need to make sure we [mirror](../../OCP_CI_Tutorials/Scenario_Development/Scenario_Development_Guide.md#mirror-quay-images) or [promote](../../OCP_CI_Tutorials/Scenario_Development/Scenario_Development_Guide.md#how-to-promote-images-to-the-registry) a new image `release-v1.2` to the internal registry so that our expanded scenario is valid.

> **IMPORTANT:**
>
> We need to be diligent when checking for any product/test version mismatches. We cannot have tests running against oadp 1.2 that only work for oadp 1.1.
>
> Also we must ensure that a rehearsal job is run and passes based on the new expansion code `/pj-rehearse <job name>` from within your PR.

## Scenario Deprecations

> This is the responsibility of the PQE team.

**Currently under evaluation**:
When a layered product version needs to be expanded to test the latest GA we also need to stop testing the older version. This is due to resource and budget limitations. A scenario deprecation will just involve updating the config to the newer version. This will in turn deprecate testing on the older layered product version

 
## OpenShift Deployment

>This responsibility is abstracted away from both the CSPI-QE team and PQE teams to the workflow OWNERS. 

This responsibility falls on the owner of the workflow that is being used to deploy the OpenShift environment needed by the OpenShift Layered Product Scenario. Each workflow will most likely have a different owner. If you need support for a problem in a workflow you'll need to find the workflow that you are using in the [release repo's step-registry](https://github.com/openshift/release/tree/master/ci-operator/step-registry) and contact the users in the OWNERS file.

## Layered Product Deployment

>This is the responsibility of the PQE team.

The model being followed by this [layered product onboarding](../../Onboarding/Onboarding_Guide.md) is built on the idea that automation that is already created and being maintained should not be recreated by a different team attempting to achieve the same result. Therefore the onboarding of the scenario will ensure that the layered product deployment relies on automation built by that product's QE team. If there are failures in the OCP CI scenario for the layered product deployment then it will need to be fixed at the source, which will be the layered product QE team's repositories.

- If a product is able to be installed using the steps existing in the [install-operators ref](https://steps.ci.openshift.org/reference/install-operators) than we can make use of that instead. 

> **NOTE:**
>
> Most likely there will be more to the product deployment then just the main operator install.

## Test Execution

>This is the responsibility of the PQE team.

Similar to the layered product deployment the code being used to setup and execute the tests will be coming from the product QE's test repositories. If there are tests failing for any reason the product QE team will need to fix the problem in their test repo.

## Triggering Cadence

>This is the responsibility of the CSPI-QE team.

If there is ever a change to the cadence of testing we must update the [cron based trigger job](https://github.com/openshift/release/blob/master/ci-operator/config/rhpit/interop-tests/rhpit-interop-tests-main__weekly_trigger.yaml#L14). The current model is to trigger on a weekly cadence each Monday morning.

See the [Triggering Guide](../../OCP_CI_Tutorials/Triggering/Triggering_Guide.md) for more information.

## Reporting Tests

>This is the responsibility of the CSPI-QE team.

This is part of the value that the CSPI-QE team provides. We will be responsible for consolidating reports for many layered product scenarios consumable by PM, leadership, engineering & QE. It will be the responsibility of the CSPI-QE team to ensure that the test results being generated by the layered products tests are reported quickly & accurately to all the appropriate places.

See the [Reporting Guide](../../OCP_CI_Tutorials/Scenarios/Reporting_Guide.md) for information into how reporting is done.

## Skipping Scenarios

>This is the responsibility of the CSPI-QE team.

With the introduction of a trigger job we now are able to skip scenario's easily by updating our [JSON_TRIGGER_LIST](https://github.com/openshift/release/blob/master/ci-operator/config/rhpit/interop-tests/rhpit-interop-tests-main__weekly_trigger.yaml#L24) file that we store in vault.

This file holds a list of all job names for our program and a value `active` which will be either true or false. When we need to skip a scenario for whatever reason we just need to edit the file in vault to turn `active: true` to `active: false`.

This action is restricted to only a handful of people in the CSPI-QE org.
