# (WIP) OpenShift CI Scenario Provision Guide<!-- omit from toc -->

## Table of Contents<!-- omit from toc -->
- [Overview](#overview)
- [Create Foundational OpenShift CI Scenario Files](#create-foundational-openshift-ci-scenario-files)
  - [1. Create a directory within the config directory](#1-create-a-directory-within-the-config-directory)
  - [2. Create directories for the scenario in the step-registry](#2-create-directories-for-the-scenario-in-the-step-registry)
  - [3. Create the Scenario Chain](#3-create-the-scenario-chain)
  - [4. Create Config in Directory from Step #1.](#4-create-config-in-directory-from-step-1)
    - [OCP Image Version](#ocp-image-version)
    - [Test stanza creation](#test-stanza-creation)
  - [5. Submit PR](#5-submit-pr)

## Overview
Here we will be submitting our first PR to the [openshift/release](https://github.com/openshift/release) repo.

We will create the file structure that is common to all interop scenarios.
The result will be a prow job that is capable of deploying an OCP cluster.
## Create Foundational OpenShift CI Scenario Files
To make sure you start out correctly please see the [Scenario Developers Guide](../../Onboarding/Developers_Guide.md)
### 1. Create a directory within the config directory
    ci-operator/config/{organization}/{repository}
- **organization**: Github organization name that the test repository belongs to.
- **repository**: Test repository from org above.
### 2. Create directories for the scenario in the step-registry
    ci-operator/step-registry/interop/{product_name}
    ci-operator/step-registry/interop/{product_name}/orchestrate/
    ci-operator/step-registry/interop/{product_name}/execute/
    ci-operator/step-registry/interop/{product_name}/report/
- **product_name**: The shortname of the product under test.

### 3. Create the Scenario Chain
    ci-operator/step-registry/interop/{product_name}/interop-{product_name}-chain.yaml
- **product_name**: The shortname of the product under test.
```
chain:
  as: interop-{product_name}
  steps:
  - ref: operatorhub-subscribe
  - ref: interop-{product_name}-orchestrate
  documentation: |-
    Runs the {product_name} interop scenario
```

### 4. Create Config in Directory from Step #1.
    ci-operator/config/{organization}/{repository}/{organization}-{repository}-{branch}_{product_short_name}-ocp4.{xx}-interop.yaml

Copy and Paste the [template](https://github.com/openshift/release/blob/master/ci-operator/config/rhpit/interop-tests/rhpit-interop-tests-master__installer-rehearse-4.12.yaml) into the file that you've created.

#### OCP Image Version
- TODO populate this section describing image version selection once in place.


#### Test stanza creation
Now you will need to make changes to the following to make this file specific to your scenario.

```
tests:
- as: {product_name}-interop-aws
  cron: 0 1 * * 1
  steps:
    cluster_profile: aws-cspi-qe
    env:
      BASE_DOMAIN: aws.interop.ccitredhat.com
    workflow: aws-ipi
```
- **product_name**: The shortname of the product under test.

### 5. Submit PR
See [PR process](../../Onboarding/Developers_Guide.md#pr-process)