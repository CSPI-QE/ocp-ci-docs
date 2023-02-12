# OpenShift CI Scenario Triggering Guide<!-- omit from toc -->

## Table of Contents<!-- omit from toc -->

- [Overview](#overview)
- [Cadence](#cadence)
- [OpenShift Build](#openshift-build)
- [Cron Config](#cron-config)

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
