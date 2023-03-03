# Scenario Development Guide<!-- omit from toc -->

## Table of Contents<!-- omit from toc -->

- [Scenario Development Guide](#scenario-development-guide-1)
  - [Developing a Scenario](#developing-a-scenario)
    - [Getting Started](#getting-started)
    - [Write the Scenario](#write-the-scenario)
    - [Build a Container Image to Execute Tests](#build-a-container-image-to-execute-tests)
  - [Reporting](#reporting)
    - [TestGrid](#testgrid)
    - [Report Portal](#report-portal)
    - [Slack](#slack)
  - [Ephemeral Cluster Guide](#ephemeral-cluster-guide)
    - [How does this work?](#how-does-this-work)
    - [Important Workflows](#important-workflows)
  - [Multiple private repo environment (Images)](#multiple-private-repo-environment-images)
    - [So How do we solve this?](#so-how-do-we-solve-this)
  - [Scenario and Step Registry Documentation](#scenario-and-step-registry-documentation)
- [Developer's Guide](#developers-guide)
  - [Pull Request Process](#pull-request-process)
  - [Make Update](#make-update)
  - [Rehearsal Job](#rehearsal-job)
  - [Spot Instances](#spot-instances)
  - [Local Development](#local-development)
    - [OpenShift Local (CRC)](#openshift-local-crc)
    - [Hive](#hive)
    - [Quicklabs](#quicklabs)
    - [OpenShift CI Native Debugging](#openshift-ci-native-debugging)

## Scenario Development Guide

This guide is meant to explain our scenario development process and to point developers to the resources they may need during the development process. This guide does not serve as step-by-step instructions, but rather a set of guidelines and ideas to help make you successful in your development process.

### Developing a Scenario

#### Getting Started

1. Set up your environment
   1. If you haven't already, fork the [openshift/release](https://github.com/openshift/release) repository into your personal GitHub organization.
   2. Clone your fork to your computer using SSH (HTTP will not work).
   3. Create a new branch.
2. Create a directory (or directories) within the `ci-operator/config` directory, if it doesn't exist already.
   - OpenShift CI uses the test repositories GitHub organization and repository name to format these directories.
   - **Format:** `ci-operator/config/{GitHub organization}/{GitHub repository}`.
   - **Example:** For the MTR scenario, the tests live in the [windup/windup-ui-tests](https://github.com/windup/windup-ui-tests) repository. The directories for this scenario would be `ci-operator/config/windup/windup-ui-tests`.
     - If the scenario has multiple test repos then a central repo maintained by PQE can be used or other solutions to this should be discussed.
3. Create the scenario's configuration file in the directory created in the previous step.
   - OpenShift CI also uses the test repositories GitHub organization and repository name in the format of this filename. It also uses the desired branch name and allows you to add additional information.
   - **Format:** `{GitHub organization}-{GitHub repository}-{desired branch}__{product_short_name}-ocp4.{xx}-lp-interop.yaml`.
   - **Example:** For the MTR scenario's OCP 4.13 config file, `windup-windup-ui-tests-main__mtr-ocp4.13-lp-interop.yaml`.
   - **Note:** The `-lp-interop` value in the format is to allow us to report this job reliably. The use of unique identifier allows us to write automation to report our jobs easily as well as opens possibilities to easily manipulate our scenarios in the future if needed.
4. Create a directory in the `ci-operator/step-registry` directory for your product if one does not exist. The name of this folder should be the shortname of the layered product you are working on.
   - **Example:** For the MTR scenario, the directory's name is `ci-operator/step-registry/mtr`.
   - This folder will hold steps that are specific to your product and scenario, like a step to install the product and step to execute the test suite.

#### Write the Scenario

Now that you have the setup complete, you can start to write the scenario. First, you'll want to get a shell for your configuration file written. In the configuration file you created in step 2 of the [Getting Started](#getting-started) section, use the guide below to start writing your configuration file:

`some-mock-scenario-main__mock-ocp4.13-lp-interop.yaml`

```yaml
base_images:
  cli:
    name: "4.13"
    namespace: ocp
    tag: cli
build_root:
  image_stream_tag:
    name: release
    namespace: openshift
    tag: golang-1.19
images:
- context_dir: .
  dockerfile_path: dockerfiles/Dockerfile
  to: mock-runner
releases:
  latest:
    candidate:
      product: ocp
      stream: nightly
      version: "4.13"
tests:
- as: mock-scenario-aws
  cron: 0 1 * * 1
  steps:
    cluster_profile: aws-cspi-qe
    env:
      BASE_DOMAIN: aws.interop.ccitredhat.com
    test:
    - chain: mock-operator-install
    - ref: mock-execute-tests
    workflow: ipi-aws
```

**Brief description of these key/value pairs:**

> **IMPORTANT**
>
> The description for these key/value pairs is how we understand them right now. Unfortunately the official documentation for these values is not always clear. Please see the [official OpenShift CI documentation](https://steps.ci.openshift.org/ci-operator-reference) for more information, if needed.

- `base_images` : The list of base images describe which images are going to be necessary outside of the pipeline. The key will be the alias that other steps use to refer to this image.
- `build_root` : The image defined in this stanza is the image that will be used in the container that actually builds your test image defined in the `images` stanza.
- `images` : A list of custom images to use in your tests. You are able to use images from a registry, a Dockerfile, an image stream, etc. For CSPIs purposes, typically use a Dockerfile. See the [Container Creation Guide](../Containers/Container_Creation_Guide.md) for more information.
- `releases` : Releases maps semantic release payload identifiers to the names that they will be exposed under. For instance, an 'initial' name will be exposed as $RELEASE_IMAGE_INITIAL. The 'latest' key is special and cannot co-exist with 'tag_specification', as they result in the same output. See the [official OpenShift CI documentation](https://steps.ci.openshift.org/ci-operator-reference) for more information.
- `tests` : A list of tests to execute.
  - `as` : The name of a test.
  - `cron` : The schedule this test should run on. For CSPI purposes, it should be set to `0 1 * * 1`.
  - `steps` : A list of steps in this test object.
    - `cluster_profile` : Used during test cluster provisioning. The CSPI cluster profile for an AWS cluster would be `aws-cspi-qe`. Please don't use the profile if you are not in the CSPI organization as it will charge our organization for your cluster provisioning.
    - `env` : A list of environment variables needed for your tests to run. For each variable, use the following format: `{ENV_VAR_NAME}: {ENV_VAR_VALUE}`.
    - `test` : The list of chains and refs to execute, in order, in the test (scenario).
      - `chain` : Execute a chain, please see the [Step Registry - Chain Guide](../Step_Registry/Step_Registry_Chain_Guide.md) for more information.
      - `ref` : Execute a ref, please see the [Step Registry - Ref Guide](../Step_Registry/Step_Registry_Ref_Guide.md) for more information.
    - `workflow` : The workflow used to provision/deprovision our test cluster. See the [Step Registry - Workflow Guide](../Step_Registry/Step_Registry_Workflow_Guide.md) for more information on workflows. See the [Ephemeral Cluster Guide](#ephemeral-cluster-guide) section of this document for a list of workflows CSPI uses.

##### Scenario Structure Philosophy<!-- omit from toc -->

As CSPI has worked through how we would use OpenShift CI as our new OpenShift interop testing pipeline, we have gone through many iterations of how we planned on structuring our scenarios and our usage of the step registry. The structure proposed in this document is subject to change and is purposely written to be less rigid than the model our scenarios currently use.

When writing a scenario in the CSPI OpenShift CI pipeline, please use the following as a guideline:

1. **Do** write re-usable and useful steps in the step registry
   - As mentioned in step 4 of the [Getting Started](#getting-started) section above, each layered product should have it's own folder in the step registry. This folder should be populated by you with [steps](../Step_Registry/Step_Registry_Guide.md) that are specific to the layered product you are testing. Some examples could be:
     - A step to install the layered product from Operator Hub
     - A step to gather some important information to be used in test execution
     - A step to execute the tests themselves
   - When writing the steps, please make sure to make use of OpenShift CI variables, even if you think a variable isn't going to change often, make use of the default values of OpenShift CI variables. This allows other teams to utilize our steps and brings our work closer to the OpenShift organization.
2. **Do not** write monolithic steps in the step registry.
   - As you are planning your scenario, try to break the requirements up into sensible steps. 
   - Please try to keep in mind that we are trying to write re-usable steps, so writing one step to do _all_ of the setup necessary in one BASH script/ref would not be re-usable and would be difficult to maintain.
   - Please try to write the setup and execution of your scenario as modular as is reasonable. 
     - This allows for us to easily add new versions of your product to be tested or to use your setup steps in another scenario that may need them.

Please see the folder structure below as an example of the step registry folder for a layered product. In this example, we see the different steps that it takes to setup and execute the MTR scenario. For more information, see the [MTR scenario pull request](https://github.com/openshift/release/pull/36181/files):

- `install`: Used to install the MTR operator. This chain simply makes use of the `operatorhub-subscribe` ref that already exists, but populates the required variables with values for the MTR operator.
- `deploy-windup`: A small step that is required to be executed prior to the tests running, this step deploys Windup to the test cluster using variables defined in the ref. These variables have default values that can be overridden by others who may want to use the step.
- `retrieve-cluster-url`: In order for the tests to execute properly, they must target our test cluster. This step is used to retrieve the URL of that cluster and store it for later use in the `execute-***` steps.
- `execute-ui-tests`: A step to execute the test suite provided by PQE. These tests take a set of arguments that determines which tests are executed. By providing this step, we are leaving the door open for PQE to migrate this test-suite to OpenShift CI in the future very easily, if they would like to.
- `execute-interop-ui-tests`: This chain makes use of the `execute-ui-tests` step above and populates the variables with values that will ultimately result in only the interop tests being executed.

**MTR Step Registry Folder:**

```plaintext
├── deploy-windup
│   ├── mtr-deploy-windup-commands.sh
│   ├── mtr-deploy-windup-ref.metadata.json
│   ├── mtr-deploy-windup-ref.yaml
│   ├── OWNERS
│   └── README.md
├── execute-interop-ui-tests
│   ├── mtr-execute-interop-ui-tests-chain.metadata.json
│   ├── mtr-execute-interop-ui-tests-chain.yaml
│   ├── OWNERS
│   └── README.md
├── execute-ui-tests
│   ├── mtr-execute-ui-tests-commands.sh
│   ├── mtr-execute-ui-tests-ref.metadata.json
│   ├── mtr-execute-ui-tests-ref.yaml
│   ├── OWNERS
│   └── README.md
├── install
│   ├── mtr-install-chain.metadata.json
│   ├── mtr-install-chain.yaml
│   ├── OWNERS
│   └── README.md
├── OWNERS
└── retrieve-cluster-url
    ├── mtr-retrieve-cluster-url-commands.sh
    ├── mtr-retrieve-cluster-url-ref.metadata.json
    ├── mtr-retrieve-cluster-url-ref.yaml
    ├── OWNERS
    └── README.md
```

 We understand that the guidelines above are sparse and leave room for imagination, but that is what we are trying to do with this project. We would like to give the engineers writing scenarios more freedom to be creative and enable them to contribute useful steps to the OpenShift CI pipeline.

#### Build a Container Image to Execute Tests

Each scenario should have at least one container image to execute the tests within the test repository. Please follow the [Container Creation Guide](../Containers/Container_Creation_Guide.md) for help. Please keep in mind, the `Dockerfile` for this image should live inside the test repository. Please see the [MTR scenario's Dockerfile](https://github.com/windup/windup-ui-tests/blob/main/dockerfiles/interop/Dockerfile) as an example.

### Reporting

#### TestGrid

The majority (from what we can tell) of reporting in OpenShift CI is done through TestGrid. Luckily, the only thing we need to do to ensure this happens properly for our scenarios is include `-lp-interop` in the name of our configuration files (see step 3 of the [Getting Started](#getting-started) section above). This is covered for in depth in the [TestGrid section of the Reporting Guide](../Reporting/Reporting_Guide.md#testgrid)

#### Report Portal

<!--TODO: WRITE THIS WHEN A DECISION IS MADE ON REPORT PORTAL AND SLACK-->

#### Slack

<!--TODO: WRITE THIS WHEN A DECISION IS MADE ON REPORT PORTAL AND SLACK-->

### Ephemeral Cluster Guide

Here we will discuss perhaps the biggest value of OpenShift CI which is the amount of supported flavors of OpenShift Installations.

OpenShift CI hides the complexity of assembling an ephemeral OpenShift 4.x release payload, thereby allowing authors of end-to-end test suites to focus on the content of their tests and not the infrastructure required for cluster setup and installation.

#### How does this work?

We are able to leverage the work done by the OpenShift team through the use of workflows. See the [Step Registry - Workflow Guide](../Step_Registry/Step_Registry_Workflow_Guide.md) for more information about what workflows are and how to use them.

There are many of these workflows within the step-registry that we can choose from. We can easily view them by navigating to the [OpenShift CI Step Registry](https://steps.ci.openshift.org/#workflows) this page is auto-generated based on what exists in [openshift/release/ci-operator/step-registry](https://github.com/openshift/release/tree/master/ci-operator/step-registry)

Reading through the steps of the workflow to understand what it does and understand the vars that it needs is the first step when filling the need of automated OCP deployments.

#### Important Workflows

Below we'll list and provide some details about the usage of some workflows that we intend on using heavily.

##### `ipi-aws`<!-- omit from toc -->

step-registry doc: [ipi-aws](https://steps.ci.openshift.org/workflow/ipi-aws)

This workflow is going to serve as the base installation mechanism for our self-manage OpenShift installation. AWS is the first cloud provider that we are building out this testing on. As you can see in the link above this workflow is equipped with steps for deprovisioning which include must-gather that we get for free simply just by using this.

As many other OCP install workflows it will place the kubeconfig file and other information about the cluster in the $SHARED_DIR available to all containers.

We have the option to set an env variable in the config to use AWS spot instances to save costs in development.

```yaml
    env:
      SPOT_INSTANCES: "true"
```

> **IMPORTANT:**
> 
> Spot instances are not to be used in production.

##### Important Workflow.Next<!-- omit from toc -->

As this level of testing expands to different flavors of OCP installations we will cover future important workflows here for reference.

### Multiple private repo environment (Images)

You may run into a situation where PQE needs many different test environments to execute all of their tests. This becomes difficult when the images (read Dockerfiles) are stored in private github repos. We are not able to build all of these images from a single config file through the private repo access being offered by OpenShift CI. This is because the only private repo that we will have access to is the "src" repo (the repo that is associated with the directory naming convention). We also cannot just store all of the Dockerfiles into this one "src" repo and hope to git clone the files from the other private repos because a personal access token (PAT) would be required. If we tried using the [secrets functionality built into OpenShift CI](../Secrets/Secrets_Guide.md), the PAT would only be accessable during the execution of a ref, not during the image's build-process.

#### So How do we solve this?

We can promote images to the registry: `registry.ci.openshift.org`. Doing this makes the image available to be used as a base_image within OpenShift CI.

We do this by creating a [config file](https://github.com/openshift/release/pull/36522/files#diff-0bc957cbdffcb18de5385fa02b826cbdee5486f53f3cc9d06f371a658279c333) for each private test repo. We [specify the Dockerfile](https://github.com/openshift/release/pull/36522/files#diff-0bc957cbdffcb18de5385fa02b826cbdee5486f53f3cc9d06f371a658279c333R7) from that repo that we need to build an image from, then we [promote the image](https://github.com/openshift/release/pull/36522/files#diff-0bc957cbdffcb18de5385fa02b826cbdee5486f53f3cc9d06f371a658279c333R9) to registry.ci.openshift.org. The example here will be available at registry.ci.openshift.org/acm-qe/clc-ui-e2e:release-2.7. This allows us to use this image as a base_image in our scenario's config file (same as how any other base_image is being used). 

One major benefit that we get from doing this is that we know this image will have been pre-built and tested on any PR submitted by the PQE team. Setting up this promotion process automatically sets up a CI system for the image build in the test repo. If the image is failing to build, they will not be able to merge their code without explicitly skipping the automated check. Here's an [example PR](https://github.com/stolostron/clc-ui-e2e/pull/461) that triggered a [pre-submit job](https://prow.ci.openshift.org/view/gs/origin-ci-test/pr-logs/pull/stolostron_clc-ui-e2e/461/pull-ci-stolostron-clc-ui-e2e-release-2.7-images/1631327055738048512) to ensure that the image was built successfully. Once merged the post-submit job will run and promote the tested image to the registry where we can then make use of it.

Since these images will contain code from within a private repo, we need to make sure that unauthenticated users do not have access to these images. We prevent this by adding an [RBAC file](https://github.com/openshift/release/pull/36522/files#diff-e129896e392bc9ced499cb19ddf549b361784048825b25670755fc432f87f542) to only allow authenticated users to pull from the target registry namespace.

### Scenario and Step Registry Documentation

The current documentation for tests and steps in the step registry within OpenShift CI leaves a lot to be desired. The CSPI team has made it a priority to document our steps and tests in OpenShift CI thoroughly. As you are working on your scenario, please make sure to follow these documentation policies:

- [Scenario Documentation Policy](../../Policy/Documentation/Scenario_Documentation_Policy.md)
- [Step Registry Documentation Policy](../../Policy/Documentation/Step_Registry_Documentation_Policy.md)

These policies are not followed throughout the OpenShift CI platform, but CSPI will be enforcing them for and scenario or step written by CSPI. Your pull request **will not** be merged without up-to-date documentation. Hopefully as we populate this documentation, other OpenShift CI users will start to do the same.

## Developer's Guide

This section is meant to provide additional information and resources to anyone developing a scenario within OpenShift CI.

### Pull Request Process

We should always create a personal fork of the repo that we are submitting a PR to. Below is an example of how to do that (for the release repo).

1. Fork the [openshift/release](https://github.com/openshift/release) repository
2. Clone the forked repo using SSH (Do not use HTTP)
3. Create a new branch
4. Make your changes
5. Run [`make update`](#make-update) from the root of release repo. You may need to use `sudo` is some situations.
6. `git add` changed files
7. `git commit -m 'commit message'`
8. `git push origin {branch name}`
9. Submit PR from GitHub UI
10. Run a [rehearsal job](#rehearsal-job) (if needed).

### Make Update

Running `make update` provides many different checks to ensure your changes follow the standards of the release repo. When you fail the `make update` command you are given output describing where the problem lies and how to fix it.

If you create/update a ci-operator/config file it will:

- Create/update the Prow job for that specific config file (Prow jobs never need to be changed manually).
  - See [make jobs](https://docs.ci.openshift.org/docs/how-tos/onboarding-a-new-component/#generating-prow-jobs-from-ci-operator-configuration-files) docs which will run as part of the `make update` command.
- Create/update metadata and store it in the config file at the bottom of the file.

### Rehearsal Job

A rehearsal job is meant to execute a Prow job to prove that your changes are valid prior to merging. Not all PRs will require rehearsals, the `openshift-ci-robot` will comment on your PR alerting you that a rehearsable test has been affected by your change.

> **IMPORTANT:**
> Make sure that you review the affected jobs and only run the rehearsal if you know the jobs that will execute are the ones you are targeting. We don't want to actually run other teams jobs which will use their cloud infrastructure and accrue unnecessary costs.

Simply add a comment to your PR that says `/pj-rehearse` (pj = prow job) to trigger rehearsal jobs for each rehearsable job required in the pull request. Once the jobs start (after 1-2 min) you'll see the jobs in the automated checks at the bottom of the PR. If you'd like to trigger only one of the jobs, you can add the name of the Prow job as an argument to `/pj-rehearse`. Example: `/pj-rehearse periodic-ci-windup-windup-ui-tests-main-lp-interop-ocp4.13-mtr-interop-aws`

### Spot Instances

> **NOTE:**
>
> Spot instances are AWS specific at the time of writing this document.

Using spot instances is meant to save costs when we are developing a scenario. They are not meant to be used in Production. We have an easy way to take advantage of using [spot instances](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-spot-instances.html) when deploying OCP on AWS. We simply need to use the [cucushift-installer-rehearse-aws-ipi-spot](https://steps.ci.openshift.org/workflow/cucushift-installer-rehearse-aws-ipi-spot) workflow that uses spot instances rather then regular instances.

For example while we are developing a scenario we can use this workflow like we do in this tests stanza below.

```yaml
tests:
- as: mock-scenario
  cron: 0 1 * * 1
  steps:
    cluster_profile: aws-cspi-qe
    env:
      BASE_DOMAIN: aws.interop.ccitredhat.com
    test:
    - chain: mock-operator-install
    - ref: mock-execute-tests
    - ref: mock-archive-results
    workflow: cucushift-installer-rehearse-aws-ipi-spot
```

### Local Development

These are just some of the ways that have been identified to test and develop efficiently but its really up to all of us to learn and identify the best way to do this for the problem that we are trying to solve. We definitely have creative freedom here. We just have to make sure that time and budget isn't being wasted testing small changes in the large PR rehearsal job the deploys a cluster each time something small needs to be tested.

#### OpenShift Local (CRC)

When possible please execute your code on a local cluster using OpenShift local ([Getting Started Guide](https://access.redhat.com/documentation/en-us/red_hat_openshift_local/2.12/html/getting_started_guide/index)). 

This will save:

- **Time**: OpenShift local will come up faster and you will have it running longer, without needing to worry about how to keep up a cloud cluster through waits.
- **Money**: Since it runs local on your machine we are not accruing cloud costs for any of your development that can be done with OpenShift Local.

#### Hive

Deploy Openshift cluster using [Hive](https://gitlab.cee.redhat.com/-/snippets/5832) (saves cloud costs and can stay up for long periods of time). We can use this when we need to test product QE's stuff on a cluster that has multinode backing it.

#### Quicklabs

Deploy clusters using [quicklabs](https://quicklab.upshift.redhat.com/). Same use case as above just not using our openstack as env, limited scale.

#### OpenShift CI Native Debugging

Testing prow job specifics secrets, vars, and things like that can be done through a PR to the release repo, but a cluster deployment is not needed. So if you need to test that a secret that you are using is going to show up properly you can submit a PR strictly for debugging purposes to run the prow job and discover the necessary things that you need to do. Once you've solved for that small problem you can then integrate the solution back into the larger PR for your scenario.
