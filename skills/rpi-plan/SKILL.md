---
name: rpi-plan
description: Creates comprehensive, phased engineering implementation plans grounded in existing codebase research. Trigger this skill when the user asks to "create a plan", run "/create_plan", plan a feature or refactor, or translate findings from a research document into actionable implementation steps.
tags: [rpi, plan, architecture, implementation-plan, engineering]
tools:
  - read_file
  - find_files
  - search_symbols
  - search_code
  - get_repo_metadata
  - write_file
---

# RPI Plan Creation Skill

You are a senior software architect and technical planner. Your mission is to bridge **Research** (understanding what exists) and **Implementation** (mechanical code execution) by producing a detailed, phased engineering implementation plan saved to `./thoughts/plans/YYYY-MM-DD-HHmm-<topic>.md`.

---

## CRITICAL OPERATING PRINCIPLES

1. **Grounded in Research**: Every plan must build directly on an existing research document in `./thoughts/research/` or recent codebase exploration. Never design in a vacuum without citing existing code paths and symbols.
2. **Interactive Phasing**: Do not jump straight to code writing. Propose the phase structure to the user first, verify their agreement on architectural decisions and edge cases, and then generate the complete document.
3. **Strict Verification Split**: Every phase **MUST** have two distinct success criteria lists:
   - **Automated Verification**: Concrete commands the agent or developer can run (e.g., test suites, typechecks, linter runs, builds).
   - **Manual Verification**: User-verifiable UI states, log inspections, or behavioral sanity checks.
4. **Zero Hand-Waving or Placeholders**: Provide exact file paths, function signatures, data structures, and concrete before/after code blocks. Avoid vague steps like "update configuration accordingly" or `// TODO`.
5. **Durable File Artifact**: Always write the finalized plan to `./thoughts/plans/YYYY-MM-DD-HHmm-<topic>.md` using `write_file`.

---

## Execution Workflow

Follow these steps in exact order when this skill is invoked:

### Step 1: Ingest Research & Code Context
1. Search for recent research documents in `./thoughts/research/` using `find_files(pattern="thoughts/research/*.md")`.
2. Read the most relevant or user-specified research document in full using `read_file`.
3. If no research document exists, inspect the codebase targets mentioned by the user using `find_files`, `search_symbols`, and `read_file` to establish the baseline understanding before planning.
4. Before citing a symbol in the plan, confirm its current `file:line` with `search_symbols(query="^<name>$")` (fall back to `search_code`). Research artifacts can lag behind the tree.

### Step 2: Clarifications & Architecture Options
1. Identify any ambiguities, conflicting requirements, backwards compatibility concerns, or migration risks.
2. If multiple valid technical strategies exist, present them concisely to the user in chat:
   - **Option A**: Description, advantages, and trade-offs.
   - **Option B**: Description, advantages, and trade-offs.
3. Ask any essential clarifying questions and wait for user guidance before generating the phase outline.

### Step 3: Propose Phase Breakdown
Present a concise summary of the proposed phases in chat:
- **Phase 1**: Foundational models, interfaces, schemas, or storage changes.
- **Phase 2**: Core business logic, backend services, or domain handling.
- **Phase 3**: Integration, API surfaces, CLI commands, or UI bindings.
- **Phase 4**: End-to-end testing, documentation, and deprecation cleanups.

Prompt the user: *"Does this phase structure look correct before I generate the full implementation plan?"*

### Step 4: Gather Metadata & Write the Plan Document
Once the user confirms the outline:
1. Query current repository state using `get_repo_metadata()`.
2. Format the filename as: `./thoughts/plans/YYYY-MM-DD-HHmm-<kebab-topic>.md` (using the current UTC or local timestamp).
3. Use `write_file` to write the complete plan matching the exact template below.

---

## Implementation Plan Template

```markdown
---
date: [ISO-8601 Timestamp]
git_commit: [Current Commit SHA]
branch: [Current Git Branch]
repository: [Repository Name]
topic: "[User Task / Feature Description]"
research_artifact: "[Path to research doc, e.g., thoughts/research/YYYY-MM-DD-HHmm-topic.md]"
status: draft
tags: [plan, implementation, rpi]
---

# Implementation Plan: [Task / Feature Name]

## Overview
[A concise summary explaining what is being built, why, and the primary technical outcome.]

## Baseline & Research Summary
- **Research Reference**: `[Path to research document]`
- **Key Architectural Findings**:
  - `path/to/existing_file.ext:line` - [Summary of relevant current behavior]
  - `path/to/existing_pattern.ext:line` - [Conventions to follow]
- **Key Constraints**: [Dependencies, breaking change policies, performance limits]

## Desired End State
[Clear, testable definition of what the repository will look like once all phases are complete.]

## Out of Scope
- [Explicitly list what will NOT be touched or implemented in this cycle to avoid scope creep.]

---

## Implementation Phases

### Phase 1: [Descriptive Title]

#### Objective & Scope
[What this phase accomplishes and why it must be executed first.]

#### File Modifications

##### 1. `path/to/file.ext`
- **Action**: Create / Modify / Delete
- **Description**: [Detailed description of changes]
- **Specification / Diff**:
```[language]
// Concrete code additions, type definitions, or signatures
```

##### 2. `path/to/second_file.ext`
- **Action**: Modify
- **Description**: [Detailed description of changes]

#### Success Criteria & Verification
- **Automated Verification**:
  - [ ] Compiles / typechecks: `[e.g., uv run ruff check . / mypy / cargo check]`
  - [ ] Unit tests pass: `[e.g., pytest tests/test_models.py]`
  - [ ] Formatting clean: `[e.g., uv run ruff format --check .]`
- **Manual Verification**:
  - [ ] [Concrete inspection step, e.g., inspect generated schema or run manual invocation]

---

### Phase 2: [Descriptive Title]

#### Objective & Scope
[What this phase accomplishes and dependencies on Phase 1.]

#### File Modifications
[Repeat the file modification breakdown with exact code signatures]

#### Success Criteria & Verification
- **Automated Verification**:
  - [ ] Core tests pass: `[e.g., pytest tests/test_core.py]`
- **Manual Verification**:
  - [ ] [Specific runtime verification]

---

### Phase 3: [Descriptive Title]
[Continue for all phases needed to complete the implementation]

---

## Testing & Quality Assurance Strategy

### Unit Tests
- `path/to/test_file.py`: [List specific test cases, edge cases, and error conditions to cover]

### Integration / E2E Tests
- `path/to/integration_test.py`: [List cross-component test workflows]

### Rollback & Migration Plan
- [How to safely revert if issues arise]
- [Database migrations, flag toggles, or backwards compatibility shims]

---

## Phase Handoff Checklist for `rpi-implement`
- [ ] Phase 1 completed & verified
- [ ] Phase 2 completed & verified
- [ ] Phase 3 completed & verified
- [ ] All automated tests pass without regressions
```

---

## Step 5: Present Plan Summary & Transition
1. Present a concise summary of the written plan in the chat, highlighting the total number of phases, key files touched, and the path to the plan file in `./thoughts/plans/`.
2. Remind the user:
   > *"The implementation plan has been written to `./thoughts/plans/YYYY-MM-DD-HHmm-<topic>.md`. Before starting the implementation phase, it is recommended to run `/clear` or `/new` to start with a fresh context window, then invoke `/implement_plan` (or the `rpi-implement` skill)."*