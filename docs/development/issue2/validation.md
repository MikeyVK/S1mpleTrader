<!-- docs\development\issue2\validation.md -->
<!-- template=validation_report version=fe38a66d created=2026-08-18T06:11Z updated=2026-08-18 -->
# Issue #2 Validation Report

**Status:** DEFINITIVE  
**Version:** 1.1  
**Last Updated:** 2026-08-18  
**Validation Outcome:** FAIL  
**Issue:** #2  
**Cycle:** Branch-wide validation  

---

## Scope

Lightweight validation of the mechanical pgmcp v2 configuration, documentation, template, and GitHub taxonomy alignment. No production Python, pytest files, helpers, or fixtures changed.

Design and planning were skipped with explicit user approval. Evidence is therefore mapped to the Approved Strategy and Expected Results in `research.md`, not to reconstructed cycle deliverables.

## Verdict

The issue #2 implementation is complete, but mandatory validation is not green:

- `run_tests(scope='full')` failed before collection because pytest is not installed.
- `run_quality_gates(scope='branch')` selected zero Python files because this branch changes configuration and documentation only; all gates were skipped.
- The project-scope fallback selected 85 real files, proving the corrected `backend/` and `tests/backend/` globs. Ruff, mypy, and Pyright were unavailable, so the tool-level PASS is not accepted as quality evidence.

## Deliverable Evidence

| Deliverable | Evidence | Result |
|---|---|---|
| Real project roots | Project structure, policies, quality globs, and test-template base paths target `backend/` and `tests/backend/`. | PASS |
| Remove imported repository assumptions | No project-specific `mcp_server/` or `tests/mcp_server/` paths remain. Two generic `mcp_server.scaffolders.generic_scaffolder` imports remain intentionally preserved. | PASS |
| Preserve generic pgmcp contracts | Work context, cached resources, and both test-template schemas remain readable; `artifacts.yaml` is unchanged. | PASS |
| Align taxonomy | All 17 configured concrete labels and current dynamic labels match live GitHub metadata; `scope:mcp-server` is absent. | PASS |
| Executable branch validation | Pytest and static-analysis tools are unavailable; branch gates are vacuous. | FAIL |

## Approved Strategy Alignment

The branch preserves product behavior, generic pgmcp runtime contracts, configuration-driven dispatch, the unchanged `artifacts.yaml` pass-through, non-destructive label identities, and the explicit prohibition on adding pytest files or artificial TDD cycles.

The user approved one narrow documentation exception: `.pgmcp/docs/reference/tools/github.md` now documents the existing label contract and correct `.pgmcp/config/labels.yaml` path. No broader bundled pgmcp documentation was changed.

## Observable Fallback

There is no trading-runtime demonstration because issue #2 changes only the development control plane. The smallest safe observation is:

1. `health_check()` confirms pgmcp loads the configuration.
2. `get_work_context()` resolves issue #2 and its workflow state.
3. Unit and integration test-template schemas load without creating files.
4. `list_labels()` returns the aligned live taxonomy.
5. Project quality selection reaches 85 repository files instead of nonexistent imported roots.

## Deferred Work for Coordination and PR Body

### Issue #3 — Reproducible Development Environment

- Install and verify pytest, Ruff, mypy, and Pyright in the supported isolated environment.
- Prove that a clean checkout can collect and run the existing suite.
- Document the supported Python and setup commands.

### Issue #4 — Quality Gates and CI

- Make missing tools and nonzero exit codes fail authoritatively.
- Define correct zero-file behavior for project and branch scopes.
- Establish the intended Pyright configuration; `pyrightconfig.json` is currently referenced but absent.
- Define coverage policy and CI execution.
- Re-run branch and project gates after issue #3 provides the toolchain.

These limitations are planned follow-up work and were not suppressed or repaired under issue #2.

### CO Triage — Lightweight Chore Workflow for Codex

**Proposed issue:** `Port and harden the lightweight chore workflow for S1mpleTrader`

Issue #2 demonstrated that the full feature workflow is disproportionate for mechanical configuration,
template, documentation, and GitHub-metadata changes. Codex followed the configured phase instructions
correctly; the overwork came from applying feature-level research, cycle-based TDD, full validation,
and documentation reconciliation to a non-behavioral maintenance change.

Coordination should triage a first-class `chore` workflow without adding another classification
dimension. The workflow type remains the single source for phase order and execution behavior.

Proposed contract:

- phases: `research → implementation → ready`;
- compact research with a checklist and Approved Strategy;
- `implementation.cycle_based: false`;
- changed-file quality gates;
- tests only when executable behavior changes;
- no mandatory validation or documentation phase for configuration-only maintenance;
- Ready must scaffold the PR body and transfer delivered scope, deferred work, and tracking state to CO;
- use YAML anchors or aliases only for contract blocks with exactly identical semantics.

Required S1mpleTrader hardening compared with the ypsia prior art:

- call `run_quality_gates(scope='files', files=[...])` using the current pgmcp tool contract;
- define test applicability explicitly by changed surface;
- do not require a Python branch gate when a configuration-only branch legitimately selects zero Python files;
- retain the existing non-destructive approval and PR-merge boundaries;
- validate the workflow against issue #2 as a retrospective case and against one behavioral issue as a regression case.

**Tracking state:** CO triage required; no dedicated S1mpleTrader issue exists yet.

References:

- [ypsia chore contract](https://github.com/MikeyVK/ypsia/blob/main/.pgmcp/config/contracts.yaml)
- [ypsia workflow registry](https://github.com/MikeyVK/ypsia/blob/main/.pgmcp/config/workflows.yaml)
- [GitHub guidance on YAML anchors and aliases](https://docs.github.com/en/actions/concepts/workflows-and-actions/reusing-workflow-configurations)
- [OpenAI guidance on lean instructions and explicit autonomy boundaries](https://developers.openai.com/api/docs/guides/latest-model)

## Evidence

- Tests: `run_tests(scope='full')` → FAIL, pytest unavailable; zero tests collected.
- Branch gates: `run_quality_gates(scope='branch')` → zero files, all gates skipped.
- Project fallback: `run_quality_gates(scope='project')` → 85 files selected; missing Ruff, mypy, and Pyright execution.
- Independent implementation QA → PASS with no gaps.

## Related Documentation

- `docs/development/issue2/research.md`
- `docs/coding_standards/QUALITY_GATES.md`
- `.pgmcp/config/quality.yaml`
- `.pgmcp/config/project_structure.yaml`
- `.pgmcp/config/labels.yaml`

---

## Version History

| Version | Date | Author | Changes |
|---|---|---|---|
| 1.0 | 2026-08-18 | Agent | Record lightweight validation and planned follow-up ownership. |
| 1.1 | 2026-08-18 | User / Agent | Add referenced chore-workflow optimization as CO-triaged deferred work. |
