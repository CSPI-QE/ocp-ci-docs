# Component Readiness — agent playbook

This playbook is for **coding agents and automation** that drive **layered-product LP Interop** work through **Component Readiness (CR)**. It coordinates **openshift/release**, **openshift/sippy**, and **openshift-eng/ci-test-mapping** (optional **ocp-ci-docs**). **User/requester** steps (for example `make`) stay in [Reporting Guide](Reporting_Guide.md) and [Scenario Development Guide](../Scenario_Development/Scenario_Development_Guide.md).

## At a glance (agents)

1. **Phase 0 first** — `WORKDIR`, clones (skip if dirs exist), **`upstream`** / **`origin`** remotes, sync **`main`**, feature branches, then **derive** identifiers from **`openshift/release`** YAML before editing other repos. If Git operations in Phase 0 cannot run in this environment, **stop** until the requester confirms how to proceed (for example by preparing the workspace locally).
2. **Implement in dependency order** — **release** → **sippy** → **ci-test-mapping**; prefer **release** merging first.
3. **Do not run** `make` in **sippy** / **ci-test-mapping** unless `[MAKE_MAINTAINER]` is **agent** and policy allows; otherwise **record** maintainer commands in PR bodies.
4. **Every commit message** must cite `**[ocp-ci-docs] LP_Interop_CR_Agent_Playbook**` **as the source** of the procedure used.
5. **Deliverables** — Paste-ready PR descriptions per touched repo, PR URL list, remaining `make` steps, blockers; **[per-run log file and shell trace](#run-logging-agents)** (start/end times, correlated names; **never `git push` those artifacts to `origin`** unless the requester explicitly overrides); then **Cleanup** `WORKDIR`.

---

## Before you start: prerequisites (user/requester fills in)

Provide **once**; paste into the [prompt template](#prompt-template-for-agent-execution).

1. **Product display name** — CR name; drives **`<lpProductName>`** in **`lp-ocp-compat--<lpProductName>`**. Example: `MyProduct`.
2. **Product slug** — Lowercase, hyphenated; **`lp-interop-<slug>`** and branches. Example: `myproduct`.
3. **Target OCP minor release(s)** — e.g. `4.22`. Agent derives Sippy **`view=`** from **`openshift/sippy`** `config/views.yaml` (**`<minor>-LP-Interop`**); no separate view string from the requester.
4. **`openshift/release` config path(s)** — e.g. `ci-operator/config/Org/repo/Org-repo-branch__variant.yaml`.
5. **Periodic job name fragment** — For **`setLayeredProduct`** (often **`-lp-interop-cr-…`**). Example: `-lp-interop-cr-myproduct`.
6. **OCPBUGS / `DefaultJiraComponent`** — If known. Example: `MyProduct`.
7. **Remote / fork policy** — **Default:** in **`release`**, **`sippy`**, and **`ci-test-mapping`**, use **`upstream`** for the official GitHub repo (**`openshift/release`**, **`openshift/sippy`**, **`openshift-eng/ci-test-mapping`**) and **`origin`** for the contributor fork (see [Phase 0 → Remotes](#phase-0--sanitized-workspace-mandatory-before-edits)). Example **`origin`**: `git@github.com:<GH_USER>/release.git`. Only override this layout if your team documents a different convention (single remote, other names, or HTTPS instead of SSH).
8. **GitHub username (fork owner)** — The GitHub user or org that hosts forks of **`release`**, **`sippy`**, and **`ci-test-mapping`** (same `<GH_USER>` in all three **`origin`** URLs unless you say otherwise). Example: `janedoe`.
9. **Maintainer for `make`** — **requester** or **agent**, per team policy.

Optional: links to existing **PRs**, **Prow** jobs, or **GCS** artifacts.

---

## Prompt template for agent execution

Copy the list below into the agent chat. Replace **`[PLACEHOLDERS]`** with prerequisites.

```text
You are executing LP Interop Component Readiness onboarding.

Inputs (from the user/requester — use exactly; do not invent):
- Product display name: [PRODUCT_DISPLAY_NAME]
- Product slug: [PRODUCT_SLUG]
- OCP release(s): [OCP_MINOR_LIST]
- Periodic job name substring: [JOB_SUBSTRING]
- openshift/release config file(s): [RELEASE_CONFIG_PATHS]
- OCPBUGS / DefaultJiraComponent (if known): [JIRA_COMPONENT]
- GitHub username (fork owner, for origin remotes): [GH_USER]
- Maintainer for make (who runs `make update`, `make update-variants`, `make mapping`): [MAKE_MAINTAINER]

Instructions (execute in this order):
1. Follow Phase 0 in docs/OCP_CI_Tutorials/Reporting/LP_Interop_CR_Agent_Playbook.md (WORKDIR, clone-if-missing, default **`upstream`** / **`origin`** remotes using **`[GH_USER]`** unless **`[REMOTE_POLICY]`** overrides, reset **`main`** from **`upstream`**, feature branches; identifier capture from **release** YAML when applicable, no forbidden `make`, commit-message rule). If Phase 0 Git steps cannot run here, **stop** until the requester confirms how to proceed. Run **Cleanup** only after branches are pushed and local clones are no longer needed.
2. With `[RELEASE_CONFIG_PATHS]` open: from **tests** (jobs, steps, workflow, commands, chain/ref, env), list **step-registry** paths to touch and confirm **`LayeredProduct`** implied for Sippy. Only ask the user/requester if YAML is incomplete.
3. From **OCP release(s)** + `openshift/sippy/config/views.yaml`, derive **`view=`** (**`<minor>-LP-Interop`**). Do not invent a view outside that file.
4. From **Product display name**, set **`DR__RP__CR_COMP_NAME`** / mapped suite to **`lp-ocp-compat--<lpProductName>`** (Scenario Development Guide for normalization). Same string in **release**, **sippy**, **ci-test-mapping**.
5. Honor `[MAKE_MAINTAINER]` for `make update`, `make update-variants`, `make mapping`: **requester** → document in PRs, do not run yourself unless policy says **agent** may. Default: no `make` in **sippy** / **ci-test-mapping** for agents (Reporting Guide).
6. Implement: **openshift/release** → **openshift/sippy** → **openshift-eng/ci-test-mapping**; Cross-link PRs. Prefer **release** merging first.
7. Every commit message: cite **[ocp-ci-docs] LP_Interop_CR_Agent_Playbook** as the **source** (short trailer or footer line is 
enough).
8. End state: branches pushed, checklist satisfied, **Cleanup** removes `$WORKDIR`.
9. Run logging: write the per-run Markdown log and correlated executable shell trace under `docs/OCP_CI_Tutorials/Reporting/` (see **Run logging** in the playbook): same `<slug>_<RUN_ID>` basename, log includes **start** and **end** times; **do not** commit or **`git push … origin`** those artifacts unless the requester explicitly says otherwise; if you cannot write to that repo, paste both artifacts in full for the requester.

Deliverables: paste-ready PR description per modified repo; list of PR URLs; maintainer-only commands left; blockers; per-run log file and correlated shell trace (see Run logging; **do not `git push` those artifacts to `origin`** unless the requester explicitly overrides).
```

---

## Repositories in scope

1. **[openshift/release](https://github.com/openshift/release)**:
   **a.** **CI configuration** — `ci-operator/config/**`, periodic job names, workflows (`Firewatch-ipi-aws-cr`), env vars (`DR__RP__CR_COMP_NAME`, `MAP_TESTS`, …).
   **b.** **Step-registry changes** — apply test mapping in test scripts (commands/refs under `ci-operator/step-registry`).
2. **[openshift/sippy](https://github.com/openshift/sippy)**: **`testSuitePatterns`** in `pkg/db/suites.go` (add regex only for **new** suite prefixes; see Reporting Guide). `setLayeredProduct`, `config/views.yaml`, tests.
3. **[openshift-eng/ci-test-mapping](https://github.com/openshift-eng/ci-test-mapping)**: `includeSuitePatterns`, **`Matchers`** (**`Suite`** / **`SuiteRegEx`**), component package, `pkg/registry/registry.go`, capabilities.

Supporting libraries (for example [RedHatQE/OpenShift-LP-QE--Tools](https://github.com/RedHatQE/OpenShift-LP-QE--Tools) for `ExitTrap--PostProcessPrep`) are consumed from CI scripts; they are **not** typically forked as part of this onboarding unless the scenario requires upstream changes there.

## Phase 0 — Sanitized workspace (mandatory before edits)

The agent needs a working **Git** client to clone, set remotes, sync **`main`** from **`upstream`**, and create feature branches. If the environment **cannot** run those operations (no Git, blocked network, read-only sandboxes, and so on), **do not continue** with edits in **release** / **sippy** / **ci-test-mapping** until the **user/requester** confirms how to proceed (for example by running Phase 0 locally and supplying patches or branches). Output remediation (**what failed**, **what is needed**, and **stale artifacts left behind**—for example a partial **`$WORKDIR`** or clone dirs that were **not** removed because [Cleanup](#cleanup) did not run) instead of unverified code changes.

Use one **`$WORKDIR`** for all git work; destroy it in [Cleanup](#cleanup).

1. **Working root**

   ```bash
   WORKDIR=$(mktemp -d -t cr-onboarding-XXXXXX)
   cd "$WORKDIR"
   ```
   
2. **Repos** — Clone only missing dirs under **`$WORKDIR`** (reuse if continuing):

   ```bash
   [ ! -d release ] && git clone https://github.com/openshift/release.git
   [ ! -d sippy ] && git clone https://github.com/openshift/sippy.git
   [ ! -d ci-test-mapping ] && git clone https://github.com/openshift-eng/ci-test-mapping.git
   ```

   For a **full** re-clone, remove those dirs or pick a new **`$WORKDIR`**.

   **Remotes (default)** — After each repo’s first clone under **`$WORKDIR`**, set **`upstream`** to the official remote and **`origin`** to the requester’s fork. Substitute **`<GH_USER>`** with prerequisite **GitHub username (fork owner)** (HTTPS examples; use `git@github.com:<GH_USER>/…` if that is your policy):

   ```bash
   cd release
   git remote rename origin upstream
   git remote add origin https://github.com/<GH_USER>/release.git

   cd ../sippy
   git remote rename origin upstream
   git remote add origin https://github.com/<GH_USER>/sippy.git

   cd ../ci-test-mapping
   git remote rename origin upstream
   git remote add origin https://github.com/<GH_USER>/ci-test-mapping.git
   ```

   If a repo was already configured in an existing **`$WORKDIR`**, skip duplicate **`git remote add`**. If **`[REMOTE_POLICY]`** overrides the default layout, follow that policy instead.

3. **Sync `main`** (or default branch) from **`upstream`** when using the default remotes:

   ```bash
   cd release && git fetch upstream && git checkout main && git pull --ff-only upstream main
   cd ../sippy && git fetch upstream && git checkout main && git pull --ff-only upstream main
   cd ../ci-test-mapping && git fetch upstream && git checkout main && git pull --ff-only upstream main
   ```

4. **Feature branches** — One slug across repos (examples **`onboarding-myproduct`** / **`myproduct`**):

   ```bash
   cd release && git checkout -b onboarding-myproduct
   cd ../sippy && git checkout -b myproduct
   cd ../ci-test-mapping && git checkout -b myproduct
   ```

5. **Identifiers** — After reading **`openshift/release`** configs from prerequisites, fix once: **`lp-ocp-compat--<lpProductName>`** / **`DR__RP__CR_COMP_NAME`**, **`LayeredProduct`**, periodic substring (**`-lp-interop-cr-<slug>`**). **Case-consistent** everywhere. Go package directory: **`<slug>`** with hyphens removed, plus **`lpinterop`** (example slug **`mpexoperator`** → **`mpexoperatorlpinterop`**).

6. **No forbidden `make`** in **sippy** / **ci-test-mapping** for agents unless policy overrides — document **`make update-variants`** / **`make mapping`** in PRs ([Reporting Guide](Reporting_Guide.md)).

7. **Commit messages:** Every commit message must note that the work was generated using this playbook — reference **`[ocp-ci-docs] LP_Interop_CR_Agent_Playbook`** so reviewers can trace the workflow (one line in the body or as a trailer is sufficient).

### Cleanup

```bash
cd /
rm -rf "$WORKDIR"
```

Leave **`$WORKDIR`** first; copy anything to keep **before** `rm -rf`.

---

## Recommended order of work

```mermaid
flowchart LR
  release[openshift/release]
  sippy[openshift/sippy]
  ctm[ci-test-mapping]
  release --> sippy
  release --> ctm
```

1. **`openshift/release`** — [Make a Job CR-Compliant](../Scenario_Development/Scenario_Development_Guide.md#make-a-job-cr-compliant); [Map the JUnit tests output](../Scenario_Development/Scenario_Development_Guide.md#map-the-junit-tests-output). **`lp-ocp-compat--<lpProductName>`** and periodic fragments match what **sippy** will ingest.
2. **`openshift/sippy`** — [Reporting Guide → Sippy](Reporting_Guide.md#sippy): **`testSuitePatterns`**, `setLayeredProduct`, `config/views.yaml`, tests. Substring **order** matters.
3. **`openshift-eng/ci-test-mapping`** — [Reporting Guide → CI Test Mapping](Reporting_Guide.md#ci-test-mapping): `includeSuitePatterns`, **`Matchers`**, registry, capabilities.
4. **`ocp-ci-docs`** (optional) — Doc alignment.

PRs may open in parallel once identifiers are fixed; **release** should merge **first** when possible.

---

## Run logging (agents)

Each playbook execution must leave **two correlated artifacts** in **`ocp-ci-docs`** (default directory: this folder, `docs/OCP_CI_Tutorials/Reporting/`), unless the **user/requester** names a different path in the prompt. Use the **same basename** for both files.

**Never push these artifacts to `origin`.** Do not `git add`, commit, or **`git push … origin`** the run log (`*_log.md`) or shell trace (`*_commands.sh`) from an automated playbook run—on **openshift/release**, **openshift/sippy**, **openshift-eng/ci-test-mapping**, or **ocp-ci-docs**—unless the **user/requester** explicitly instructs otherwise. Default: keep them **local** or deliver them **in chat / attachments** only so fork branches and product PRs stay free of trace noise and accidental paths or tokens.

### 1. Run log file (Markdown)

Create **one log file per run** that records everything the agent produced for that execution (or a faithful summary if the chat tool cannot write the full transcript). The log **must** include:

- **Start time** and **end time** — capture at the beginning of work (before or at Phase 0) and when deliverables are complete (before **Cleanup**). Use ISO 8601 (recommend **one line in local offset** and **one line in UTC**, e.g. `date -Iseconds` and `date -u +%Y-%m-%dT%H:%M:%SZ`).
- **Prerequisites** as received (product name, slug, OCP minors, job substring, Jira component, fork owner, `make` policy, release paths if any).
- **Phase 0** outcome (clones, remotes, **`main`** sync, feature branches) and any deviation from the default remote layout.
- **What changed** repo by repo (files, identifiers, tests run).
- **Deliverables** — PR blurbs, PR URLs when known, maintainer **`make`** lines, **blockers** (auth, missing forks, policy).
- A **Related** line linking to the correlated shell trace file (see below).

**Suggested filename** (replace `<slug>` with the product slug; `<RUN_ID>` with a UTC timestamp taken at **start** so both artifacts share it):

`LP_Interop_CR_Agent_Run_<slug>_<RUN_ID>_log.md`  
Example RUN_ID: `20260513T215811Z` from `date -u +%Y%m%dT%H%M%SZ`.

**Example (PoC):** [LP_Interop_CR_Agent_Playbook_PoC_agent_log.md](LP_Interop_CR_Agent_Playbook_PoC_agent_log.md). #TODO: add poc

### 2. Correlated shell trace (executable)

Create **one Bash script per run** that lists **all terminal commands** executed in order (clone, remotes, fetch, branch, verification commands such as `go test` / `gofmt` when used, and so on). Rules:

- **`#!/usr/bin/env bash`** at the top; document **`GH_USER`**, **`FEATURE_BRANCH`**, and optional **`WORKDIR`** / **`RUN_VERIFY`** if the script supports replay.
- Use **`#` comments** for steps that were **not** shell-driven (for example hand-edited files, or **`make`** left to the maintainer).
- Prefer **`|| true`** only where the historical run **expected** a non-zero exit (for example a probe command used only to record environment limits); otherwise keep failures visible for replay debugging.
- **`chmod +x`** on the script when the filesystem allows it.
- The script’s basename must match the log file’s basename except for the suffix: **`_log.md`** vs **`_commands.sh`**.

**Suggested filename:**

`LP_Interop_CR_Agent_Run_<slug>_<RUN_ID>_commands.sh`  
(same `<slug>` and `<RUN_ID>` as the log file for that run.)

**Example (PoC):** [LP_Interop_CR_Agent_Playbook_PoC_commands.sh](LP_Interop_CR_Agent_Playbook_PoC_commands.sh). # TODO: add poc

### 3. Checklist tie-in

Treat the log and shell trace as part of **deliverables** alongside PR text. If the agent cannot write to **`ocp-ci-docs`**, state that in the log and give the **full pasted content** of both artifacts in the chat so the requester can save them locally with the correct names. In all cases, follow **[Never push these artifacts to `origin`](#run-logging-agents)** above.

---

## Maintainer hand-offs (not automated)

Put in **each** PR for the **user/requester**:

1. **sippy** — `make update-variants` after variant edits (agents do **not** run by default).
2. **ci-test-mapping** — `make mapping` + review `data/` (agents do **not** run by default).
3. **release** — `make update` / `make jobs` per policy when config or step-registry changes.

---

## End-to-end checklist

- [ ] **Phase 0** — `WORKDIR`; repos cloned or reused; **`upstream`** / **`origin`** configured; **`main`** synced from **`upstream`**; feature branches created; identifiers captured from **release** YAML.
- [ ] **Commits** — Every message cites the playbook **as the source** (**`[ocp-ci-docs]  LP_Interop_CR_Agent_Playbook`**).
- [ ] **release** — CR job; **`DR__RP__CR_COMP_NAME`**; ExitTrap / grace if mapping JUnit ([Scenario Development Guide](../Scenario_Development/Scenario_Development_Guide.md#make-a-job-cr-compliant)).
- [ ] **sippy** — **`testSuitePatterns`** for prefixes; **`setLayeredProduct`** order; **`LayeredProduct`** in `*-LP-Interop` views.
- [ ] **ci-test-mapping** — `includeSuitePatterns`; **`Matchers`**; **`DefaultJiraComponent`**; README / `jira-verify` as needed.
- [ ] **PRs** — Maintainer **`make`** lines + cross-repo links + **paste-ready** per-repo descriptions (what / why / identifiers / related PRs).
- [ ] **Run log** — Markdown file with **start** and **end** times, prerequisites, Phase 0 outcome, changes, deliverables, blockers; **Related** link to the shell trace; **not pushed to `origin`** unless the requester explicitly says otherwise.
- [ ] **Shell trace** — Executable **`_commands.sh`** with the same **`<slug>_<RUN_ID>`** basename as the log; terminal commands in order; comments for non-shell steps; **not pushed to `origin`** unless the requester explicitly says otherwise.

---

## Further reading

- [Scenario Development Guide — Make a Job CR-Compliant](../Scenario_Development/Scenario_Development_Guide.md#make-a-job-cr-compliant)
- [Reporting Guide — Component Readiness](Reporting_Guide.md#component-readiness)
