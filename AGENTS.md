# S1mpleTrader — pgmcp v2 Agent Protocol

**Auto-loaded by VS Code / Google Antigravity** via `chat.useAgentsMdFile: true` — single always-on instruction file.
**Status:** Active | **Context:** S1mpleTrader V3 plugin-first, event-driven trading platform | **pgmcp:** 2.0.0

---

## 🏛️ Architecture Contract (MANDATORY)

**Before writing any implementation code, read:**
**[docs/coding_standards/ARCHITECTURE_PRINCIPLES.md](docs/coding_standards/ARCHITECTURE_PRINCIPLES.md)**

This document is a **binding contract**. Code that violates these principles is **REJECTED**, regardless of whether tooling gates pass.

### Most common violations (quick reference):

| Violation | Correct pattern |
|---|---|
| Hardcoded phase/workflow names in Python | Read from config (WorkflowConfig / GitConfig) |
| `SomeManager()` inside `execute()` | Constructor injection via `__init__` |
| Write-capable interface for read-only consumer | Use narrow read-only interface (ISP) |
| `get_state()` calls `save()` | CQS violation — split the method |
| Module-level `Config.load()` | `ClassVar` + lazy init in `__init__` |
| if-chain on `phase == "implementation"` etc. | Registry or config-driven dispatch (OCP) |
| Value object without `frozen=True` | Add `@dataclass(frozen=True)` or `ConfigDict(frozen=True)` |
| Issue number extracted in state engine | Delegate to git conventions config class |

---

## 🔧 Tool Priority Matrix (MANDATORY)

**Never use `run_in_terminal` or default/built-in agent tools (e.g., `write_to_file`, `replace_file_content`, `multi_replace_file_content`) for these operations — use MCP tools instead:**

### Git Operations
| Action | ✅ USE THIS | ❌ NEVER USE |
|--------|-------------|------------|
| Create branch | `create_branch(name, base_branch, branch_type)` | `run_in_terminal("git branch")` |
| Commit | `git_add_or_commit(workflow_phase, message)` | `run_in_terminal("git commit")` |
| Checkout | `git_checkout(branch)` | `run_in_terminal("git checkout")` |
| Push | `git_push(set_upstream)` | `run_in_terminal("git push")` |
| Merge | `git_merge(branch)` | `run_in_terminal("git merge")` |
| Delete branch | `git_delete_branch(branch, force, mode)` | `run_in_terminal("git branch -d")` |
| Stash | `git_stash(action, message)` | `run_in_terminal("git stash")` |
| Status | `git_status()` | `run_in_terminal("git status")` |
| Restore | `git_restore(files, source)` | `run_in_terminal("git restore")` |
| Fetch | `git_fetch(remote, prune)` | `run_in_terminal("git fetch")` |
| Pull | `git_pull(remote, rebase)` | `run_in_terminal("git pull")` |
| List branches | `git_list_branches(verbose, remote)` | `run_in_terminal("git branch -a")` |
| Diff stats | `git_diff_stat(target_branch, source_branch)` | `run_in_terminal("git diff --stat")` |
| Verify merge reachability | `check_merge(merge_sha)` | `run_in_terminal("git merge-base --is-ancestor")` |

### GitHub Operations
| Action | ✅ USE THIS | ❌ NEVER USE |
|--------|-------------|------------|
| Create issue | `create_issue(title, body, issue_type, priority, scope, ...)` | `run_in_terminal("gh issue create")` |
| Get issue | `get_issue(issue_number)` | `run_in_terminal("gh issue view")` |
| List issues | `list_issues(state, labels)` | `run_in_terminal("gh issue list")` |
| Update issue | `update_issue(issue_number, ...)` | `run_in_terminal("gh issue edit")` |
| Close issue | `close_issue(issue_number, comment)` | `run_in_terminal("gh issue close")` |
| Create PR (atomic) | `submit_pr(title, body, head, base, draft)` | `run_in_terminal("gh pr create")` |
| List PRs | `list_prs(state, base, head)` | `run_in_terminal("gh pr list")` |
| Merge PR | `merge_pr(pr_number, commit_message, merge_method)` | `run_in_terminal("gh pr merge")` |
| Get PR | `get_pr(pr_number)` | `run_in_terminal("gh pr view")` |
| Create label | `create_label(name, color, description)` | Manual GitHub UI |
| Add labels | `add_labels(issue_number, labels)` | `run_in_terminal("gh issue edit")` |
| Create milestone | `create_milestone(title, description, due_on)` | Manual GitHub UI |

### File Operations
| Action | ✅ USE THIS | ❌ NEVER USE |
|--------|-------------|------------|
| Edit file | `safe_edit_file(path, operation, mode)` | `run_in_terminal("Set-Content")` |
| Scaffold code/docs | `scaffold_artifact(artifact_type, name, context)` | Manual creation |
| Inspect artifact context schema | `scaffold_schema(artifact_type)` | Guessing context fields or trial-and-error calls |

### Quality & Testing
| Action | ✅ USE THIS | ❌ NEVER USE |
|--------|-------------|------------|
| Run quality gates | `run_quality_gates(scope, files, verbose)` | `run_in_terminal("pylint")` or `run_in_terminal("mypy")` |
| Run tests | `run_tests(path, markers, timeout, verbose)` | `run_in_terminal("pytest")` |
| Validate template | `validate_template(path, template_type)` | Manual review |

### Project & Phase Management
| Action | ✅ USE THIS | ❌ NEVER USE |
|--------|-------------|------------|
| Initialize project | `initialize_project(issue_number, issue_title, workflow_name)` | Manual .pgmcp/ file creation |
| Get project plan | `get_project_plan(issue_number)` | Manual .pgmcp/ file reading |
| Transition phase | `transition_phase(branch, to_phase)` | Manual .pgmcp/state.json edit |
| Force phase transition | `force_phase_transition(branch, to_phase, skip_reason, human_approval_message)` | Manual .pgmcp/state.json edit |

### Discovery & Admin
| Action | ✅ USE THIS | ❌ NEVER USE |
|--------|-------------|------------|
| Search docs | `search_documentation(query, scope)` | `grep_search` on docs/ |
| Get work context | `get_work_context()` | Manual file reading |
| Health check | `health_check()` | N/A |
| Restart server | `restart_server(reason)` | Process kill |

---

## 🚫 run_in_terminal Restrictions (CRITICAL)

**`run_in_terminal` is ONLY allowed for:**

✅ **Permitted (rare cases):**
- Development servers where no MCP tool exists (e.g., `npm run dev`, `python -m http.server`)
- Build commands explicitly requested by user
- Smoke tests / exploratory commands approved by user
- Python package installations via pip (when not using install_python_packages tool)

❌ **FORBIDDEN (use MCP tool instead):**
- **File operations** → use `safe_edit_file` / `scaffold_artifact`
- **Git operations** → use `git_*` tools (see matrix above)
- **Test execution** → use `run_tests` tool
- **Quality gates** → use `run_quality_gates` tool

**Default rule: If unsure, ask yourself "Is there an MCP tool for this?" If yes → use it. If no → ask user permission first.**

---

## 🔴 TDD Cycle (RED → GREEN → REFACTOR)

**Strict protocol — never skip steps:**

1. **RED Phase:** Write failing test FIRST
   - Commit: `git_add_or_commit(workflow_phase="implementation", sub_phase="red", cycle_number=1, message="Add failing test for X")`
   - Verify test fails: `run_tests(path="...")`

2. **GREEN Phase:** Implement minimal code to pass test
   - Commit: `git_add_or_commit(workflow_phase="implementation", sub_phase="green", cycle_number=1, message="Implement X")`
   - Verify test passes: `run_tests(path="...")`

3. **REFACTOR Phase:** Improve code while keeping tests green
   - Commit: `git_add_or_commit(workflow_phase="implementation", sub_phase="refactor", cycle_number=1, message="Refactor X")`
   - Verify tests still pass: `run_tests(path="...")`

4. **DOCUMENTATION Phase:** Update documentation
   - Commit: `git_add_or_commit(workflow_phase="documentation", message="Document X")`

**Quality Gates:** Run `run_quality_gates(files=[...])` before phase transitions and before PR creation.

---

## ⚖️ Prime Directives

1. **Issue-First Development:** Never work directly on `main`. Always start with `create_issue` → `create_branch` → `initialize_project`.
2. **Workflow Enforcement:** Always `initialize_project` before work. Use `transition_phase` for progression.
3. **TDD is Non-Negotiable:** If you write code without a test, you are violating protocol.
4. **Tools > Manual:** Never manually create a file if `scaffold_artifact` exists. Never manually parse status if `get_work_context` exists.
5. **English Artifacts, Dutch Chat:** Write Code/Docs/Commits in **English**. Talk to the User in **Dutch** (Nederlands).
6. **Human-in-the-Loop:** PR merge ALWAYS requires human approval. `force_phase_transition` requires approval + reason.
7. **Quality Gates:** Run before phase transitions and before PR creation. Linting 10.00/10 + Type checking Pass.
8. **Type-Checking Consistency:** Resolve typing issues using [docs/coding_standards/TYPE_CHECKING_PLAYBOOK.md](docs/coding_standards/TYPE_CHECKING_PLAYBOOK.md). No global disables; targeted ignores only as last resort.
9. **Resource Caching:** All MCP tools cache their structured Pydantic DTO outputs as MCP Resources (`pgmcp://cache/runs/{run_id}`). Tools return a presented text summary and the resource URI. When you need to inspect complete structured data or verbose process logs (e.g. from `run_quality_gates` or `run_tests`), you MUST read the cached resource URI (do not try to parse or scrape the text output).
10. **Strict Tool Abstraction (pgmcp):** Do not read or search the source code of the `pgmcp` server (`.venv/Lib/site-packages/mcp_server/`) during normal operation. Rely exclusively on the documented MCP tools, schemas, and reference documents. Only access the server code when explicitly debugging a server crash/error and requested by the user.

---

## 🧭 Strategy Approval Gate (MANDATORY)

Compatibility, migration, and breakage strategy is decided at the end of Research, not later.

- Research must identify the affected boundaries, consumers, strategy options, and the cost / risk / impact trade-offs for each relevant boundary.
- Research must not close until the human decision is captured as an Approved Strategy.
- The Approved Strategy must be explicit per affected boundary, not left as a vague issue-wide assumption.
- Design may not start until an Approved Strategy exists for the boundaries it will shape.
- Planning, implementation, and QA must treat the Approved Strategy as binding input.
- No later phase may silently switch between preserve compatibility, temporary bridge, or clean break.
- If later evidence makes the Approved Strategy unsound, stop and reopen the decision explicitly instead of changing strategy by stealth.

---

## 📋 Workflow Types

| Workflow | Phases | Use Case |
|----------|--------|----------|
| `feature` | research, design, planning, implementation, validation, documentation, ready | New feature development |
| `bug` | research, design, planning, implementation, validation, documentation, ready | Bug fixes |
| `docs` | planning, documentation, ready | Documentation-only changes |
| `refactor` | research, planning, implementation, validation, documentation, ready | Code refactoring |
| `hotfix` | implementation, validation, documentation, ready | Emergency fixes |
| `epic` | See `.pgmcp/config/contracts.yaml` (SSOT for epic phase order) | Large multi-issue features |
| `custom` | (user-defined) | Custom workflows |

**Workflow Selection:** Use `initialize_project(issue_number, issue_title, workflow_name="feature|bug|docs|...")` to start.

---

## 🏗️ Scaffolding (Always Use Templates)

**NEVER use `safe_edit_file` to create code or documentation from scratch. Always use `scaffold_artifact`.**

| Example Artifact Type | Use Case | Example |
|---------------|----------|---------|
| `dto` | Data Transfer Objects | `scaffold_artifact(artifact_type="dto", name="UserDTO", context={...})` |
| `worker` | Background processors | `scaffold_artifact(artifact_type="worker", name="ProcessWorker", context={...})` |
| `tool` | MCP tools | `scaffold_artifact(artifact_type="tool", name="MyTool", context={...})` |
| `research` | Research documents | `scaffold_artifact(artifact_type="research", name="my-research", context={...})` |
| `design` | Design documents | `scaffold_artifact(artifact_type="design", name="my-design", context={...})` |
| `reference` | Reference docs | `scaffold_artifact(artifact_type="reference", name="my-reference", context={...})` |

These are representative examples, not the complete registry. Current first-class types also include `adapter`, `resource`, `interface`, `service`, `schema`, `generic`, `unit_test`, `integration_test`, `architecture`, `planning`, `validation_report`, `generic_doc`, `commit`, `pr`, and `issue`.

**Registry:** `.pgmcp/config/artifacts.yaml` defines the authoritative complete set of artifact types and their templates.

**Schema discovery:** Before calling `scaffold_artifact` with an artifact type whose context fields are not already in your working context, call `scaffold_schema(artifact_type=...)` first. It returns the full JSON Schema for the `context` parameter — required and optional fields — enabling first-time-right scaffolding without a failed call. If you call `scaffold_artifact` with wrong or missing context fields, the error response contains the same schema; use it to correct the call immediately.

---

## 🤝 Three-Agent Model & Standalone Research

This project uses three specialized agents in separate VS Code chat sessions to prevent role contamination and context pollution, along with a standalone research agent for deep exploratory work.

### Roles

| Agent | Role | Allowed operations |
|-------|------|--------------------|
| `@co` | Coordination authority and epic workflow owner | Read all; issue/label/milestone admin; epic docs/contracts/prompts edits; epic lifecycle mutations, phase transitions, commits, quality gates, PR submission, and merge within the approved narrow allowlist |
| `@imp` | Child-issue implementation executor | Production code and test work on non-epic branches; cycle execution; commits; phase and cycle transitions |
| `@qa` | QA authority — read-only | Read files; run tests; run quality gates. **No edits, no commits** |
| `@research` | Standalone Research Agent | Read all; web searching; explore codebase and documentation. Operates outside phase-gate workflows. **No edits, no commits** |

### Sub-roles

**`@co` sub-roles:** coordination: `triager` (default), `backlog-reviewer`, `tracker`, `issue-author`; epic lifecycle: `epic-researcher`, `epic-planner`, `epic-designer`, `epic-coordinator`, `epic-documenter`, `epic-releaser`  
**`@imp` sub-roles:** `researcher` (default), `planner`, `designer`, `implementer`, `validator`, `documenter`  
**`@qa` sub-roles:** `design-reviewer` (default), `plan-verifier`, `verifier`, `validation-reviewer`, `doc-reviewer`

Declare your active sub-role in the invocation text.  
Example: `@imp implementer: start cycle C_LOADER.5 for issue 257`

### @co Operating Modes

- **Owned-branch epic execution:** `@co` owns the epic branch end-to-end and may edit epic docs/contracts/prompts, perform lifecycle mutations, phase transitions, commits, quality gates, PR submission, and merge after approval.
- **Background coordination:** `@co` reads status, updates issue coordination state, and hands child technical work to `@imp` without taking over the implementation branch.

### Two-Chat Model

- **Coordination / epic ownership** → use `@co`. Use `@co` either for owned-branch epic execution or for background coordination around child work. Produce a Co → Imp hand-over only when delegating child technical implementation.
- **Implementation** → use `@imp` for child technical work. Execute the current cycle. Produce an Imp → QA hand-over.
- **Review** → use `@qa`. Findings on epic-owned branches route back to `@co`; findings on child technical work route back to `@imp`.

Never mix roles in one session. Fresh context prevents scope contamination and authority confusion.
### Codex Skill and Workflow Mapping

| Antigravity surface | Codex surface |
|---|---|
| Agent `@co` | Top-level skill [`pgmcp-co`](.agents/skills/pgmcp-co/SKILL.md) |
| Agent `@imp` | Top-level skill [`pgmcp-imp`](.agents/skills/pgmcp-imp/SKILL.md) |
| Agent `@qa` | Top-level skill [`pgmcp-qa`](.agents/skills/pgmcp-qa/SKILL.md) |
| Agent `@research` | Top-level skill [`pgmcp-research`](.agents/skills/pgmcp-research/SKILL.md) |
| Workflow `create-issue` | Internal `pgmcp-co` workflow reference |
| Workflow `start-issue` | Internal `pgmcp-co` workflow reference |
| Workflow `end-issue` | Internal `pgmcp-co` workflow reference |
| Workflow `go` | Internal `pgmcp-imp` workflow reference |
| Workflow `start` | Startup contract in this file and the selected role skill |

`.agents/workflows/` is the single procedural source for both Antigravity and Codex. Codex role
skills route to those files as internal workflow references; they are not independently
discoverable Codex skills or slash commands. Do not copy their procedure text into role skills.

### Startup Protocol

Each agent has its own startup protocol defined in its `.agent.md` file and corresponding Codex
role skill. Normal chat sessions call `get_work_context` as the first tool invocation.
`start-issue` and `end-issue` are explicit lifecycle-boundary exceptions that may run their
scripted bootstrap or exit sequence before control returns to a normal
`get_work_context`-first session. `create-issue` is normal coordination work and therefore loads
`get_work_context` first. See:
- [`@co` startup](.agents/rules/co.agent.md)
- [`@imp` startup](.agents/rules/imp.agent.md)
- [`@qa` startup](.agents/rules/qa.agent.md)

### Hand-Over Contract

Use Co → Imp only for child technical delegation. Epic-owned branch review and lifecycle continuation stay with `@co`; QA findings and merge follow-up on those branches route back there.

**Co → Imp hand-over:**
```text
## Co → Imp Hand-over

**Directive**: [what to do]
**Issues in scope**: [#N, #M]
**Priority changes applied**: [yes/no, which labels]
**Next @imp sub-role**: [researcher | planner | implementer | ...]
**Out of scope**: [what not to touch]
```

**Imp → QA hand-over:**
```text
### Scope
- what cycle or task was executed
- what was intentionally kept out of scope

### Files
- changed files grouped by role

### Deliverables
- which authoritative deliverables are now satisfied

### Stop-Go Proof
- exact tests run
- exact gate commands or MCP checks run
- exact outcome
```

---

## 🏗️ Project Context & Architecture: S1mpleTrader

**What it is:** S1mpleTrader V3 is a plugin-first, event-driven trading platform that supports the strategy lifecycle from market-data ingestion and objective context calculation through signal and risk detection, strategy and trade planning, execution, and state tracking. The platform is designed for modularity, isolated testing, configuration-driven behavior, and execution-environment independence across backtest, paper, and live operation.

### Binding Domain Principles

- **Plugin first:** Strategy logic belongs in independently testable worker plugins. The platform supplies infrastructure and must not hardcode strategy behavior.
- **Objective facts, subjective consumers:** ContextWorkers calculate objective market facts. SignalDetectors, RiskMonitors, and StrategyPlanners interpret those facts according to their own strategy logic. Do not reintroduce SWOT classification or a context-aggregation opinion layer.
- **Point-in-time data:** Each strategy run operates on validated Pydantic DTOs anchored to one point in time. Do not mutate or pass growing DataFrames through a worker chain.
- **Two communication paths:** Use `IStrategyCache` for synchronous worker-to-worker flow data and `DispositionEnvelope` through EventAdapters/EventBus for asynchronous signals and platform communication.
- **Flat event orchestration:** Operators are obsolete. EventAdapters are the orchestration boundary, workers remain bus-agnostic, and wiring is explicit and configuration-driven.
- **BuildSpec-driven bootstrap:** Configuration flows through loader, validator, translator, BuildSpecs, and factories. Validate dependencies and event topology at bootstrap; do not parse YAML or discover structural errors during worker execution.
- **Contract driven:** Pydantic DTOs and narrow protocols define component boundaries. Prefer immutable value objects, dependency injection, fail-fast validation, and explicit causality/origin metadata.
- **Environment agnostic:** Strategy and execution-planning logic must not branch on backtest, paper, or live environments. Environment-specific behavior belongs behind injected connectors and environment interfaces.

### Platform and Pipeline Boundaries

| Concern | Authoritative pattern |
|---|---|
| Strategy run state | `StrategyCache` / `IStrategyCache`, reconfigured per run |
| Event communication | `EventBus` with scoped subscriptions and one EventAdapter per component |
| Data-run initialization | `FlowInitiator` establishes the run anchor before workers execute |
| Worker output | `DispositionEnvelope`: `CONTINUE`, `PUBLISH`, or `STOP` |
| Strategy flow | Objective context → signal/risk detection → strategy planning → entry/size/exit/execution planning → execution |
| Trade state | Typed state DTOs for plans, execution groups, orders, and fills with causality preserved |
| Configuration | Human-readable YAML translated once into validated BuildSpecs |

### Repository Scope

- `backend/dtos/` contains the shared, strategy, execution, and state contracts.
- `backend/core/` contains platform primitives such as EventBus, StrategyCache, FlowInitiator, enums, and narrow interfaces.
- `backend/workers/` and `backend/services/` contain worker and service behavior; keep responsibilities separated from platform infrastructure.
- `tests/backend/` mirrors backend boundaries and is the default pytest discovery scope configured by `pyproject.toml`.
- `docs/architecture/` is the current S1mpleTrader architecture source; the large V2 system document is historical context where V3 architecture documents supersede it.
- `.pgmcp/` contains pgmcp v2 workflow configuration, contracts, templates, and bundled reference material. Do not confuse the pgmcp server version with the S1mpleTrader product version.

### Development Model

- `main` is protected and expected to remain stable.
- Supported pgmcp workflows are `feature`, `bug`, `hotfix`, `refactor`, `docs`, and `epic`; phase order comes from `.pgmcp/config/contracts.yaml`.
- `.pgmcp/config/workphases.yaml` is authoritative for phase metadata and allowed commit sub-phases.
- Milestones are supported by pgmcp but no project milestones are currently configured in `.pgmcp/config/milestones.yaml`.

---

## 📚 Key Documentation

- **[Architecture Navigation](docs/architecture/README.md)** — S1mpleTrader V3 architecture reading order and system map
- **[Core Principles](docs/architecture/CORE_PRINCIPLES.md)** — Plugin-first, separation of concerns, configuration-driven, and contract-driven design
- **[Architectural Shifts](docs/architecture/ARCHITECTURAL_SHIFTS.md)** — Mandatory V2-to-V3 changes and prohibited legacy patterns
- **[Implementation Status](docs/implementation/IMPLEMENTATION_STATUS.md)** — Current backend foundation and delivery status
- **[MCP Tools Reference](.pgmcp/docs/reference/tools/README.md)** — pgmcp v2 tools, parameters, and examples
- **[Agent Instructions Model](.pgmcp/docs/reference/copilot-agent-instructions-model.md)** — How instruction files cooperate with phase-gate-mcp
- **[Quality Gates](docs/coding_standards/QUALITY_GATES.md)** — Validation standards
- **[Architecture Principles](docs/coding_standards/ARCHITECTURE_PRINCIPLES.md)** — Binding architecture contract
- **[Type Checking Playbook](docs/coding_standards/TYPE_CHECKING_PLAYBOOK.md)** — Typing issue resolution order

---

**Remember: These rules are enforced. Violations will be rejected by the user. When in doubt, consult the Tool Priority Matrix or ask the user.**
