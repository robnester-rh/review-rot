# EC-2275 automated PR bottleneck baseline

Snapshot time: 2026-11-18 03:03 UTC

## Scope and method

The inventory covers open pull requests authored by `renovate[bot]` in the
`conforma` and `enterprise-contract` organizations. GitHub search returned 75
Conforma PRs and 6 Enterprise Contract PRs. PR age is measured in whole days
from `createdAt` to the snapshot time. Go classifications use PR titles and
labels for triage; they are not a substitute for inspecting the Renovate
branch or changed files. Security is an overlapping label, while update kind
and module PR shape are reported as separate dimensions.

## Baseline

| Scope | Open Renovate PRs | Oldest PR | Largest repository backlog |
| --- | ---: | ---: | --- |
| `conforma` | 75 | 390 days | `conforma/cli` (35), `conforma/policy` (14) |
| `enterprise-contract` | 6 | 388 days | `enterprise-contract/enterprise-contract-controller` (6) |
| **Total** | **81** | **390 days** | |

The largest Conforma Renovate queues after CLI and policy are
`conforma/conforma.github.io` (5), `conforma/go-gather` (4), and
`conforma/policy-builder` and `conforma/user-guide` (3 each). These figures
are Renovate-only; they should not be mixed with counts of PRs from other
bots.

## Go update classification

### Update kind

| Update kind | `conforma` | `enterprise-contract` | Total | Oldest age |
| --- | ---: | ---: | ---: | ---: |
| Go module | 31 | 2 | 33 | 388 days |
| Go toolchain | 5 | 2 | 7 | 258 days |
| Go image | 4 | 2 | 6 | 258 days |
| **Go-related total** | **40** | **6** | **46** | **388 days** |

### Go module PR shape

| Module PR shape | `conforma` | `enterprise-contract` | Total | Oldest age |
| --- | ---: | ---: | ---: | ---: |
| Grouped modules | 14 | 2 | 16 | 388 days |
| Single-dependency modules | 17 | 0 | 17 | 144 days |
| **Go module total** | **31** | **2** | **33** | **388 days** |

Sixteen Go-related PRs are marked as security updates in `conforma`: 10 have
the `security` label and 6 more have a `[SECURITY]` title marker (oldest 35
days). This is a subset of the update-kind and module-shape counts above, not
an additional category. The 46 Go-related PRs were identified by Go-specific
labels or title terms such as `golang`, `go modules`, `go toolchain`, `gomod`,
`.mod`, or `gopkg.in`; grouped module PRs were identified from their group
titles. Borderline cases should be confirmed from changed files before
changing policy.

Repository view for Go-related PRs: `conforma/cli` has 27 (oldest 264 days),
`conforma/policy` has 10 (oldest 303 days), `conforma/go-gather`,
`conforma/e2e-tests`, and `conforma/crds` have one each, and
`enterprise-contract/enterprise-contract-controller` has 6 (oldest 388 days).

## Required versus optional checks

Required contexts are repository-specific. The observed branch-protection
contexts include:

| Repository | Required context(s) |
| --- | --- |
| `conforma/policy` | `all-tests-and-checks` |
| `conforma/policy-builder` | `build` |
| `conforma/crds` | `Analyze (go)`, `Checks`, `CodeQL` |
| `conforma/go-gather` | `Commit-Lint`, `Test` |
| `conforma/step-actions` | `integration-tests`, `unit-tests` |
| `conforma/pr-size-label-action` | `Analyze (actions)`, `Analyze (javascript-typescript)`, `CodeQL` |
| `enterprise-contract/enterprise-contract-controller` | `Checks`, `Analyze (go)` |

Several repositories, including `conforma/cli`, reported no classic branch
protection status-check contexts through the API; their merge gating needs to
be checked through rulesets or repository-level policy rather than assumed
from check names.

All six open Renovate PRs in `enterprise-contract/enterprise-contract-controller`
currently fail required `Checks` while `Analyze (go)` passes. The six are two
Go image updates ([#755](https://github.com/enterprise-contract/enterprise-contract-controller/pull/755),
[#753](https://github.com/enterprise-contract/enterprise-contract-controller/pull/753)),
two toolchain updates ([#754](https://github.com/enterprise-contract/enterprise-contract-controller/pull/754),
[#752](https://github.com/enterprise-contract/enterprise-contract-controller/pull/752)),
and two grouped module updates ([#638](https://github.com/enterprise-contract/enterprise-contract-controller/pull/638),
[#634](https://github.com/enterprise-contract/enterprise-contract-controller/pull/634)).
They range from 257 to 388 days old, making the failing `Checks` job a
concrete, concentrated bottleneck to diagnose.

Representative aged PRs:

- `conforma/cli#3366`: failed `Test`, required review; blocked.
- `conforma/policy#1632`: failed `all-tests-and-checks`, required review.
- `conforma/policy-builder#43`: failed required `build`, required review; blocked.
- `conforma/conforma.github.io#481`: failed `build-hugo`, required review; blocked.
- `conforma/cli#3074`: failed checks, required review, and dirty merge state;
  this is a conflict/staleness candidate in addition to CI.
- `conforma/policy#1899`: no failed checks in the sampled result but still
  required review, demonstrating that green CI does not imply mergeable.

## Recommended priorities

1. Start with `conforma/cli`: it has the largest Go queue (27), and 16 of the
   19 failed-required-check PRs in the earlier sample were in this repository.
2. Diagnose the controller's required `Checks` job: all six of its aged Go
   PRs fail this one context while `Analyze (go)` passes.
3. Triage `conforma/policy` next: it has 10 open Go PRs and accounted for 3 of
   the 19 failed-required-check PRs in the earlier sample.
4. Separate review-required from CI-failed PRs before changing automation;
   the `ready-for-merge` backlog shows that reviewer/merge policy is actionable
   independently of test failures.
5. Treat `conforma/cli` as a ruleset investigation, not a branch-protection
   investigation, because classic required contexts are absent from the API
   response.
6. Review abandoned security and release-branch PRs for closure or replacement
   rather than spending review capacity on stale branches.

## Sources

- [Conforma open Renovate PRs](https://github.com/conforma/review-rot)
- [Enterprise Contract controller PRs](https://github.com/enterprise-contract/enterprise-contract-controller/pulls)
- [Enterprise Contract controller PR #755](https://github.com/enterprise-contract/enterprise-contract-controller/pull/755)
- [Enterprise Contract controller PR #754](https://github.com/enterprise-contract/enterprise-contract-controller/pull/754)
- [Enterprise Contract controller PR #753](https://github.com/enterprise-contract/enterprise-contract-controller/pull/753)
- [Enterprise Contract controller PR #752](https://github.com/enterprise-contract/enterprise-contract-controller/pull/752)
- [Enterprise Contract controller PR #638](https://github.com/enterprise-contract/enterprise-contract-controller/pull/638)
- [Enterprise Contract controller PR #634](https://github.com/enterprise-contract/enterprise-contract-controller/pull/634)
- [Conforma CLI PR #3366](https://github.com/conforma/cli/pull/3366)
- [Conforma policy PR #1632](https://github.com/conforma/policy/pull/1632)
- [Conforma policy-builder PR #43](https://github.com/conforma/policy-builder/pull/43)
- [Conforma CLI PR #3074](https://github.com/conforma/cli/pull/3074)

## Follow-up: Go Renovate mitigation (2026-11-17)

The current shared Renovate configuration groups Go modules, runs
`gomodTidy`, and inherits `:automergePatch`. That preset covers patch,
pin, and lock-file-maintenance updates; minor updates are not covered by it.
Major updates remain manual. See the [Renovate preset definition](https://docs.renovatebot.com/presets-default/#automergepatch).

The shared auto-merge workflow only enables GitHub auto-merge after a
collaborator approves a Renovate PR. Required reviews therefore remain a
separate gate even when Renovate has automerge enabled. See [Renovate's
required-review guidance](https://docs.renovatebot.com/key-concepts/automerge/#required-pull-request-reviews).

The organization baseline already lists the Renovate GitHub App as a
`pull_request` bypass actor. The sampled Go module minor PRs
([cli#3738](https://github.com/conforma/cli/pull/3738),
[cli#3730](https://github.com/conforma/cli/pull/3730),
[cli#3729](https://github.com/conforma/cli/pull/3729),
[policy#1966](https://github.com/conforma/policy/pull/1966)) and Go toolchain
minor PR [cli#3672](https://github.com/conforma/cli/pull/3672) say
`Automerge: Disabled by config`; Go module patch PR
[cli#3716](https://github.com/conforma/cli/pull/3716) says automerge is
enabled but has failed required checks. The shared patch preset does not
enable minor updates.

A live Conforma search found 40 open Renovate PRs with Go/module/Golang in
their titles or Go labels; 21 were labeled `ready-for-merge` and still
required a review. A separate check-context snapshot on 2026-11-17, compared
with the active rulesets, found 19 with a failed required check. Four had no
failed required check, but all
four had a separate failed status: `conforma/cli`
[#3738](https://github.com/conforma/cli/pull/3738),
[#3730](https://github.com/conforma/cli/pull/3730), and
[#3729](https://github.com/conforma/cli/pull/3729) had a failed Konflux
status that is not required for their release branches; `conforma/policy`
[#1966](https://github.com/conforma/policy/pull/1966) targets `release-v0.8`,
where neither rulesets nor classic branch protection require status checks.
All four still had `REVIEW_REQUIRED` and `BLOCKED` merge states. The
`ready-for-merge` label does not identify which gate is currently blocking a
PR.

Ruleset inspection found that both `conforma/cli` and `conforma/policy`
inherit the organization baseline requiring two approvals, code-owner review,
approval of the last push, and resolved review threads. The CLI also has
branch-specific required checks; policy's main branch requires
`all-tests-and-checks`, `codecov/patch`, and `codecov/project`.

**Recommendation:** don't add a duplicate Go patch automerge rule. Most of
the sampled queue still has required-check failures. The four with no failed
required check are security updates with a separate failed status, so they
are not a low-risk pilot. A Go-module minor auto-merge rule for stable
dependencies could address future non-security updates; automerging
vulnerability alerts needs a separate, explicit decision. Either choice lets
Renovate use its existing review bypass. Keep major updates manual and
required checks mandatory, and handle Go toolchain updates separately. The
optional release-branch failures on the current examples also need an
explicit decision.

## Follow-up: failed required checks (2026-11-17)

The 19 PRs have overlapping failures. The main recurring causes are Go
version drift across files and compile/type-check failures from grouped
dependency updates.

By required context, the failures are: `Test` (12 PRs), `Acceptance` (9),
`Lint` (4), `Tools` (3), `Analyze (go)` (1), policy
`all-tests-and-checks` (3), and CLI main-branch Konflux checks (7). These
counts overlap; several PRs fail multiple required contexts.

- **Go version consistency (`Test`, 12 CLI PRs):** [#3672](https://github.com/conforma/cli/pull/3672), [#3671](https://github.com/conforma/cli/pull/3671), [#3669](https://github.com/conforma/cli/pull/3669), [#3665](https://github.com/conforma/cli/pull/3665), [#3617](https://github.com/conforma/cli/pull/3617), [#3613](https://github.com/conforma/cli/pull/3613), [#3605](https://github.com/conforma/cli/pull/3605), [#3554](https://github.com/conforma/cli/pull/3554), [#3369](https://github.com/conforma/cli/pull/3369), [#3365](https://github.com/conforma/cli/pull/3365), [#3131](https://github.com/conforma/cli/pull/3131), and [#3130](https://github.com/conforma/cli/pull/3130). `Check go versions` rejects inconsistent pins. Toolchain PRs update only `.tool-versions`; module PRs raise `go.mod` directives while Dockerfiles, Containerfiles, or sibling modules retain older versions.
- **Dependency API incompatibilities:** [CLI #3719](https://github.com/conforma/cli/pull/3719), [#3717](https://github.com/conforma/cli/pull/3717), and [#3716](https://github.com/conforma/cli/pull/3716) fail type checks because Cosign's attestation type no longer supports the existing type assertion. [#3669](https://github.com/conforma/cli/pull/3669), [#3131](https://github.com/conforma/cli/pull/3131), and [#3130](https://github.com/conforma/cli/pull/3130) expose structured-merge-diff v7/v6 type conflicts; #3669 and #3130 also use Rekor generated parameter fields (`Context`) absent from the updated API. These errors appear across `Acceptance`, `Lint`, `Tools`, and stress-benchmark build jobs.
- **Lint tool compatibility:** [CLI #3369](https://github.com/conforma/cli/pull/3369) and [#3130](https://github.com/conforma/cli/pull/3130) fail because the type checker cannot read Go export data version 5 (its maximum is version 4). #3131's lint and CodeQL jobs fail while downloading Go dependencies, without a diagnostic explaining why.
- **Policy module sums:** [policy #1964](https://github.com/conforma/policy/pull/1964) fails `all-tests-and-checks` with missing `go.sum` entries for packages in the updated dependency set.
- **Policy aggregate failures without detail:** [policy #1686](https://github.com/conforma/policy/pull/1686) and [#1632](https://github.com/conforma/policy/pull/1632) fail `all-tests-and-checks`, but their run summaries expose no underlying test or compile diagnostic.
- **Konflux on CLI `main`:** [CLI #3728](https://github.com/conforma/cli/pull/3728), [#3605](https://github.com/conforma/cli/pull/3605), [#3554](https://github.com/conforma/cli/pull/3554), [#3369](https://github.com/conforma/cli/pull/3369), [#3365](https://github.com/conforma/cli/pull/3365), [#3131](https://github.com/conforma/cli/pull/3131), and [#3130](https://github.com/conforma/cli/pull/3130) have failing required pipeline checks. Their details link to the internal Konflux UI, which requires employee SSO, so causes remain unverified.

Some acceptance failures on CLI #3671, #3613, and #3369 expose only a
process exit, with no diagnostic in the run summary. The stress benchmark
checks on #3131 and #3130 stop while compiling the same dependency API
errors; the benchmark comparison itself does not run.

**Assessment:** these are active merge gates, not review-only backlog.
Automerge policy changes would leave these PRs blocked until the Go version
pins and dependency compatibility issues are resolved; keep required checks
enabled.
