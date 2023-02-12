# Scenario Documentation Policy<!-- omit from toc -->

## Table of Contents<!-- omit from toc -->

- [Introduction](#introduction)
- [How to Document a Scenario](#how-to-document-a-scenario)
  - [Getting Started](#getting-started)
  - [Adding Documentation](#adding-documentation)
    - [General Information](#general-information)
    - [Purpose](#purpose)
    - [Process](#process)
    - [Custom Images](#custom-images)
- [Conclusion](#conclusion)

## Introduction

Documenting our scenarios is an important part of the CSPI onboarding and maintenance process. Ensuring we have a comprehensive document for each scenario will help current and future team members debug and maintain each scenario. Please follow this documentation policy to write helpful documentation for each new scenario.

## How to Document a Scenario

Each scenario written by CSPI within OpenShift CI should have a `README.md` file associated with it in the same folder as the scenario's configuration file. For example the MTR scenario's documentation can be found at `ci-operator/config/windup/windup-ui-tests/README.md` along with the configuration YAML file. Documentation should be written in Markdown. If you are unfamiliar with Markdown, please use [this list of resources](Markdown_Resources.md) to familiarize yourself with it.

### Getting Started

1. Create the `README.md` file in the same directory that holds the scenario's configuration YAML file.
   - **Note:** This file should be in the `ci-operator/config/{test-org}/{test-repo}/` directory within the [openshift/release](https://github.com/openshift/release) repository.
2. Add a level-1 header to the top of the file with the name of the scenario
   - **Note:** Please add `<!-- omit from toc -->` to the end of the header to avoid adding it to the Table of Contents.
   - **Example:** `# windup-windup-ui-tests-main<!-- omit from toc -->`
3. Add a level-2 header below the first header with the text "Table of Contents"
   - **Example:** `## Table of Contents <!-- omit from toc -->`
4. Add the Table of Contents
   1. In Visual Studio Code, make sure the [Markdown All in One](https://marketplace.visualstudio.com/items?itemName=yzhang.markdown-all-in-one) plugin is installed
   2. Navigate to the line below the "Table of Contents" header and press `CTRL+SHIFT+P` to bring up the Command Pallette.
   3. Type in `Markdown All in One: Create Table of Contents`
   4. Press enter
   - **Note:** This plugin will create the Table of Contents for you and keep it automatically updated with hyperlinks upon every save of the document. 

### Adding Documentation

The outline of these documents should generally follow this structure:

- General Information
- Purpose
- Process
  - Cluster Provisioning and Deprovisioning
  - Test Setup, Execution, and Reporting Results
- Custom Images

If you'd like to expand on this structure, please feel free to. But, this structure should be the minimum for each new scenario created.

#### General Information

This section of the documentation should include the following items:

- A link to the test repository
- Which layered product(s) is being tested
- Who maintains this scenario?

Please feel free to add any additional information or links that may be helpful in this section as well.

**Example:**

`ci-operator/config/windup/windup-ui-tests/README.md`

```markdown
## General Information

- **Repository**: [windup/windup-ui-tests](https://github.com/windup/windup-ui-tests)
- **Operator Tested**: [MTR (Migration Toolkit for Runtimes)](https://developers.redhat.com/products/mtr/overview)
- **Maintainers**: Interop QE
```

#### Purpose

This section of the documentation should outline the purpose of this scenario. While it may seem obvious to us, OpenShift CI is used by many teams and they may stumble across this scenario. It is important that people outside of CSPI and PQE know what this configuration does and why it is being run.

This section can be as short as a sentence or two:

`ci-operator/config/windup/windup-ui-tests/README.md`

```markdown
## Purpose

To provision the necessary infrastructure and using that infrastructure to execute MTR interop tests. The results of these tests will be reported to the appropriate sources following execution.
```

#### Process

This section of your documentation should describe how this scenario works. The bulk of the scenario's documentation will be done here. Be sure to provide a brief overview of the steps this scenario takes to run and why they are necessary.

##### Cluster Provisioning and Deprovisioning<!-- omit from toc -->

Under the "Process" header of your documentation be sure to include how and where the test cluster for this scenario is being provisioned and deprovisioned. 

This could be as simple as linking to a workflow being utilized to do this work as the workflow (and its documentation) may not be maintained by CSPI or PQE:

`ci-operator/config/windup/windup-ui-tests/README.md`

```markdown
### Cluster Provisioning and Deprovisioning: `ipi-aws`

Please see the [`ipi-aws`](https://steps.ci.openshift.org/workflow/ipi-aws) documentation for more information on this workflow. This workflow is not maintained by the Interop QE team.
```

##### Test Setup, Execution, and Reporting Results<!-- omit from toc -->

This section of the scenario documentation should document how the scenario preforms it's setup, test execution, and reporting following the test cluster being provisioned. This section may be as simple as listing the steps utilized and linking to their `README.md` files.

**Example:**

`ci-operator/config/windup/windup-ui-tests/README.md`

```markdown
## Test Setup, Execution, and Reporting Results - `mtr-interop-aws`

Following the test cluster being provisioned, the following steps are executed in this order:

1. [`mtr-install-chain`](../../../step-registry/mtr/install/README.md)
2. [`mtr-deploy-windup-ref`](../../../step-registry/mtr/deploy-windup/README.md)
3. [`mtr-execute-interop-ui-tests-chain`](../../../step-registry/mtr/execute-interop-ui-tests/README.md)
4. [`lp-interop-tooling-archive-results-ref`](../../../step-registry/lp-interop-tooling/archive-results/README.md)
```

#### Custom Images

This section of the documentation should outline any custom container images used during the scenario's execution. Under the "Custom Images" header, add a new header for each custom image. Under each custom image's header: explain what the image is, what it does, and a GitHub link to the Dockerfile that it is built from.

**Example:**

`ci-operator/config/windup/windup-ui-tests/README.md`

```markdown
## Custom Images

### `mtr-runner`

- [Dockerfile](https://github.com/windup/windup-ui-tests/blob/main/dockerfiles/interop/Dockerfile)

The custom image for this step uses the [`cypress/base`](https://hub.docker.com/r/cypress/base) image as it's base. The image should have all of the required dependencies installed and the [windup/windup-ui-tests repository](https://github.com/windup/windup-ui-tests) copied into `/tmp/windup-ui-tests`.
```

## Conclusion

Please make sure your documentation is helpful and thorough. Don't feel the need to outline each chain, ref, workflow, etc. used during the scenario's execution. Linking to [each step's documentation](Step_Registry_Documentation_Policy.md) should provide enough information for somebody to follow what is happening. That being said, it is good to outline the order in which these step's occur to execute your scenario successfully.
