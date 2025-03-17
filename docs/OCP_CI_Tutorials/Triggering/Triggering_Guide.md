# OpenShift CI Scenario Triggering Guide<!-- omit from toc -->

## Table of Contents<!-- omit from toc -->

- [Overview](#overview)
- [Cadence](#cadence)
- [OpenShift Build](#openshift-build)
- [Weekly Trigger Job](#weekly-trigger-job)
- [Cron Config](#cron-config)
- [Gangway API](#gangway-api)
  - [How to Add Gangway API Triggering to a Scenario](#how-to-add-gangway-api-triggering-to-a-scenario)
  - [How to Trigger a Job Manually](#how-to-trigger-a-job-manually)
- [Job-re-trigger](#job-re-trigger)
  - [How to Set Up a Job Re-triggering app in AWS Lightsail](#how-to-set-up-a-job-re-triggering-app-in-aws-lightsail)
  - [Verify the running app](#verify-the-running-app)


## Overview

This guide is meant to cover the expected triggering cadence that will be used to generate test data for the layer product reports.

## Cadence

We will trigger on a weekly cadence. Specifically 6:00UTC on Mondays for self-managed OCP pre-GA testing. Rosa-sts-hypershift testing runs at 10:00 UTC on Mondays.

## OpenShift Build

We will be testing using nightly OpenShift pre-GA builds. Within the config files we specify the version of OCP by doing the following.

```yaml
releases:
  latest:
    candidate:
      product: ocp
      stream: nightly
      version: "4.14"
```

## Weekly Trigger Job

In order to allow the interop program to manage which scenarios are active/inactive on a weekly basis we needed to go beyond trigging each scenario using its cron value in the config.

Our trigger mechanism is now an independent job that will read a file from vault and trigger only those jobs which are set to `active: true` using the gangway API. 

- [trigger-jobs ref](https://github.com/openshift/release/tree/master/ci-operator/step-registry/trigger-jobs) holds the logic to read the jobs list from vault, check the gangway api and one by one trigger each scenario. (See README).
- [weekly trigger config](https://github.com/openshift/release/blob/master/ci-operator/config/rhpit/interop-tests/rhpit-interop-tests-main__weekly_trigger.yaml) is responsible for using the ref to create a prowjob meant to trigger at the time each specific testing is supposed to run.
  - The cron value here is the one that will determine when this testing runs.
- [Example job result](https://prow.ci.openshift.org/view/gs/origin-ci-test/logs/periodic-ci-rhpit-interop-tests-main-weekly_trigger-ocp-self-managed-layered-product-interop/1706186938068766720) - the [build-log.txt](https://gcsweb-ci.apps.ci.l2s4.p1.openshiftapps.com/gcs/origin-ci-test/logs/periodic-ci-rhpit-interop-tests-main-weekly_trigger-ocp-self-managed-layered-product-interop/1706186938068766720/artifacts/ocp-self-managed-layered-product-interop/trigger-jobs/build-log.txt) file here shows what the job read and the actions it took.
- [Example slack notification of job result](https://redhat-internal.slack.com/archives/C04PK4QPSR1/p1695621977083089)


## Cron Config

Now that we control when to run the scenario with the weekly trigger above, we needed to stop using the cron in the scenario config to trigger the weekly testing.

- We cannot simply remove the cron as that would make the job a pre-submit job rather then a periodic.
- We cannot just set the cron value to a date that doesn't exist or far into the future.
- We have settled on setting the cron to run once per year during company shutdown.
  - So each scenarios cron value should be as you see below.

```yaml
tests:
- as: {layered_product}-interop-aws
  cron: 0 6 25 12 *
```

## Gangway API

There is now functionality in OpenShift CI for jobs to be manually triggered using an API. This API is named [Gangway](https://docs.google.com/document/d/1PAYVOqQ9z4GlOkXqkfWLZRRdGcAzqT8329Wm9QksFYY/edit#) and it is very easy to add to a scenario and to use.

### How to Add Gangway API Triggering to a Scenario

The Gangway API functionality is now built into all jobs automatically in OpenShift CI so we no longer have to add specific code to our configs and jobs.

[Here is the PR](https://github.com/openshift/release/pull/40928) that was used to remove what was previously needed by jobs to support the API.

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
   - `**RUN_ID**` = The value of the `id` key in the API response above.

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

## Job Re-trigger

As part of our workflows, we have integrated the [RedHatQE/openshift-ci-job-trigger](https://github.com/RedHatQE/openshift-ci-job-trigger) tool to handle the automation for re-triggering and reporting.

The [job-re-trigger](https://steps.ci.openshift.org/reference/job-re-trigger) phase is based on this tool, which essentially wraps `gangway-api` requests for jobs that fails within a set of rules.

### How to Set Up a Job Re-triggering app in AWS Lightsail

In practice, the application currently served in the Openshift-Ci env is configured as an [AWS Lightsail](https://aws.amazon.com/lightsail/) instance, running continuously. 

Since the tool is based on the Flask framework, it is natural to deploy a Lightsail container and the configuration is quite easy, with respect to the tool's current configuration options.

Step by step deployment:

1. Sign in to your AWS account, go to the Lightsail control panel, and then go to `Containers`
2. Press `create container service` on the top right button
3. Inside the `Create Container Service` page, make sure the available region is automatically selected (usually the region closest to the user)
4. Choose a `Micro` power and a one node scale (can be changed later if necessary)
5. Name the container. We currently use `mpiit-ci-jobs-trigger`
6. Press `Create container service`. It will probably take a few minutes.
7. Once you can see the onboarded container in the Lightsail containers dashboard, go to it's `Deployments` section
8. Create a new deployment with the following configuration:

```commandline
Container name: ci-jobs-trigger
Image: quay.io/redhat_msi/ci-jobs-trigger
```

Add the following environment variables:

```commandline
[Key]:[Value]
FLASK_DEBUG: 1
CI_JOBS_TRIGGER_LISTEN_IP: 0.0.0.0
```

Open ports:

```commandline
[Port]:[Protocol]
5000: HTTP
```

9. Add a public endpoint corresponded to the container name, and change the Health check path to: `/healthcheck`
10. Hit `Save and deploy` and once the deployment is active, you're set to go!

For a further information-
See a doc about deploy [Flask apps on AWS Lightsail](https://aws.amazon.com/tutorials/serve-a-flask-app/).

### Verify the running app

For verify the app is working, make sure you:

- Set the server url path in Vault to the containers domain, should be something similar to:

```commandline
https://mpiit-ci-jobs-trigger.<identifier>.<aws-zone>.cs.amazonlightsail.com/openshift-ci-re-trigger
```

- Manually trigger the job `periodic-ci-rhpit-interop-tests-main-retrigger-poc-failure-test` using the gangway api. The job will send a Slack verification of the working app.
