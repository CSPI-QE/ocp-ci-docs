# Worked Example: MPEXOperator (POC)

Documentation-only proof of concept. The MPEXOperator CI Operator Job Conf. in [`openshift/release`](https://github.com/openshift/release) may be fictional; [`openshift/sippy`](https://github.com/openshift/sippy) and [`openshift-eng/ci-test-mapping`](https://github.com/openshift-eng/ci-test-mapping) PR content below is still structurally valid for onboarding.

## Invocation

```text
/lp-ocp-compat-cr-onboarding lp-name=MPEXOperator lp-slug=mpexoperator lp-repo=redhatqe/mpexoperator lp-branch=main lp-ver=lpGA ocp-release=4.22 release-config=ci-operator/config/redhatqe/mpexoperator/redhatqe-mpexoperator-main__ocp-4.22-lpGA-lp-ocp-compat.yaml test-variant=aws cron="0 6,18 * * *" gh-user=GH_USERNAME make-maintainer=requester
```

## Identifier table

| Identifier                                   | Value                                                                                        |
|----------------------------------------------|----------------------------------------------------------------------------------------------|
| `DR__RP__CR_COMP_NAME` / TS prefix           | `lp-ocp-compat--MPEXOperator`                                                                |
| `.tests[].as`                                | `cr--mpexoperator--aws`                                                                      |
| Periodic CI Operator Job name                | `periodic-ci-redhatqe-mpexoperator-main-ocp-4.22-lpGA-lp-ocp-compat-cr--mpexoperator--aws`   |
| Sippy `layeredProductPatterns` sub-string    | `-lpga-lp-ocp-compat-cr--mpexoperator--`                                                     |
| Sippy CR Variant `LayeredProduct`            | `lp-ocp-compat--mpexoperator--lpGA`                                                          |
| CR View (`view=`)                            | `4.22-LP-OCP-Compat--lpGA`                                                                   |
| ci-test-mapping Go package                   | `lpmpexoperator`                                                                             |
| Jira / CR Component                          | `LP--MPEXOperator`                                                                           |
| `SuiteRegEx`                                 | `` `^lp-ocp-compat--MPEXOperator--` ``                                                       |
| Registry symbol                              | `LPmpexoperatorComponent`                                                                    |

## PR 1: openshift/release (mock)

**File:** `ci-operator/config/redhatqe/mpexoperator/redhatqe-mpexoperator-main__ocp-4.22-lpGA-lp-ocp-compat.yaml`

```yaml
tests:
  - as: cr--mpexoperator--aws
    cron: 0 6,18 * * *
    steps:
      env:
        MAP_TESTS: "true"
        DR__RP__CR_COMP_NAME: lp-ocp-compat--MPEXOperator
```

**Maintainer hand-off:** `make update` (not run in this POC; `make-maintainer=requester`).

**Verification:** After a CI Operator Job Run, JUnit XML under the CI Operator Test Step artifacts must show `<testsuite name="lp-ocp-compat--MPEXOperator--...">`.

## PR 2: openshift/sippy (mock)

**File:** `pkg/variantregistry/ocp.go` (inside `layeredProductPatterns` in `setLayeredProduct()`)

```go
{"-lpga-lp-ocp-compat-cr--mpexoperator--", "lp-ocp-compat--mpexoperator--lpGA"},
```

**File:** `config/views.yaml` (inside the `4.22-LP-OCP-Compat--lpGA` block, alphabetically sorted)

```yaml
    LayeredProduct:
      - lp-ocp-compat--mpexoperator--lpGA
```

**Files skipped (standard LP OCP Compat):** [Step 1 (`BigQuery pattern`)](../../../../docs/OCP_CI_Tutorials/Reporting/Reporting_Guide.md#step-1----confirm-bigquery-job-pattern-match), [Step 2 (`setOwner`)](../../../../docs/OCP_CI_Tutorials/Reporting/Reporting_Guide.md#step-2----map-ci-operator-job-name-to-a-cr-variant-owner), [Step 6 (`testSuitePatterns`)](../../../../docs/OCP_CI_Tutorials/Reporting/Reporting_Guide.md#step-6----confirm-ts-import-pattern-coverage).

**Maintainer hand-off:** `make update-variants` then `./sippy variants snapshot --config ./config/openshift.yaml`; commit `pkg/variantregistry/snapshot.yaml`.

## PR 3: openshift-eng/ci-test-mapping (mock)

**File:** `pkg/components/lpmpexoperator/component.go`

```go
package lpmpexoperator

import (
    "regexp"

    v1 "github.com/openshift-eng/ci-test-mapping/pkg/api/types/v1"
    "github.com/openshift-eng/ci-test-mapping/pkg/config"
)

type Component struct {
    *config.Component
}

var LPmpexoperatorComponent = Component{
    Component: &config.Component{
        Name:                 "LP--MPEXOperator",
        Operators:            []string{},
        DefaultJiraComponent: "LP--MPEXOperator",
        Matchers: []config.ComponentMatcher{
            {SuiteRegEx: regexp.MustCompile(`^lp-ocp-compat--MPEXOperator--`)},
        },
    },
}

func (c *Component) IdentifyTest(test *v1.TestInfo) (*v1.TestOwnership, error) {
    if matcher := c.FindMatch(test); matcher != nil {
        jira := matcher.JiraComponent
        if jira == "" {
            jira = c.DefaultJiraComponent
        }
        return &v1.TestOwnership{
            Name:           test.Name,
            Component:      c.Name,
            JIRAComponent:  jira,
            Priority:       matcher.Priority,
            Capabilities:   append(matcher.Capabilities, identifyCapabilities(test)...),
        }, nil
    }
    return nil, nil
}

func (c *Component) StableID(test *v1.TestInfo) string {
    if stableName, ok := c.TestRenames[test.Name]; ok {
        return stableName
    }
    return test.Name
}

func (c *Component) JiraComponents() (components []string) {
    components = []string{c.DefaultJiraComponent}
    for _, m := range c.Matchers {
        components = append(components, m.JiraComponent)
    }
    return components
}
```

**File:** `pkg/components/lpmpexoperator/capabilities.go`

```go
package lpmpexoperator

import (
    v1 "github.com/openshift-eng/ci-test-mapping/pkg/api/types/v1"
    "github.com/openshift-eng/ci-test-mapping/pkg/util"
)

func identifyCapabilities(test *v1.TestInfo) []string {
    capabilities := util.DefaultCapabilities(test)
    return capabilities
}
```

**File:** `pkg/registry/registry.go` (add import and registration in `NewComponentRegistry()`)

```go
    "github.com/openshift-eng/ci-test-mapping/pkg/components/lpmpexoperator"
```

```go
    r.Register("LP--MPEXOperator", &lpmpexoperator.LPmpexoperatorComponent)
```

**File skipped:** `config/openshift-eng.yaml` (`lp-ocp-compat--%` already covers TS names).

**Maintainer hand-off:** `make mapping`; commit regenerated `mapping.json`.

## PR bodies (paste-ready)

Each upstream PR body must include the identifier table above, cross-links to the other two PRs, maintainer `make` lines, and `[ocp-ci-docs] lp-ocp-compat-cr-onboarding`.

