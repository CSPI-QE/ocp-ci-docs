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
Documenting our scenarios is an important part of the CSPI onboarding and maintenance process. Ensuring we have a comprehensive document for each scenario will help future current and future team members debug and maintain each scenario. Please follow this documentation policy to write helpful documentation for each new scenario.

## How to Document a Scenario
Each scenario written by CSPI within OpenShift CI should have a `README.md` file associated with it in the same folder as the scenario's configuration file. For example the MTR scenario's documentation can be found at `ci-operator/config/windup/windup_integration_test/README.md` along with the configuration YAML file. Documentation should be written in Markdown. If you are unfamiliar with Markdown, please use [this list of resources](Markdown_Resources.md) to familiarize yourself with it.

### Getting Started
1. Create the `README.md` file in the same director that holds the scenario's configuration YAML file.
   - **Note:** This file should be in the `ci-operator/config/{test-org}/{test-repo}/` directory within the [openshift/release](https://github.com/openshift/release) repository.
2. Add a level-1 header to the top of the file with the name of the scenario
   - **Note:** Please add `<!-- omit from toc -->` to the end of the header to avoid adding it to the Table of Contents.
   - **Example:** `# windup-windup_integration_test-main<!-- omit from toc -->`
3. Add a level-2 header below the first header with the text "Table of Contents"
   - **Example:** `## Table of Contents <!-- omit from toc -->`
4. Add the Table of Contents
   1. In Visual Studio Code, make sure the [Markdown All in One](https://marketplace.visualstudio.com/items?itemName=yzhang.markdown-all-in-one) plugin is installed
   2. Navigate to the line below the "Table of Contents" header and press `CTRL+SHIFT+P` to bring up the Command Pallette.
   3. Type in `Markdown All in One: Create Table of Contents`
   4. Press enter
   - **Note:** This plugin will create the Table of Contents for you and keep it automatically updated with hyperlinks upon every save of the document. 

### Adding Documentation
The outline of these documents should generally follow the following structure:
- General Information
- Purpose
- Process
  - Cluster Provisioning and Deprovisioning
  - Orchestrate, Execute, and Report
- Custom Images

If you'd like to expand on this structure, please feel free to. But, this structure should be the minimum for each new scenario created.

#### General Information
This section of the documentation should include the following items:
- A link to the test repository
- Which layered product(s) is being tested
- Who maintains this scenario?

Please feel free to add any additional information or links that may be helpful in this section as well.

**Example:**

<sub><sup>`ci-operator/config/windup/windup_integration_test/README.md`</sup></sub>
```markdown
## General Information
- **Repository**: [windup/windup_integration_test](https://github.com/windup/windup_integration_test.git)
- **Operator Tested**: [MTR (Migration Toolkit for Runtimes)](https://developers.redhat.com/products/mtr/overview)
- **Maintainers**: CSPI QE
```

#### Purpose
This section of the documentation should outline the purpose of this scenario. While it may seem obvious to us, OpenShift CI is used by many teams and they may stumble across this scenario. It is important that people outside of CSPI and PQE know what this configuration does and why it is being run.

This section can be as short as a sentence or two:

<sub><sup>`ci-operator/config/windup/windup_integration_test/README.md`</sup></sub>
```markdown
## Purpose
To provision the necessary infrastructure and using that infrastructure to execute MTR interop tests. The results of theses tests should be reported to the appropriate sources following execution.
```

#### Process
This section of your documentation should describe how this scenario works. The bulk of the scenario's documentation will be done here. Be sure to provide a brief overview of the steps this scenario takes to run and why they are necessary.

##### Cluster Provisioning and Deprovisioning<!-- omit from toc -->
Under the "Process" header of your documentation be sure to include how and where the test cluster for this scenario is being provisioned and deprovisioned. 

This could be as simple as linking to a workflow being utilized to do this work as the workflow (and it's documentation) may not be maintained by CSPI or PQE:

<sub><sup>`ci-operator/config/windup/windup_integration_test/README.md`</sup></sub>
```markdown
### Cluster Provisioning and Deprovisioning: `ipi-aws`
Please see the [`ipi-aws`](https://steps.ci.openshift.org/workflow/ipi-aws) documentation for more information on this workflow. This workflow is not maintained by the CSPI QE team.
```

##### Orchestrate, Execute, and Report<!-- omit from toc -->
Scenario documentation should also document how the scenario preforms it's orchestration, test execution, and reporting. This part of the documentation may be short as we prefer to use a chain for each scenario that handle's most of this work. In that case, you may just be able to link to the chain created for this scenario.

**Example:**

<sub><sup>`ci-operator/config/windup/windup_integration_test/README.md`</sup></sub>
```markdown
### Orchestrate, Execute, and Report - `mtr-scenario`
All of the orchestration, test execution, and reporting for the interop MTR scenario is taken care of the by the [`interop-mtr`](../../../step-registry/interop/mtr/README.md) chain. All of the environment variables needed to execute this chain are passed to the chain using the `env` stanza. For more in-depth information on how the [`interop-mtr`](../../../step-registry/interop/mtr/README.md) chain and how it's components work, please see the README that is hyperlinked in this paragraph.
```

#### Custom Images
This section of the documentation should outline any custom container images used during the scenario's execution. Under the "Custom Images" header, add a new header for each custom image. Under each custom image's header: explain what the image is, what it does, and a GitHub link to the Dockerfile that it is built from.

**Example:**

<sub><sup>`ci-operator/config/windup/windup_integration_test/README.md`</sup></sub>
```markdown
## Custom Images

### `mtr-runner`
The `mtr-runner` image is a Python base image with all required packages for test execution installed along with the [windup/windup_integration_test](https://github.com/windup/windup_integration_test.git) repository copied into the `/tmp/integration_tests` directory. The image is used to execute the MTR interop tests.
```

## Conclusion
Please make sure your documentation is helpful and thorough. Don't feel the need to outline each chain, ref, workflow, etc. used during the scenario's execution. Linking to [each step's documentation](Step_Registry_Documentation_Policy.md) should provide enough information for somebody to follow what is happening. That being said, it is good to outline the order in which these step's occur to execute your scenario successfully.