# Reporting Guide<!-- omit from toc -->

## Table of Contents<!-- omit from toc -->

- [TestGrid](#testgrid)
  - [What is TestGrid?](#what-is-testgrid)
  - [How do I Report Jobs to TestGrid?](#how-do-i-report-jobs-to-testgrid)
  - [TestGrid Dashboard Creation and Modification Automation](#testgrid-dashboard-creation-and-modification-automation)
- [Slack](#slack)
  - [How to Setup Slack Alerts for a Scenario](#how-to-setup-slack-alerts-for-a-scenario)
- [Failure Handling (Jira)](#failure-handling-jira)
  - [How Failures Are Reported to Jira](#how-failures-are-reported-to-jira)
    - [Example](#example)
  - [How To Add Jira Reporting to a Scenario](#how-to-add-jira-reporting-to-a-scenario)
- [Component Readiness](#component-readiness)
  - [General Information](#general-information)
  - [Sippy](#sippy)
  - [CI Test Mapping](#ci-test-mapping)

## TestGrid

### What is TestGrid?

TestGrid is a Kubernetes community project that allows users to create dashboards of Prow job results. TestGrid uses its configuration files to build these dashboards, retrieve the Prow job results of all jobs defined in the dashboards, and displays the results in a grid pattern.

Please see the following resources for more information about TestGrid:

- [TestGrid Homepage](https://testgrid.k8s.io/)
- [TestGrid Documentation and Source Code](https://github.com/kubernetes/test-infra/tree/master/testgrid)

### How do I Report Jobs to TestGrid?

We have been able to eliminate a manual process for reporting jobs to TestGrid. The most important thing to know about how jobs are reported to the `-lp-interop` dashboards in TestGrid is that the automation that makes it happen looks for the unique identifier `-lp-interop` in the name of the Prow job.

As you may have read in other documents in this repository, you will need to append `-lp-interop` to the end of of your configuration file's filename when you create it. When we create configuration files in OpenShift CI, as you may know, the format for the filenames is `{GitHub Organization}-{GitHub Repository}-{Branch}__****.yaml`. After the `{Branch}` section of the filename, anything can be appended to the end. The text appended to the end is called a "variant". The use of variants will come in handy if we test multiple versions of a layered product or of OpenShift. Please add `-lp-interop` to the "variant" section of the filename.

**Examples:**

- `ci-operator/config/windup/winup-ui-tests/windup-windup-ui-tests-main-lp-interop.yaml`: Will be reported because `-lp-interop` is in the filename.
- `ci-operator/config/windup/winup-ui-tests/windup-windup-ui-tests-main.yaml`: Will NOT get reported because `-lp-interop` is not found in the filename.
- `ci-operator/config/windup/winup-ui-tests/windup-windup-ui-tests-main__ocp412-mtr-123-lp-interop.yaml`: Will be reported because `-lp-interop` is found in the filename.

### TestGrid Dashboard Creation and Modification Automation

The automation we use to automatically create and modify dashboards in TestGrid can be found in the [openshift/ci-tools](https://github.com/openshift/ci-tools) repository. We utilize the [testgrid-config-generator] tool in that repository to find any Prow jobs that contain either `-lp-interop` in their names. If a new Prow job is found that isn't being reported, the tool will create a new dashboard or modify an existing dashboard to report that job to TestGrid appropriately. After the tool has run, it will create a pull request in the [kubernetes/test-infra][kubernetes-test-infra] repository to finalize the changes.

The [testgrid-config-generator] tool is run daily and you should not need to force it to run. After the tools runs, it may take some time for the pull request to be merged into the [kubernetes-test-infra] repository. Once the pull request is merged, it will start to show in TestGrid.

To add support for automatically detecting layered product interoperability jobs, a [PR](https://github.com/openshift/ci-tools/pull/3289) was opened to [testgrid-config-generator] support these unique identifier.

> **NOTE:**
>
> The only Prow jobs that will be automatically reported in TestGrid are the jobs in the `main` branch of the [openshift/release](https://github.com/openshift/release).

## Slack

[OpenShift CI allows to set up Slack alerts](https://docs.ci.openshift.org/docs/how-tos/notification/) for our scenarios. The CSPI Interop team has decided that we should set up this Slack integration for each of our scenarios. Each scenario should alert to the Slack channel that product QE decides. The channel must be public and in redhat-internal.slack.com.

### How to Setup Slack Alerts for a Scenario

1. In the [openshift/release](https://github.com/openshift/release) repository, after you have created a configuration file for your scenario in the `ci-operator/config/...` directory and ran the `make update` or `make jobs` command, you should be able to find a `job` file for your config generated under `ci-operator/jobs/....`. Find the job file that ends in `-periodics.yaml` and open it.
2. This file may contain multiple periodic jobs for the same repository, so find the periodic job that matches the config you'd like alerts for. If you are working with layered product interop testing, the name should include `-lp-interop`. In this example, the job's name is `periodic-ci-windup-windup-ui-tests-v1.0-mtr-ocp4.13-lp-interop-mtr-interop-aws`.
3. Add a reporter_config stanza, replace the `channel:` value with the channel you're PQE team would like to use and update the `report_template:` with a different message (if you'd like to, this one is very generic and will work in most cases):

```yaml
  name: periodic-ci-windup-windup-ui-tests-v1.0-mtr-ocp4.13-lp-interop-mtr-interop-aws
  reporter_config:
    slack:
      channel: '#mtr-interop'
      job_states_to_report:
      - success
      - failure
      - error
      report_template: '{{if eq .Status.State "success"}} :slack-green: Job *{{.Spec.Job}}*
        ended with *{{.Status.State}}*. <{{.Status.URL}}|View logs> {{else}} :failed:
        Job *{{.Spec.Job}}* ended with *{{.Status.State}}*. <{{.Status.URL}}|View
        logs> {{end}}'
```

4. Commit your changes and open a Pull Request

> **IMPORTANT**
>
> Please see the [official documentation](https://docs.ci.openshift.org/docs/how-tos/notification/) for more information about how to configure Slack alerts further.


[testgrid-config-generator]: https://github.com/openshift/ci-tools/tree/master/cmd/testgrid-config-generator
[kubernetes-test-infra]: https://github.com/kubernetes/test-infra

## Failure Handling (Jira)

### How Failures Are Reported to Jira

Failures are reported to Jira using the [firewatch tool](https://github.com/CSPI-QE/firewatch). This tool is used to react to failures in OpenShift CI jobs. This tool uses a configuration defined for each scenario to help it determine how it should report certain bugs. For a more technical understanding of how to use the tool and build the configuration properly, please see the documentation below:

- [Getting started](https://github.com/CSPI-QE/firewatch/blob/main/README.md)
- [How to define the configuration](https://github.com/CSPI-QE/firewatch/blob/main/docs/cli_usage_guide.md#defining-the-configuration)

#### Example

For the purposes of how this automation works, here is a fairly simple example:

Each job/scenario in OpenShift CI consists of different steps, for this example we will say our scenario has three steps: `setup`, `test`, and `teardown`. In each of these steps, as far as the automation is concerned, there are two types of failures: `pod_failure` which means the step's script exited with a non-zero exit code and `test_failure` which means the step generated a JUnit XML file and a failure in the XML was found.

For this example, lets set some plain-english rules to make it a little easier to understand:

- If there is any type of failure in the `setup` step, report it to Interop QE (INTEROP Jira project)
- If there is any type of failure in the `teardown` step, report it to Interop QE (INTEROP Jira project)
- If there is a `pod_failure` found in the `test` step, report it to Interop QE (INTEROP Jira project)
- If there is a `test_failure` found in the `test` step, report it to Product QE (PQE Jira project)

Using the logic outlined above, we can generate a firewatch config that will result in bugs being filed to the right teams with as much information as possible to help the engineer looking at the bug. The configuration for this logic would look something like this:

```json
{
"failure_rules":
[
  {"step": "setup", "failure_type": "all", "classification": "Lorem Ipsum", "jira_project": "INTEROP", "group": {"name": "cluster", "priority": 1}, "jira_additional_labels": ["!default"]},
  {"step": "test", "failure_type": "pod_failure", "classification":  "Lorem Ipsum", "jira_project": "INTEROP", "group": {"name": "lp-tests", "priority": 1}, "jira_additional_labels": ["!default", "interop-tests"]},
  {"step": "test", "failure_type": "test_failure", "classification":  "Lorem Ipsum", "jira_project": "PQE", "group": {"name": "lp-tests", "priority": 1}, "jira_additional_labels": ["!default", "interop-tests"]},
  {"step": "teardown", "failure_type": "all", "classification": "Lorem Ipsum", "jira_project": "INTEROP", "group": {"name": "cluster", "priority": 2}, "jira_additional_labels": ["!default"]}
]
}
```

For the sake of this documentation, we will not go very deep into this configuration (again, see the documentation linked above) but this configuration will result in the plain-english rules we outlined earlier. Here is a highly-simplified flowchart of how this works:

```mermaid

flowchart TD
  failure[Test failure in setup step]
  type(Failure type determination)
  rule(Does the step and failure type combo match a rule?)
  matches(Yes)
  no_match(No)
  file_bug(File a bug in the Jira project defined in the rule)
  file_generic_bug(File a bug in the default Jira project)
  comment(Comment on duplicate bug with failure information)

  failure --> type
  type --> 
  rule --> matches & no_match
  matches --> a(Does a duplicate bug exist?) --> d(No) & c(Yes)
  no_match --> b(Does a duplicate bug exist?) --> f(No) & c
  d --> file_bug
  f --> file_generic_bug
  c  --> comment
```

### How To Add Jira Reporting to a Scenario

**If you currently use the ipi-aws workflow:**

1. Ask your PQE contact which Jira project they would like test failures to be reported to
2. Modify the scenario to use the `firewatch-ipi-aws` workflow instead of the `ipi-aws` workflow
3. Add the required environment variables:
   - `FIREWATCH_DEFAULT_JIRA_PROJECT`: This is the Jira project you'd like tickets to be filed to if the failure found does not match any rules. For Interop QE, this will probably be set to `LPINTEROP`
   - `FIREWATCH_CONFIG`: Where we define the rules for which tickets get filed where. Please see the [How to define the configuration](https://github.com/CSPI-QE/firewatch/blob/main/docs/cli_usage_guide.md#defining-the-configuration) section of the Firewatch documentation for help defining this variable.
   - `FIREWATCH_JIRA_SERVER`: `https://issues.redhat.com`
     - This value always defaults to the stage server to avoid unwanted bugs.
   - `FIREWATCH_DEFAULT_JIRA_ADDITIONAL_LABELS` : Adding the following 3 labels to every firewatch config step: `["<ocp-version>-lp","self-managed-lp","<scenario-short-name-lp>"]`

**If you currently use a custom workflow:**

1. Add the `firewatch-report-issues` ref to the end of the post steps in your workflow
2. Ask your PQE contact which Jira project they would like test failures to be reported to
3. Add the required environment variables:
   - `FIREWATCH_DEFAULT_JIRA_PROJECT`: This is the Jira project you'd like tickets to be filed to if the failure found does not match any rules. For Interop QE, this will probably be set to `LPINTEROP`
   - `FIREWATCH_CONFIG`: Where we define the rules for which tickets get filed where. Please see the [How to define the configuration](https://github.com/CSPI-QE/firewatch/blob/main/docs/cli_usage_guide.md#defining-the-configuration) section of the Firewatch documentation for help defining this variable.
   - `FIREWATCH_JIRA_SERVER`: `https://issues.redhat.com`
     - This value always defaults to the stage server to avoid unwanted bugs.
   - `FIREWATCH_DEFAULT_JIRA_ADDITIONAL_LABELS` : Adding the following 3 labels to every firewatch config step: `["<ocp-version>-lp","<platform-name>-lp","<scenario-short-name-lp>"]`

Please see [this PR](https://github.com/openshift/release/pull/39700/files) as an example of how to add these values to your scenario.

> **IMPORTANT**
>
> When defining the `FIREWATCH_CONFIG` variable, please try to cover every step that is executed during your scenario, you can view the steps that are run by going to a recent run of your scenario and viewing the artifacts. Each step should have a folder for it's artifacts and logs that you can use to build your config. If you happen to miss one of the steps and a failure occurs in that step, it will cause the failure to not match any of the rules in the config. In that case, a generic bug for the failure will be filed in the `FIREWATCH_DEFAULT_JIRA_PROJECT` project.

## Component Readiness

This section explains how layered-product results appear in **[Component Readiness](https://sippy.dptools.openshift.org/sippy-ng/component_readiness/main)** — the Sippy UI where LP interop health is tracked by OpenShift release and product.

For an **end-to-end, multi-repository workflow** (sanitized clones, branch discipline, maintainer hand-offs), see the [**LP Interop CR agent playbook**](LP_Interop_CR_Agent_Playbook.md).

### General information

- **Interop Component Readiness view:** The layered-product interop view is named `<OCPRelease>-LP-Interop`, where `<OCPRelease>` is the OpenShift minor version label Sippy expects for that stream (for example `4.22` → `4.22-LP-Interop`).
  - Open [Component Readiness](https://sippy.dptools.openshift.org/sippy-ng/component_readiness/main) with `?view=<OCPRelease>-LP-Interop`, or follow a bookmark such as [4.22-LP-Interop](https://sippy.dptools.openshift.org/sippy-ng/component_readiness/main?view=4.22-LP-Interop) for OCP 4.22.
  - **Where views are defined:** Supported releases and their view IDs are listed in Sippy’s [config/views.yaml](https://github.com/openshift/sippy/blob/main/config/views.yaml). Search for `component_readiness` entries whose names end in `-LP-Interop`; that file is the source of truth when choosing a `view=` query for a given OCP release.
  - **Release rotation:** When a new OpenShift minor ships, SHIP/TRT add the matching `<release>-LP-Interop` entry to `views.yaml` and Component Readiness moves its default spotlight forward. Layered-product teams do **not** need to request a brand-new Component Readiness view for every minor release.
- **Maintainers:** SHIP and TRT own this UI; contact `#forum-ocp-release-oversight` on Slack.

For scenario configuration that satisfies CR at the job level (cron, workflows, environment variables, JUnit suite mapping), follow [Make a Job CR-Compliant](../Scenario_Development/Scenario_Development_Guide.md#make-a-job-cr-compliant) in the Scenario Development Guide.

### Sippy

This document is a **checklist for coding agents** (and humans) adding support in [Sippy](https://github.com/openshift/sippy) for a new **MY-CMP** product that publishes CI under the layered-product / lp-interop pattern (mapped JUnit suite like `lp-ocp-compat--<lpProductName>`, Prow jobs under `…-lp-interop-…`).

---

#### Prerequisites: gather required information

Within the `openshift/release` repository, under CI Configuration files (`ci-operator/config/**/*.yaml`), confirm:

1. **Mapped JUnit suite string** as CI emits it in imported JUnit from Prow. See [Map the JUnit tests output](../Scenario_Development/Scenario_Development_Guide.md#map-the-junit-tests-output) in the Scenario Development Guide for how CI produces that mapping.

  - Often `lp-ocp-compat--<lpProductName>`, e.g. `lp-ocp-compat--OpenshiftPipelines`. It must match CI **exactly** (case-sensitive). See [Allow importing tests](#1-allow-importing-tests-pkgdbsuitesgo) below for how Sippy accepts suites via **`testSuitePatterns`**.

2. **Stable substring of periodic name**, e.g. `-lp-interop-cr- `. The variant registry matches **literal substrings** on the lowercased job name (first match wins).

  - If **multiple** patterns are required (e.g., `-lp-interop-cr-acs` and `-lp-interop-cr-acs-latest`), add **separate** rows, ensuring the **more specific patterns precede the more general ones**.

#### Note for AI / automation assistants (Sippy)

Do **not** run any `make` commands (or substitute commands) in this repository on behalf of the requester. A maintainer must run them locally when noted below.

---

#### 1. Allow importing tests: `pkg/db/suites.go`

**File:** [`pkg/db/suites.go`](https://github.com/openshift/sippy/blob/main/pkg/db/suites.go)

**Slices:** **`testSuitePatterns`** (`[]*regexp.Regexp`) and **`testSuites`** (`[]string`)

For **standard LP interop onboarding**, **do not** add product-specific suite strings to **`testSuites`**. Instead, rely on **`testSuitePatterns`**: ensure every suite-name prefix CI emits is covered by a regex. Today upstream includes LP-oriented patterns such as `^lp-chaos--`, `^lp-interop--`, and `^lp-ocp-compat--` (see [`testSuitePatterns` in `suites.go`](https://github.com/openshift/sippy/blob/main/pkg/db/suites.go#L80)). Therefore **match the existing `^lp-ocp-compat--` pattern** and require **no new row** in **`testSuites`**.

- **New prefix family:** If CI introduces suite names that **do not** match any existing pattern, add **`regexp.MustCompile(...)`** to **`testSuitePatterns`** rather than enumerating literals.
- **`testSuites` literals:** Still used for selected **legacy or non-regex** suite names (upstream examples include component-style entries such as `CNV-lp-interop`). Current onboarding routine does **not** touch this list.
- **ci-test-mapping:** Keep **`Matchers`** (**`Suite`** / **`SuiteRegEx`**) aligned with the suite strings CI actually emits; Sippy import coverage is via **`testSuitePatterns`**, not by duplicating every suite string in **`testSuites`**.

Suites that match **neither** patterns **nor** the explicit list are **not** imported into Sippy’s DB.

---

#### 2. Map Prow job names → `LayeredProduct`: `pkg/variantregistry/ocp.go`

**File:** `pkg/variantregistry/ocp.go`
**Function:** `setLayeredProduct`

In `setLayeredProduct`, append a row to the job-name substring → `LayeredProduct` mapping table:

```go
{"-lp-interop-cr-my-cmp", "lp-interop-my-cmp"},
```

**Rules:**

- **`product` value:** always use the **`lp-interop-…`** form (lowercase, hyphenated), e.g. `lp-interop-my-cmp`. This is what Component Readiness views filter on.
- **`substring`:** must appear in real periodic job names after lowercasing. Align with CI naming (often `-lp-interop-cr- `).
- **Order matters:** the slice is scanned **top to bottom**; the **first** match wins. Place **narrow** patterns (e.g. product-specific) **above** broad patterns so lp-interop jobs are not misclassified.

> **WARNING** (`pkg/variantregistry/ocp.go`, `setLayeredProduct`)
>
> That mapping table is evaluated in **slice order**: the **first** substring match wins, and later rows are ignored for that job. Do **not** append an LP-specific row **below** a broader row that can still match the same periodic name, for example `{"-virt", "virt"}` vs. `{"-lp-interop-cnv", "virt"}` and similar catch-alls. Misordering silently misclassifies jobs in Component Readiness. Keep narrow lp-interop rows **above** generic mappings.

**Optional (IBM / on-prem style job names):** When jobs include `-ibm` / `-ibmcloud` and those jobs should appear alongside bare metal in platform filtering, confirm `setPlatform` includes the `{"-ibm", "metal"}` mapping (or add it if the branch lacks it). That step is **independent** of lp-interop onboarding but determines whether the **Platform** filter lists those jobs.

---

#### 3. Include the product in LP-Interop views: `config/views.yaml`

**File:** `config/views.yaml`

For each Component Readiness view named like **`*-LP-Interop`** (e.g. `4.22-LP-Interop`, `4.21-LP-Interop`) that lists layered products under:

```yaml
variant_options:
  include_variants:
    LayeredProduct:
      - lp-interop-...
```

add:

```yaml
      - lp-interop-my-cmp
```

Use the **same string** as in `setLayeredProduct`’s `product` field. Keep the list **alphabetically sorted** unless the file already uses a different convention for that block.

**Note:** Some older views (for example certain `4.20-*` LP views) list only a subset of products. Add the product entry only alongside blocks where other `lp-interop-*` products already appear.

---

#### 4. Tests and variant snapshot (do not run `make` here)

##### 4a. Unit test (recommended)

**File:** `pkg/variantregistry/ocp_test.go`
**Test:** `TestVariantSyncer`

Add a case with a **realistic** periodic job name for MY-CMP (including release and network tokens if needed) and assert `VariantPlatform`, `VariantLayeredProduct`, etc., match what `IdentifyVariants` returns.

##### 4b. Variant snapshot

**Test:** `TestVariantsSnapshot` in `pkg/variantregistry/ocp_test.go` compares live variants for all jobs in `config/openshift.yaml` against **`pkg/variantregistry/snapshot.yaml`**.

After **any** change to variant logic in `pkg/variantregistry/ocp.go` (including `setLayeredProduct` / `setPlatform`), that snapshot **must** be regenerated or the test will fail.

**Agents must not run `make`.** Have a maintainer run the command below **after** the Go changes are merged or applied locally:

```bash
make update-variants
```

That target builds `./sippy` and runs:

```bash
./sippy variants snapshot --config ./config/openshift.yaml
```

which rewrites `pkg/variantregistry/snapshot.yaml`.

> Expect snapshot tests to fail until `make update-variants` has been run by a maintainer.

---

#### 6. What **not** to do

- Do **not** run **`make`** (including `make update-variants`, `make`, `make test`, and so on) from automation; document the need for **`make update-variants`** when variant code changes.
- Do **not** hand-edit **`snapshot.yaml`** without a documented, repo-approved process; prefer **`make update-variants`**.
- Do **not** change unrelated views, suites, or variant patterns.
- After frontend changes under `sippy-ng`, this onboarding path does not require npm; when JavaScript is modified, follow `AGENTS.md` (eslint/prettier) separately.

---

#### Summary checklist

1. **`pkg/db/suites.go`:** Confirm **`testSuitePatterns`** covers your JUnit suite prefixes (upstream LP defaults include `^lp-ocp-compat--`, `^lp-interop--`, `^lp-chaos--`); add **`regexp.MustCompile`** only if CI uses a **new** prefix. Do **not** add per-product suite literals to **`testSuites`** for standard mapped names.
2. **`pkg/variantregistry/ocp.go`:** Add `setLayeredProduct` periodic-name substring → **`LayeredProduct`** (example **`lp-interop-my-cmp`**).
3. **`config/views.yaml`:** Add `lp-interop-my-cmp` to `*-LP-Interop` views’ `LayeredProduct`.
4. **`pkg/variantregistry/ocp_test.go`:** Add `TestVariantSyncer` case (recommended).
5. **Maintainer:** Run **`make update-variants`** after variant changes.

Replace `my-cmp`, `MyProduct`, and `lp-ocp-compat--MyProduct` with the actual layered-product variant slug and mapped suite string everywhere below (see prerequisites for suite generation in CI).

### CI Test Mapping

This checklist is written for **coding agents** (AI and automation assistants) and humans who implement onboarding in the [openshift-eng/ci-test-mapping](https://github.com/openshift-eng/ci-test-mapping) repository.

The steps below add a new **layered product interop** component to that repository. Component Readiness maps each test to one **component** and optional **capabilities**. LP interop jobs publish JUnit with a dedicated mapped **test suite** name produced from `DR__RP__CR_COMP_NAME`: pattern **`lp-ocp-compat--<lpProductName>`**.

Replace placeholders below:

- **`lp-ocp-compat--MyProduct`:** exact mapped JUnit **suite** string from CI (must match `DR__RP__CR_COMP_NAME` / `includeSuitePatterns` / `Matchers`; see [Map the JUnit tests output](../Scenario_Development/Scenario_Development_Guide.md#map-the-junit-tests-output)).
- **`myproductlpinterop`:** Go **package** / directory name: lower case, no hyphens (typical 
pattern: strip `-lp-interop` and join words).
- **`MyProductLpInteropComponent`:** exported Go **variable** for the component singleton (used with `r.Register`).

---

#### Prerequisites

1. **Mapped suite string is stable** and appears on every relevant JUnit result as the suite attribute (same value supplied by `DR__RP__CR_COMP_NAME` in CI; pattern **`lp-ocp-compat--<lpProductName>`**, e.g. **`lp-ocp-compat--MyProduct`**).
2. **Registered `OCPBUGS` component name**: `DefaultJiraComponent` must correspond to a real Jira component the team owns.

   - Verify components with `./ci-test-mapping jira-verify` as described in the root [README.md](../../README.md#updating-jira-components).
   - Registered components can be found at [OCPBUGS components](https://redhat.atlassian.net/jira/software/c/projects/OCPBUGS/components).

---

#### Note for AI / automation assistants (CI Test Mapping)

Do **not** run any `make` targets (or substitute commands) in this repository on behalf of the requester. After editing `config/openshift-eng.yaml` or component code, **mapping regeneration is required** before merge: maintainers must run **`make mapping`** (see **Updating Mappings** in the root [README.md](../../README.md#updating-mappings)). Record that requirement explicitly in automation output; do not execute those targets from automation. Reviewers should inspect the resulting `data/` diff before merging.

---

#### 1. Include the suite in the OpenShift mapping config

Edit [config/openshift-eng.yaml](../../config/openshift-eng.yaml) and add a pattern matching the mapped suite to `includeSuitePatterns`, in alphabetical order with the other `*-lp-interop` entries:

```yaml
includeSuitePatterns:
  - `^my-prefix-pattern--`
```

Without this, tests from that suite may not appear in the mapping inputs at all.

---

#### 2. Add a component package

Create a new directory:

`pkg/components/myproductlpinterop/`

##### `component.go`

Model it on [pkg/components/myproductlpinterop/component.go](../../pkg/components/myproductlpinterop/component.go):

- Set `Name` to the same string as the mapped JUnit suite (e.g. `lp-ocp-compat--MyProduct`) so it matches `Register` and `Suite` matchers. Set `DefaultJiraComponent` to the **OCPBUGS** Jira component name owned by the team. Older naming styles (e.g. `MyProduct-lp-interop`) remain acceptable and do **not** need to match the suite string.
- Use **`Matchers`** so this component owns the right tests:
  - **`Suite`** — Use for an **exact** JUnit suite string. Many components still carry a **legacy** row such as `{Suite: "MyProduct-lp-interop"}` (component-style name); keep it when tests still report that suite—**do not drop it** when adding **`SuiteRegEx`**.
  - **`SuiteRegEx`** — Use `regexp.MustCompile(...)` for **additional** suite prefixes or patterns (for example `^lp-ocp-compat--MyProduct--`, `^lp-interop--MyProduct--`, `^lp-chaos--MyProduct--`). Add `"regexp"` to the imports in `component.go`. Regex suite matching in **`ComponentMatcher`** is **newer** than plain **`Suite`**; it **extends** legacy **`Suite`** rows rather than replacing them.

  ```go
  // Example only — replace MyProduct / myproductlpinterop with your product identifiers.

  import (
      "regexp"

      "github.com/openshift-eng/ci-test-mapping/pkg/config"
  )

  var MyProductLpInteropComponent = Component{
      Component: &config.Component{
          Name:                 "MyProduct-lp-interop",
          Operators:            []string{},
          DefaultJiraComponent: "MyProduct",
          Matchers: []config.ComponentMatcher{
              {Suite: "MyProduct-lp-interop"}, // legacy exact suite (keep when present)
              {SuiteRegEx: regexp.MustCompile(`^lp-ocp-compat--MyProduct--`)},
              {SuiteRegEx: regexp.MustCompile(`^lp-interop--MyProduct--`)},
              {SuiteRegEx: regexp.MustCompile(`^lp-chaos--MyProduct--`)},
          },
      },
  }
  ```

For finer-grained ownership later, add more `ComponentMatcher` entries (substrings, priorities, per-matcher Jira components) using patterns from [pkg/components/example](../../pkg/components/example).

##### `capabilities.go`

Add a `capabilities.go` next to `component.go`, modeled on [pkg/components/myproductlpinterop/capabilities.go](../../pkg/components/myproductlpinterop/capabilities.go). Define `identifyCapabilities` starting from `util.DefaultCapabilities(test)`; extend the returned slice only when capabilities beyond the defaults are required.

```go
package myproductlpinterop

import (
	v1 "github.com/openshift-eng/ci-test-mapping/pkg/api/types/v1"
	"github.com/openshift-eng/ci-test-mapping/pkg/util"
)

func identifyCapabilities(test *v1.TestInfo) []string {
	capabilities := util.DefaultCapabilities(test)
	return capabilities
}
```

---

#### 3. Register the component

Edit [pkg/registry/registry.go](../../pkg/registry/registry.go):

1. Add the import:

   ```go
   "github.com/openshift-eng/ci-test-mapping/pkg/components/myproductlpinterop"
   ```

2. Register next to the other LP interop component entries (keep ordering consistent with nearby registrations):

   ```go
   r.Register("lp-ocp-compat--MyProduct", &myproductlpinterop.MyProductLpInteropComponent)
   ```

The string passed to `Register` is the **component name** used in mappings; it must match `Name` in the `config.Component` block and the mapped JUnit **suite** string (the example above uses **`lp-ocp-compat--MyProduct`**).

---

#### 4. Validate and ship

1. Regenerate committed mapping data: after changing config or components, maintainers must run `make mapping` (see **Updating Mappings** in the root [README.md](../../README.md#updating-mappings)).
2. AI and automation assistants must not execute `make`; flag that this step is mandatory before merge.

---

#### Quick checklist

1. **`config/openshift-eng.yaml`:** Add a pattern for the mapped suite to `includeSuitePatterns` (alphabetically with other `lp-ocp-compat--…` entries).
2. **`pkg/components/myproductlpinterop/component.go`:** **`Matchers`** — keep any **legacy** **`Suite`** using pattern **`<ProductName>-lp-interop`**; add **`SuiteRegEx`** (+ `"regexp"`) for prefix **patterns** (`^lp-ocp-compat--<ProductName>--`, …); list **`Suite`** before **`SuiteRegEx`**.
3. **`pkg/components/myproductlpinterop/capabilities.go`:** `identifyCapabilities` + `util.DefaultCapabilities` (see [myproductlpinterop/capabilities.go](../../pkg/components/myproductlpinterop/capabilities.go)).
4. **`pkg/registry/registry.go`:** Import package + `r.Register(...)`.
5. **Jira / verification:** `DefaultJiraComponent` exists; `./ci-test-mapping jira-verify` clean.
6. **Maintainer:** Run **`make mapping`** (required before merge).

