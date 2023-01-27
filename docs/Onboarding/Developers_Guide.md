# OpenShift CI Developers Guide<!-- omit from toc -->

## Table of Contents<!-- omit from toc -->

- [Overview](#overview)
- [PR Process](#pr-process)
- [Make Update](#make-update)
- [Rehearsal Job](#rehearsal-job)
- [Spot Instances](#spot-instances)
- [OpenShift Local](#openshift-local)

## Overview

This document is meant to provide important information to anyone developing a scenario within OpenShift CI.

## PR Process

We should always create a personal fork of the repo that we are submitting a PR to. Below is an example of how to do that (for the release repo).

1. Fork the [openshift/release](https://github.com/openshift/release) repo
2. Git clone forked repo using SSH (Do not clone with HTTP)
3. Create branch
4. Make your changes
5. Run [make update](#make-update) from root of release repo. You may need to use `sudo` is some situations.
6. Git add changed files
7. Git commit -m 'commit message'
8. Git push origin {branch name}
9. Submit PR from UI
10. Run [rehearsal job](#rehearsal-job) (if needed).

## Make Update

Running `make update` provides many different checks to ensure your changes follow the standards of the release repo. When you fail the `make update` command you are given output describing where the problem lies and how to fix it.

If you create/update a ci-operator/config file it will:

- Create/update the Prow job for that specific config file (Prow jobs never need to be changed manually).
  - See [make jobs](https://docs.ci.openshift.org/docs/how-tos/onboarding-a-new-component/#generating-prow-jobs-from-ci-operator-configuration-files) docs which will run as part of the `make update` command.
- Create/update metadata and store it in the config file at the bottom of the file.

## Rehearsal Job

A rehearsal job is meant to execute a prow job to prove that your changes are valid prior to merging. Not all PRs will require rehearsals, the `openshift-ci-robot` will comment on your PR alerting you that a rhearsable test has been affected by your change.

> **IMPORTANT:**
> Make sure that you review the affected jobs and only run the rehearsal if you know the jobs that will execute are the ones you are targeting. We don't want to actually run other teams jobs which will use their cloud infrastructure and accrue unnecessary costs.

Simply add a comment to your PR that says `/pj-rehearse` (pj = prow job) to trigger the rehearsal job. Once the job starts (after 1-2 min) you'll see the job in the automated checks at the bottom of the PR.

## Spot Instances

This instruction is AWS specific for now.

Using spot instances is meant to safe costs when we are developing a scenario. They are not meant to be used in Production.

We have an easy way to take advantage of using [spot instances](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-spot-instances.html) when deploying OCP on AWS.

We simply need to use the [cucushift-installer-rehearse-aws-ipi-spot](https://steps.ci.openshift.org/workflow/cucushift-installer-rehearse-aws-ipi-spot) workflow that uses spot instances rather then regular instances.

For example while we are developing a scenario we can use this workflow like we do in this tests stanza below.

```yaml
tests:
- as: aws-spot-example
  cron: 0 1 * * 1
  steps:
    cluster_profile: aws-cspi-qe
    env:
      BASE_DOMAIN: aws.interop.ccitredhat.com
    workflow: cucushift-installer-rehearse-aws-ipi-spot
```

## OpenShift Local

When possible please execute your code on a local cluster using OpenShift local ([Getting Started Guide](https://access.redhat.com/documentation/en-us/red_hat_openshift_local/2.12/html/getting_started_guide/index)). 

This will save:

- **Time**: OpenShift local will come up faster and you will have it running longer, without needing to worry about how to keep up a cloud cluster through waits.
- **Money**: Since it runs local on your machine we are not accruing cloud costs for any of your development that can be done with OpenShift Local.
