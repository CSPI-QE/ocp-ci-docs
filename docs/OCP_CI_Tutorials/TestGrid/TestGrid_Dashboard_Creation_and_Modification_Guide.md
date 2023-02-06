# TestGrid Dashboard Creation and Modification Guide<!-- omit from toc -->

## Table of Contents<!-- omit from toc -->

- [Introduction](#introduction)
- [TestGrid Configuration Example and Description](#testgrid-configuration-example-and-description)
- [How to Create a New Dashboard](#how-to-create-a-new-dashboard)
- [How to Add a Job to a Dashboard](#how-to-add-a-job-to-a-dashboard)
- [Pull Request Process](#pull-request-process)

## Introduction

TestGrid is a Kubernetes community project that allow users to create dashboards of Prow job results. TestGrid uses its configuration files to build these dashboards, retrieve the Prow job results of all jobs defined in the dashboards, and displays the results in a grid pattern.

Please see the following resources for more information about TestGrid:

- [TestGrid Homepage](https://testgrid.k8s.io/)
- [TestGrid Documentation and Source Code](https://github.com/kubernetes/test-infra/tree/master/testgrid)

This guide's purpose is to give you instructions for **manually** creating or modifying a Dashboard in TestGrid.

> **IMPORTANT:**
>
> Creating or modifying should not be a normal process. Please see the [TestGrid section of the Reporting Guide](../Scenarios/Reporting_Guide.md#testgrid) for more information about how these changes should be automatically made.

## TestGrid Configuration Example and Description

`config/testgrids/openshift/redhat-openshift-4.12-interop.yaml`: 

```yaml
dashboards:
- name: redhat-openshift-4.12-interop
  dashboard_tab:
  - name: quay-3.7
    test_group_name: quay-3.7-openshift-4.12-interop
    open_test_template:
      url: https://prow.ci.openshift.org/view/gs/<gcs_prefix>/<changelist>
    results_url_template:
      url: https://prow.ci.openshift.org/job-history/<gcs_prefix>

test_groups:
- name: quay-3.7-openshift-4.12-interop
  gcs_prefix: origin-ci-test/logs/periodic-ci-rhpit-interop-tests-master-ocp-412-quay37-interop-quay-tests-aws-ipi-ocp412
  days_of_results: 60
```

- `dashboards`: A YAML stanza to describe a TestGrid dashboard.
  - `name`: The name of the dashboard. Please notice this name is the same as the filename.
  - `dashboard_tab`: A YAML stanza defining a list of "tabs" in a dashboard. Each of these items can be thought of as a single scenario.
    - `name`: The name of the "tab" or test. This should be more readable than the long Prow job name. In this example I am using "quay-3.7" rather than the long Prow job name defined later in the configuration.
    - `test_group_name`: Referring to the test group set up for a specific Prow job. See the `test_groups` stanza for the list of test groups.
    - `open_test_template`: Template for the URL linking to a specific test.
      - `url`: URL template. This should not change for our purposes.
    - `results_url_template`: Template for the URL linking to specific test results.
      - `url`: URL template. This should not change for our purposes.

- `test_groups`: A list of Prow jobs to be able to use in your dashboard.
  - `name`: Essentially an alias for the Prow job name. We use this name in the `test_group_name` key above. This value needs to be unique to not only this document, but other `test_groups` items in other config files.
  - `gcs_prefix`: The Google Cloud Storage prefix path for the Prow results. Should always be `origin-ci-test/logs/` plus the name of the Prow job.
  - `days_of_results`: How many days of results should be seen on TestGrid. I haven't seen this value go above 60 days, but I also haven't tried going any higher than that.

## How to Create a New Dashboard

Creating a new TestGrid dashboard is very simple. Before you get started you will need to [fork](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/working-with-forks/about-forks) the [kubernetes/test-infra](https://github.com/kubernetes/test-infra) repository into your personal GitHub organization.


Once you have your fork created, follow these instructions to create a new dashboard:

1. Prepare Git:
   1.  Clone a copy of your fork to your computer.
   2.  Create a new branch and name it whatever you'd like.
2. In your new branch, create a new file in the `config/testgrids/openshift` directory. The name of the file should be the same name as your new dashboard.
   - **Example:** `config/testgrids/openshift/redhat-openshift-4.12-interop.yaml`

> **IMPORTANT**:
>
> Do not use `-release-` at all in your file name or your dashboard name. This will break the OpenShift CI automation that generates their TestGrid dashboards. See this pull request for more information: [kubernetes/test-infra PR #28593](https://github.com/kubernetes/test-infra/pull/28593 )

3. Write your configuration file.
   - **Note:** Use the example in the [TestGrid Configuration Example and Description](#testgrid-configuration-example-and-description) section of this document.
4. After you have pushed your changes to your fork, follow the [pull request process](#pull-request-process) to submit a PR and have your dashboard created.

## How to Add a Job to a Dashboard

If there is already a suitable dashboard created in TestGrid to hold your new Prow job, follow this guide to get it appended to the existing dashboard. Before you get started you will need to [fork](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/working-with-forks/about-forks) the [kubernetes/test-infra](https://github.com/kubernetes/test-infra) repository into your personal GitHub organization.

1. Prepare Git:
   1.  Clone a copy of your fork to your computer.
   2.  Create a new branch and name it whatever you'd like.
2.  In your new branch, find the configuration file for the dashboard you'd like to use.
    - **Note**: The config files should be in the `config/testgrids/openshift` directory and the name of the file should be the same as the name on the TestGrid website.
    - **Example:** `config/testgrids/openshift/redhat-openshift-4.12-interop.yaml`
3. Create an entry in the `test_groups` stanza of the config file.
    - **Note:** Use the example in the [TestGrid Configuration Example and Description](#testgrid-configuration-example-and-description) section of this document.
    - **Note:** The `name` value in this item needs to be unique, not just in the document, but in the other configs as well. You may want to search the repo for the string you decide on to make sure it isn't used.
4. Create a new `dashboard_tab` entry in the `dashboards` stanza of the configuration file. 
5. After you have pushed your changes to your fork, follow the [pull request process](#pull-request-process) to submit a PR and have your dashboard created.

## Pull Request Process

This section is meant to outline the specifics around submitting a PR to the [kubernetes/test-infra](https://github.com/kubernetes/test-infra) repository for TestGrid configuration changes. This will not go over [how to create a pull request](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/proposing-changes-to-your-work-with-pull-requests/creating-a-pull-request) or [what it is](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/proposing-changes-to-your-work-with-pull-requests/about-pull-requests). 

The process for getting your pull request merged is mostly automated or handled by a maintainer in the repository.

1. Once you have [created the Pull request](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/proposing-changes-to-your-work-with-pull-requests/creating-a-pull-request) for your changes, the `linux-foundation-easycla` bot will comment on your PR.
   - If you have not signed the Linux Foundation CLA, it will give you a link to use to sign the CLA.
   - Use [the official documentation](https://github.com/kubernetes/community/blob/master/CLA.md) for instructions on signing this CLA.
2. The `k8s-ci-bot` will add a few labels to your PR and assign the PR to 2 community members for review. The two I got were both Red Hatters, so you may have the same experience.
3. Once one of the community members marks your PR as `ok-to-test`, your changes will be tested. If they pass all tests, it will be approved and merged in by the community member.
   - If the tests do not pass, make the necessary changes and test again until it passes.

> **NOTE:**
>
> Please take a look at [this pull request](https://github.com/kubernetes/test-infra/pull/28449) for an idea of how they work.

> **IMPORTANT:**
>
> If you need more help, please see the [official repository contribution guide](https://github.com/kubernetes/test-infra/blob/master/CONTRIBUTING.md).
