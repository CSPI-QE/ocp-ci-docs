# OpenShift CI Scenario Triggering Guide<!-- omit from toc -->

## Table of Contents<!-- omit from toc -->

- [Overview](#overview)
- [Cadence](#cadence)
- [OpenShift Build](#openshift-build)
- [Cron Config](#cron-config)
- [Gangway API](#gangway-api)
  - [How to Add Gangway API Triggering to a Scenario](#how-to-add-gangway-api-triggering-to-a-scenario)
  - [How to Trigger a Job Manually](#how-to-trigger-a-job-manually)

## Overview

This guide is meant to cover the expected triggering cadence that will be used to generate test data for the layer product reports.

## Cadence

We will trigger on a weekly cadence. Specifically 1:00am on Mondays.

## OpenShift Build

We will be testing using nightly OpenShift pre-GA. Within the config files we specify the version of OCP by doing the following.

```yaml
releases:
  initial:
    candidate:
      product: ocp
      stream: nightly
      version: "4.13"
  latest:
    candidate:
      product: ocp
      stream: nightly
      version: "4.13"
```

## Cron Config

We use the cron option under the ci-operator test stanza to configure the jobs that make up this layered product testing to run on this weekly cadence.

```yaml
tests:
- as: {layered_product}-interop-aws
  cron: 0 1 * * 1
```

We see here the cron is set to Monday at 1am to trigger this specific test. Once a config holding this code is merged into the release repo the cron becomes active.

## Gangway API

There is now functionality in OpenShift CI for jobs to be manually triggered using an API. This API is named [Gangway](https://docs.google.com/document/d/1PAYVOqQ9z4GlOkXqkfWLZRRdGcAzqT8329Wm9QksFYY/edit#) and it is very easy to add to a scenario and to use.

### How to Add Gangway API Triggering to a Scenario

1. Open the configuration file for the scenario you'd like to add this functionality to. It can be found in the [openshift/release](https://github.com/openshift/release) repository under `ci-operator/config/....`
2. For each test stanza in the `tests:` stanza, add `remote_api: true`. Example:

   ```yaml
   tests:
   - as: mtr-interop-aws
     cron: 0 6 * * 1
     remote_api: true
     steps:
       cluster_profile: aws-cspi-qe
       env:
         BASE_DOMAIN: cspilp.interop.ccitredhat.com
         MTR_TESTS_UI_SCOPE: interop
         OPERATORS: |
           [
               {"name": "mtr-operator", "source": "redhat-operators", "channel": "alpha", "install_namespace": "mtr", "operator_group":"mtr-operator-group", "target_namespaces": "mtr"}
           ]
       test:
       - ref: install-operators
       - ref: mtr-deploy-windup
       - ref: mtr-tests-ui
       workflow: ipi-aws
   ```

3. Execute the `make update` command

### How to Trigger a Job Manually

> **IMPORTANT**
>
> At the time of writing this, only a few people have access to the API key in Vault. This may change, but if you do not have access, ask Adam, Caleb, Badre, or Chetna to execute the API command using the key if you need a manual rerun.

1. Open the [OpenShift CI Vault](https://vault.ci.openshift.org/ui/vault/secrets) and login using OIDC
2. Navigate to `kv/cspi-qe/gangway-api` and copy the `token` value
3. Execute the following command in a command prompt, replacing the values below appropriately:
   - `**OCPCI-JOB-NAME**` = The prow job name of the scenario you'd like to run. This can be found in the [openshift/release](https://github.com/openshift/release) repository under `ci-operator/jobs/....`. It is also the same value as the job name that shows in TestGrid or Sippy
   - `**TOKEN**` = The token value found in step 2

   ```bash
   curl -X POST -d '{"job_execution_type": "1"}' -H "Authorization: Bearer **TOKEN**" https://gangway-ci.apps.ci.l2s4.p1.openshiftapps.com/v1/executions/**OCPCI-JOB-NAME**
   ```

   The command above should return something like this:

   ```json
   {
   "id": "f0d18d2f-eb7c-11ed-88e0-0a580a81022a",
   "job_name": "**OCPCI-JOB-NAME**",
   "job_type": "PERIODIC",
   "job_status": "TRIGGERED",
   "gcs_path": ""
   }
   ```

4. To check the status of a job, execute the following command prompt, replacing the values below appropriately:
   - `**TOKEN**` = The token value found in step 2
   - `**RUN_ID**`: The value of the `id` key in the API response above.

   ```bash
   curl -X GET -H "Authorization: Bearer **TOKEN**" https://gangway-ci.apps.ci.l2s4.p1.openshiftapps.com/v1/executions/**RUN_ID**
   ```

   The command above shoudl return something like this:

   ```json
   {
    "id": "f0d18d2f-eb7c-11ed-88e0-0a580a81022a",
    "job_name": "",
    "job_type": "JOB_EXECUTION_TYPE_UNSPECIFIED",
    "job_status": "SUCCESS",
    "gcs_path": ""
   }
   ```
