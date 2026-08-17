<!-- docs\development\issue2\research.md -->
<!-- template=research version=8b7bb3ab created=2026-08-17T13:31Z updated= -->
# Research: Align pgmcp v2 Configuration and GitHub Taxonomy with S1mpleTrader

**Status:** APPROVED  
**Version:** 1.0  
**Last Updated:** 2026-08-17

---

## Purpose

Provide evidence-backed design and planning input for issue #2 without changing product behavior or generic pgmcp v2 lifecycle contracts.

## Scope

**In Scope:**  
Project-specific pgmcp paths, scopes, policies, artifact output paths, quality selection, GitHub label taxonomy, direct consumers, repository validation surfaces, and affected project documentation.

**Out of Scope:**  
Trading-platform redesign; product-code remediation; installed pgmcp server source changes; replacement of generic lifecycle or tool contracts; environment bootstrap or CI implementation; broad rewrites of bundled pgmcp documentation; external research.

## Prerequisites

1. Issue #2 and AGENTS.md.
2. `docs/coding_standards/ARCHITECTURE_PRINCIPLES.md`.
3. `docs/coding_standards/DOCUMENTATION_STANDARD.md`.
4. `docs/development/issue1/research.md` and its approved strategy.
5. Repository evidence gathered on `feature/2-align-pgmcp-v2-config`.

---

## Problem Statement

The checked-in control plane mixes valid generic pgmcp v2 runtime contracts with project-specific assumptions imported from the phase-gate-mcp source repository. Consequently, quality gates can inspect zero project files, path and scaffold rules name nonexistent roots, configured scopes do not describe S1mpleTrader, and GitHub contains only a partial, metadata-inconsistent subset of the configured taxonomy.

## Research Goals

- Distinguish generic pgmcp v2 contracts from S1mpleTrader-specific configuration.
- Identify affected configuration boundaries, consumers, tests, helpers, and fixtures.
- Compare compatibility and migration policies without selecting implementation details.
- Define evidence-based expected results for design and planning.
- Record assumptions, risks, and remaining design questions explicitly.

---

## Background

Epic #1 approved two relevant policies: preserve compatibility for generic pgmcp v2 lifecycle and tool contracts, and make a clean break from imported source-repository assumptions in project-specific paths, scopes, quality selection, taxonomy, and project structure. The user selected repository-only research for issue #2.

The worktree contained only pgmcp workflow metadata changes when research began: `.pgmcp/state.json` and `.pgmcp/deliverables.json`. No issue-specific QA verdict or pull request exists.

---

## Findings

### Configuration and repository mismatch

| Boundary | Observed evidence | Consequence |
|---|---|---|
| Project identity | `.pgmcp/.version` is `2.0.0`; `pyproject.toml` identifies `simpletrader-backend` 3.0.0 and discovers `backend*`. | pgmcp versioning and product structure are distinct boundaries. |
| Quality selection | `.pgmcp/config/quality.yaml` selects `mcp_server/**/*.py` and `tests/mcp_server/**/*.py`; neither root exists. Pytest discovery is `tests/backend`. | Project checks can report success without inspecting project files. |
| Type gates | Gate 4c targets `mcp_server`. Pyright references absent `pyrightconfig.json`. The DTO mypy gate and exclusion of `backend/dtos/validation_fixture_gate4.py` are project-relevant. | Path alignment must not silently settle the separate quality-policy and environment workstream. |
| Project structure | `project_structure.yaml` includes nonexistent `mcp_server` and `tests/mcp_server` trees, omits actual backend subroots, and only partly maps artifact roles. | Path validation and scaffold authorization do not consistently model this repository. |
| File policy | `policies.yaml` blocks creation under nonexistent local `mcp_server/*` paths in addition to `backend/**`. | Imported project-output restrictions require classification and alignment. |
| Test templates | Unit and integration templates emit to `tests/mcp_server/*`; their `mcp_server.scaffolders.*` modules identify the installed pgmcp runtime. | Output paths are project-specific; runtime imports are generic compatibility boundaries. |
| Artifact registry | `.pgmcp/config/artifacts.yaml` is a generic pass-through without repository-path dependencies. | Preserve the generic pgmcp v2 contract; verify absence of stale path dependencies, but do not modify this file under issue #2. |
| Scopes | `scopes.yaml` and `labels.yaml` contain six imported or generic scopes including `mcp-server`. | Issue creation and filtering lack an agreed S1mpleTrader ownership vocabulary. |
| GitHub labels | `list_labels` reports 20 labels. Only part of the configured type, priority, phase, scope, and parent taxonomy exists; custom labels generally have gray metadata instead of YAML values. | Alignment is reconciliation of partial state, not only first-time creation. |
| Milestones | `milestones.yaml` is empty and issue #2 has no milestone. | No milestone taxonomy should be invented. |

### Blast radius and consumers

| Area | Direct surfaces | Consumers and validation impact |
|---|---|---|
| Primary configuration | `.pgmcp/config/{labels,scopes,project_structure,quality,policies}.yaml` | Issue creation, path validation, file policy, gate selection, branch/project checks |
| Adjacent scaffold configuration | `.pgmcp/templates/config/{unit_test,integration_test}.yaml` | Test output placement; generic scaffolder runtime remains separate |
| Cross-config contracts | `issues.yaml`, `workphases.yaml`, `artifacts.yaml` | Required issue categories and dynamic phase validation are reference-only unless repository-specific drift is proven; `artifacts.yaml` remains an unchanged generic pass-through. |
| Production and tests | `backend/`, `tests/backend/`, `tests/backend/conftest.py` | Actual code inventory and mirrored test discovery; conftest has no substantive shared fixtures |
| Quality fixture | `backend/dtos/validation_fixture_gate4.py` | Deliberate exclusion must remain explicit rather than disappear during glob cleanup |
| Documentation | Project coding-standard references to old commands; bundled `.pgmcp/docs` | Project docs may need alignment, while generic bundled docs retain pgmcp meaning |
| GitHub | Existing default and custom labels | Issue creation, search, filters, and visible metadata |

### Existing patterns and prior art

- `quality.yaml` already models structured Ruff, mypy, and Pyright gates. Reusing this config-driven runner avoids a competing quality path.
- `pyproject.toml` is direct evidence for package discovery and pytest layout.
- `tests/backend/` mirrors backend boundaries and is the configured pytest surface.
- `issues.yaml`, `workphases.yaml`, and dynamic `parent:*` and `phase:*` patterns provide generic pgmcp lifecycle contracts.
- Epic #1 already establishes the generic-versus-project boundary and the transparent-baseline policy.

### Architectural constraints and forbidden approaches

- Configuration remains the SSOT for paths, workflow values, label categories, and gate selection.
- Invalid or empty intended scope must fail explicitly; silent or vacuous success conflicts with Fail-Fast and Explicit-over-Implicit.
- Generic lifecycle contracts, cached resources, tool schemas, and installed runtime scaffolders must not be rewritten as if they were local product modules.
- The control plane must not encode old trading architecture or choose future business capabilities.
- No production-code fixes, compatibility masking, duplicated quality runner, or pgmcp server source inspection belongs in this issue.

### Strategy-sensitive boundaries

| Boundary | Viable policies | Research recommendation |
|---|---|---|
| Generic lifecycle, tools, resources, and runtime scaffolders | Preserve compatibility; temporary bridge; clean break | Preserve compatibility, consistent with Epic #1. |
| Project paths, output paths, policies, scopes, and quality selection | Preserve; temporary bridge; clean break | Clean break from nonexistent source-repository assumptions after classifying generic references. |
| Scope vocabulary | Stable repository ownership boundaries; business-capability scopes | Prefer stable ownership boundaries; capability labels could encode unresolved product architecture. Exact names belong to design. |
| Existing default GitHub labels | Preserve alongside canonical labels; destructive cleanup | Preserve harmless defaults; removal is unnecessary for expected behavior. |
| Existing custom-label metadata | Update in place; delete/recreate; tolerate drift | Require canonical parity, subject to a safe and idempotent mechanism. |
| Intended zero-file quality selection | Fail; explicit per-gate N/A; silent success | Fail when the overall intended project or branch selection is zero. A gate may be N/A only within a nonzero selected scope. |

### Assumptions and risks

- The five files named by the issue are primary surfaces, not proof that adjacent policies, template output paths, or project documentation are out of scope.
- A broad replacement of every `mcp_server` string would break valid runtime references.
- Repository-oriented labels reduce architectural speculation but still require a human-approved vocabulary.
- Existing label tooling may not support metadata updates; destructive recreation must not be inferred.
- Quality alignment could collide with the separate environment and CI workstream if strictness or dependency policy is chosen here.
- This is configuration and repository-taxonomy setup work. New product tests, broad test matrices, artificial TDD cycles, and test-suite expansion would add scope and maintenance cost without proportionate evidence.
- Verification should reuse existing configuration validation and narrowly targeted coordination smoke checks. If executable product or pgmcp tool code becomes necessary, the issue scope and workflow approach require explicit reconsideration before implementation.
- Green results obtained by shrinking selection, adding suppressions, or treating zero files as success would violate the approved transparent-baseline strategy.

---

## Open Questions

- Which exact repository-bound scope vocabulary is authoritative for S1mpleTrader?
- Can available GitHub tooling update existing label metadata safely and idempotently?
- Which read-only and narrowly mutating coordination smoke checks are sufficient acceptance evidence?

---

## Approved Strategy

Epic-approved policies already bind issue #2:

| Boundary | Selected strategy | Constraint |
|---|---|---|
| Generic pgmcp v2 lifecycle, tool, resource, and runtime contracts | Preserve compatibility | Do not replace generic contracts or valid installed-runtime references. |
| Imported project assumptions | Clean break | Project-specific paths, scopes, quality selection, taxonomy, and structure must describe S1mpleTrader. |
| Product code | Preserve as evidence | Do not remediate production behavior under this control-plane issue. |
| Quality failures | Transparent baseline | Do not mask failures, suppress findings, or accept vacuous success. |
| Research sources | Repository-only | Make no external claims. |

Issue-level strategy approved by the user on 2026-08-17:

- Scope labels describe stable repository ownership boundaries rather than business capabilities.
- Harmless GitHub default labels remain; canonical custom labels and metadata are reconciled non-destructively where tooling permits.
- An intended project or branch quality selection that resolves to zero files fails explicitly.
- This setup-only issue adds no product tests, test matrix, or artificial TDD cycles. Evidence comes from existing config validation and minimal coordination smoke checks. Any need to change executable code reopens the scope decision.
- Issue #2 includes repository-path alignment in `policies.yaml`, test-template output paths, and directly affected project coding-standard references; generic bundled pgmcp documentation remains unchanged.
- `.pgmcp/config/artifacts.yaml` remains unchanged because it is a generic pass-through without repository-path dependencies. Verification may confirm this boundary, but may not turn the file into a project-specific registry.
- Issue #2 aligns Ruff, mypy, and Pyright file selection with `backend/` and `tests/backend/`. Tool installation, interpreter choice, dependency policy, gate strictness, and CI execution remain assigned to issues #3 and #4.
- Existing GitHub label metadata is reconciled only through safe in-place operations. If tooling cannot do that, implementation stops and records follow-up work; labels are not deleted and recreated, and metadata drift is not silently accepted.

---

## Expected Results

- Every project-specific path and direct consumer is classified without confusing it with the installed pgmcp runtime.
- Project structure, scaffold output, policy, and quality selection describe actual S1mpleTrader roots.
- Scope taxonomy represents stable ownership boundaries without selecting a trading architecture.
- Configured and GitHub type, priority, phase, parent, and scope taxonomy agree.
- Project and branch quality checks select the intended nonzero file set.
- Generic lifecycle, dynamic-label, tool/resource, cached-output, and runtime-scaffolder behavior remains compatible.
- Core coordination smoke evidence and configuration changes are documented reproducibly.

## Related Documentation

- `docs/development/issue1/research.md`
- `docs/coding_standards/ARCHITECTURE_PRINCIPLES.md`
- `docs/coding_standards/DOCUMENTATION_STANDARD.md`
- `pyproject.toml`
- `.pgmcp/config/quality.yaml`
- `.pgmcp/config/project_structure.yaml`
- `.pgmcp/config/scopes.yaml`
- `.pgmcp/config/labels.yaml`
- `.pgmcp/config/policies.yaml`
- `.pgmcp/config/artifacts.yaml`
- `.pgmcp/templates/config/unit_test.yaml`
- `.pgmcp/templates/config/integration_test.yaml`

---

## Version History

| Version | Date | Author | Changes |
|---|---|---|---|
| 0.1 | 2026-08-17 | Agent | Initial scaffold |
| 0.2 | 2026-08-17 | Agent | Add repository evidence, blast radius, strategy options, risks, and expected results |
| 1.0 | 2026-08-17 | User / Agent | Approve setup-only strategy without new tests or artificial TDD cycles |
| 1.1 | 2026-08-17 | User / Agent | Approve explicit boundary decisions; preserve `artifacts.yaml` as an unchanged generic pass-through |
