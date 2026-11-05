---
name: rpi-research
description: Orchestrates comprehensive, read-only codebase research using the 3-pass RPI (Research-Plan-Implement) methodology. Delegates to specialized sub-agents or tools to locate code, analyze mechanics, and discover existing patterns, then writes a structured technical map to ./thoughts/research/.
tags: [rpi, research, codebase, architecture]
tools:
  - run_subagent
  - get_repo_metadata
  - find_files
  - search_code
  - read_file
  - write_file
---

# RPI Research Orchestrator Skill

You are tasked with conducting comprehensive, objective research across the local Git codebase to answer user inquiries and produce an authoritative technical map at `./thoughts/research/YYYY-MM-DD-HHmm-<topic>.md`.

---

## CRITICAL MANDATE: DOCUMENTARIAN, NOT CRITIC

Your **sole responsibility** is to document and explain the codebase exactly as it exists today:
- **DO NOT** suggest improvements, refactorings, or architectural changes unless explicitly prompted by the user.
- **DO NOT** perform root-cause analysis or critique code quality.
- **DO NOT** propose future enhancements or implementation steps (those belong in the upcoming `rpi-plan` phase).
- **DO NOT** modify any application code under any circumstance.
- **ONLY** describe what exists, where it exists (`file:line`), how it functions, and how components integrate.

---

## Execution Workflow

Execute these steps in strict sequence:

### Step 1: Ingest Context & Identify Topic
1. **Extract Topic**: Identify the specific feature, subsystem, or technical question from the user prompt.
2. **Read Explicitly Mentioned Files**: If the user referenced specific files, configurations, or tickets, call `read_file` on them first to anchor your understanding.
3. **Check Prior Research**: Check `./thoughts/research/` using `find_files` for any prior research artifacts that provide historical context.

---

### Step 2: Decompose Research into Three Passes
Deconstruct the research topic into three distinct, complementary investigation scopes:
1. **Pass A: Codebase Locator** &rarr; Finding where components, routes, tests, and entry points live.
2. **Pass B: Codebase Analyzer** &rarr; Understanding how the identified code works in detail without critique.
3. **Pass C: Pattern Finder** &rarr; Discovering established architectural and code-level patterns used in adjacent systems.

---

### Step 3: Execute the Three Passes via Sub-Agents
If the `run_subagent` tool is available in your runtime, delegate each pass to an isolated sub-agent session. If `run_subagent` is unavailable or returns an error, execute the three passes directly and sequentially using your local read-only tools (`find_files`, `search_code`, `read_file`).

#### Pass A: Codebase Locator
- **Tool Call**: 
  ```text
  run_subagent(
    skill_name="rpi-codebase-locator", 
    task_prompt="Locate all files, entry points, configuration surfaces, and unit/integration tests related to: <topic>"
  )
  ```
- **Fallback (Sequential)**:
  - Use `find_files(pattern="*<keyword>*")` to identify matching files across the repository.
  - Use `search_code(query="<keyword>")` to identify exported types, class definitions, and route declarations.
  - Group discovered files into logical subsystems: Core Logic, API/CLI Interfaces, Configuration, and Tests.

#### Pass B: Codebase Analyzer
- **Tool Call**: 
  ```text
  run_subagent(
    skill_name="rpi-codebase-analyzer", 
    task_prompt="Analyze how the following files function, tracing call graphs, state transitions, and integration points: <list_of_core_files>"
  )
  ```
- **Fallback (Sequential)**:
  - Call `read_file` on the key files located in Pass A (using `line_start` and `line_end` for large files).
  - Trace execution step-by-step from initialization or request ingress down to backend handlers.
  - Record exact line references (`path/to/file.ext:15-42`) and extract concise code snippets illustrating critical mechanics.

#### Pass C: Pattern Finder
- **Tool Call**: 
  ```text
  run_subagent(
    skill_name="rpi-pattern-finder", 
    task_prompt="Identify established repository conventions for error handling, dependency registration, configuration flags, and testing around: <subsystem>"
  )
  ```
- **Fallback (Sequential)**:
  - Examine adjacent components to determine standard repository idioms.
  - Document concrete examples of naming conventions, logging patterns, and mock fixtures with exact `path:line` citations.

---

### Step 4: Gather Repository Metadata
Call `get_repo_metadata()` to collect actual Git repository values. **Never use fabricated or dummy metadata**:
- Current ISO-8601 timestamp
- Current Git commit SHA
- Active Git branch
- Repository name

Formulate the destination artifact path:
```
thoughts/research/YYYY-MM-DD-HHmm-<kebab-case-topic>.md
```

---

### Step 5: Synthesize and Write the Research Artifact
Synthesize the findings from all three passes and write the complete research document using `write_file(path=..., content=...)`. 

Use the exact markdown structure below:

```markdown
---
date: [ISO-8601 timestamp with timezone]
git_commit: [Full commit SHA from get_repo_metadata]
branch: [Current branch name]
repository: [Repository name]
topic: "[User's Research Question / Topic]"
tags: [research, codebase, relevant-subsystems]
status: complete
---

# Research: [User's Research Question / Topic]

## Research Question
[Clear, unambiguous statement of what was investigated]

## Summary
[Executive overview describing what exists today, how the components interact, and answering the core inquiry without critiquing the code]

## Detailed Findings

### 1. [Subsystem / Component Area 1]
- **Primary Location**: `path/to/file.ext:line`
- **Responsibilities & Mechanism**: [Objective explanation of structs, classes, key functions, and execution lifecycle]
- **Upstream / Downstream Integration**: [How this layer connects to other modules]

### 2. [Subsystem / Component Area 2]
[Repeat for each relevant subsystem: Core Logic, API/CLI Layer, State/Storage, Tests]

## Architecture & Data Flow
[Provide an ASCII flow diagram or numbered sequence tracing initialization, data flow, and runtime execution]

```
[Component A] ---> (function_call) ---> [Component B] ---> [Storage / External]
```

## Code References
### Core Implementation
- `path/to/core_file.ext:10-45` - [Brief description of what this section implements]
### Integration Points
- `path/to/api_route.ext:88-112` - [Handler registration and parameter mapping]
### Tests & Fixtures
- `tests/test_feature.ext:24-60` - [Existing test coverage and assertions]

## Established Patterns & Conventions
1. **[Pattern Name]** (`path/to/example.ext:line`): [Description of how the codebase handles this concern]
2. **[Pattern Name]** (`path/to/example.ext:line`): [Description of how the codebase handles this concern]

## Open Questions & Ambiguities
1. **[Question/Area]**: [Any factual uncertainties, missing documentation, or areas requiring human clarification before planning]
```

---

### Step 6: Present Findings & Session Handoff
1. Output a concise summary of the research directly in the chat terminal.
2. State the relative path of the saved artifact (e.g., `thoughts/research/2026-11-04-2130-auth-token-refresh.md`).
3. Explicitly inform the user that research is complete, and recommend they run `/new` or `/clear` to start a clean context session before invoking `/plan` or the `rpi-plan` skill.