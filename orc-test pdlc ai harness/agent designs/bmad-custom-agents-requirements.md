# BMAD Custom Agent Development Requirements

## Overview

This document outlines the initial requirements for researching and developing prototype custom agents using the BMAD Agent Builder. These agents are intended to run within a BMAD workflow. Junior team members should use this as a guide when building agents.

---

## Agent Inventory

### Language & Tech Stack Agnostic Agents

These agents are not tied to a specific programming language or tech stack, but **will likely need tech stack context** as input, since architectural decisions and ADRs can be influenced by the tech stack in use.

Each of these agents requires a paired **adversarial reviewer agent** that critically evaluates the quality of the primary agent's output before the stage gate is passed. This follows the `stage.adversarial` pattern used in the orc-test SDLC Pipeline and is how the workflow enforces output quality for these stages.

| Agent | Description | Adversarial Reviewer Agent |
|---|---|---|
| **BA Agent** | Business Analyst agent | **BA Reviewer Agent** |
| **Architect Agent** | Software architecture agent | **Architect Reviewer Agent** |
| **ADR Agent** | Architecture Decision Record agent | **ADR Reviewer Agent** |
| **NFR Agent** | Non-Functional Requirements agent | **NFR Reviewer Agent** |

---

### Tech Stack–Specific Agent Groups (`^`)

These agents are grouped by tech stack. Each group consists of three agents: a **Developer agent**, a **Unit Test agent**, and an **Integration Test agent**. The Developer and Unit Test agents are the **highest current priority**.

> **Key design requirement:** The unit testing framework and integration test framework should each be a **runtime parameter** (not hardcoded), so the appropriate framework can be selected at execution time (e.g., JUnit vs. TestNG for unit tests, REST Assured vs. Spring Boot Test for integration tests in Java).

**Quality assurance approach:** Developer and tester agents do **not** use adversarial reviewer agents. Instead, they are responsible for checking their own work using the following:

- **Facts, policies, and constitutions** — agent-level rules and constraints that define what correct, complete, and compliant output looks like for that agent's role
- **Code coverage tool output** — coverage reports must confirm 100% path coverage before the agent considers its work done
- **Linting and code style tool output** — linter and code style checker results must be clean before handoff
- **Design guidelines** — output must conform to the architectural and design guidelines established in the `plan` stage artifacts

A **PR Review Agent** will run at the end of the `code-quality` stage, after all self-checking by developer and tester agents is complete. This agent is **deferred for now** and will be scoped and built in a future iteration.

#### Java Tech Stack
- Java Developer Agent
- Java Unit Test Agent *(unit test framework = runtime parameter)*
- Java Integration Test Agent

#### React Tech Stack
- React Developer Agent
- React Unit Test Agent *(unit test framework = runtime parameter)*
- React Integration Test Agent

#### Angular Tech Stack
- Angular Developer Agent
- Angular Unit Test Agent *(unit test framework = runtime parameter)*
- Angular Integration Test Agent

---

### Language-Agnostic Framework Agents (`*`)

These agents target frameworks that support multiple programming languages (e.g., Java, C#, TypeScript, Python, JavaScript). The **target language must be a runtime parameter** so it can be determined at execution time.

| Agent | Supported Languages (examples) |
|---|---|
| **Cucumber Agent** | Java, C#, TypeScript, Python, JavaScript |
| **Playwright Agent** | Java, C#, TypeScript, Python, JavaScript |

---

## Workflow Integration

All agents will be built as **BMAD agents** intended to run inside the **orc-test SDLC Pipeline** (`orc-test-sdlc`) workflow. The pipeline is defined in `orc-test.workflow/v1` and consists of the following stages in order:

```
feature → specify → plan → stories → tasks → development → code-quality → pr-review
```

Each stage uses an adversarial confidence gate (default threshold: 80) before passing to the next. The `code-quality` stage can loop back to `development` if convergence gaps are found.

**Artifact persistence rules:**

- **Pre-development stages** (`feature`, `specify`, `plan`, `stories`, `tasks`) — agents in these stages must persist their output artifacts to Jira via the **Jira Avro MCP**, attaching them to the appropriate work item (epic, feature, story, or task) depending on the stage and artifact type.
- **Development and post-development stages** (`development`, `code-quality`, `pr-review`) — code changes and repository changes produced by agents in these stages must be committed to GitHub on a branch scoped to the specific work item being implemented. Decision rationale and traceability records for these agents must also be persisted to Jira via the **Jira Avro MCP**, linked to the corresponding work item.

---

### Recommended Agent Placement

#### `specify` stage — BA Agent, NFR Agent, BA Reviewer Agent, NFR Reviewer Agent
The **BA Agent** is a natural fit as a candidate generator in the `specify` stage, which produces `spec.md`. It can serve alongside or replace `skill:speckit-specify` and `skill:bmad-create-prd`. The **NFR Agent** should also run here, either as a `post_generate` step or embedded in the BA Agent's output, since non-functional requirements belong in the spec before architecture begins.

Both the BA Agent and NFR Agent require paired **adversarial reviewer agents**. The **BA Reviewer Agent** critically evaluates the spec for completeness, clarity, and alignment with the feature brief. The **NFR Reviewer Agent** evaluates the non-functional requirements for coverage, measurability, and architectural feasibility. These reviewer agents emit a confidence score that must meet the stage threshold before the gate passes, consistent with the `stage.adversarial` pattern.

**Inputs available at this stage:**
- `.orc-test/outputs/planning-artifacts/feature-{feature}.md`

**Outputs expected:**
- `spec.md` (primary artifact consumed by all downstream stages)
- NFR section within spec, or a standalone `nfr.md` artifact

**Persistence:** Output artifacts and reviewer findings must be persisted to the corresponding Jira feature/epic via the **Jira Avro MCP**.

---

#### `plan` stage — Architect Agent, ADR Agent, Architect Reviewer Agent, ADR Reviewer Agent
The **Architect Agent** maps directly to the `plan` stage, which is responsible for `architecture.md`, `data-model.md`, `contracts/`, and `research.md`. It can serve as a candidate generator alongside or replacing `skill:speckit-plan`, `agent:agent-architecture-orc-test`, and `skill:bmad-create-architecture`.

The **ADR Agent** should run as a `post_generate` step within `plan`, producing ADR documents after the architecture draft is complete. ADRs depend on both the spec and the architecture, and since tech stack context influences architectural decisions, the Architect and ADR agents will need tech stack passed as a runtime input.

Both the Architect Agent and ADR Agent require paired **adversarial reviewer agents**. The **Architect Reviewer Agent** evaluates the architecture for soundness, completeness, consistency with the spec, and appropriateness for the tech stack. The **ADR Reviewer Agent** evaluates each decision record for clarity of context, justification of the decision, and completeness of consequences. Both reviewer agents emit a confidence score that must meet the stage threshold before the gate passes, consistent with the `stage.adversarial` pattern.

**Inputs available at this stage:**
- `.orc-test/outputs/specs/{slug}/spec.md`

**Outputs expected:**
- `architecture.md`
- `.orc-test/outputs/architecture/{slug}/adrs/` (new output — ADR Agent artifacts)

**Persistence:** Output artifacts, ADR documents, and reviewer findings must be persisted to the corresponding Jira feature/epic via the **Jira Avro MCP**.

---

#### `development` stage — Tech Stack Developer Agents
The tech stack–specific **Developer Agents** (Java, React, Angular) are worker candidates in the `development` stage, which uses a `stage.worker-pool` that fans out over `jira.stories`. Each developer agent would be selected based on the tech stack parameter for the story being implemented.

**Inputs available at this stage:**
- `.orc-test/outputs/tasks/{slug}/tasks.md`
- `.orc-test/outputs/architecture/{slug}/architecture.md`
- `.orc-test/outputs/specs/{slug}/spec.md`
- `jira.stories`

**Outputs expected (per developer agent):**
- `.orc-test/outputs/dev/{slug}/dev-log.md` (execution status + context for downstream test agents)
- The dev-log must include: tech stack used, modules/classes created, test entry points, and any context needed by unit and integration test agents

**Persistence:** All code changes and repository changes must be committed to **GitHub on a branch scoped to the specific work item** (story or task) being implemented. Decision rationale and traceability records must also be persisted to the corresponding Jira story/task via the **Jira Avro MCP**.

**Self-check requirement:** Developer agents do not have an adversarial reviewer. Instead, before emitting the dev-log, each developer agent must check its own work against:
- **Facts, policies, and constitutions** defined for the agent's role and tech stack
- **Linter and code style tool output** — all linting and style violations must be resolved (tools are runtime parameters)
- **Code coverage tool output** — coverage must meet the 100% path coverage target before handoff (tool is a runtime parameter)
- **Design guidelines** from `architecture.md`, `data-model.md`, and `contracts/`
- **Compile and build success** — code must compile and build cleanly

The agent must not emit its dev-log until all of the above checks pass.

---

#### `code-quality` stage — Unit Test Agents, Integration Test Agents, Cucumber Agent, Playwright Agent

The `code-quality` stage uses a `stage.check-pool`. The existing built-in checks (`lint`, `typecheck`, `test`, `build`, `security`) remain in place. The custom test agents are added as additional named checks in the pool and are executed in two ordered tiers: unit/integration tests first, then E2E tests (Cucumber, Playwright) which depend on their outputs.

##### Check Tier 1 — Unit Testing (`unit-test`)

The Unit Test agents verify the application at the **per-method level**. Every class, interface, and component produced by the developer agents must be tested in isolation to confirm they work correctly individually and in combination with their direct dependencies.

Unit test agents must validate all of the following:

- **Interface and class contracts** — all public methods behave according to their signatures and documented contracts
- **Component integration** — interfaces, classes, and components work together correctly and do not fail when wired together
- **Dependency management** — all dependencies (injected or resolved) are managed correctly and do not produce errors under normal and error conditions
- **Happy path coverage** — all expected, successful execution paths are exercised
- **Unhappy path coverage** — all failure, edge-case, and error paths are exercised
- **Business rules and requirements** — tests assert actual business logic and requirements from `spec.md`, not just structural correctness
- **Coverage target** — **100% path coverage** is required; all branches, conditions, and code paths must be covered

The unit test framework is a **runtime parameter**. The linter, code coverage checker, and coverage reporter are also **runtime parameters** so they can match the project's toolchain. Unit test agents must also confirm the code compiles and builds successfully as part of their check.

> **Note:** Developer agents must self-check against these same criteria before emitting their dev-log. The unit test agent is not an adversarial reviewer — it is a formal verification check that confirms the developer agent's self-check was accurate and complete. Both developer and unit test agents enforce quality through self-checking using facts/policies/constitutions, tool outputs, and design guidelines.

##### Check Tier 1 — Integration Testing (`integration-test`)

The Integration Test agents verify the application from a **higher-level, system view**, testing end-to-end happy path and unhappy path scenarios based on the system design in `architecture.md` and the business rules in `spec.md`.

Integration tests must cover:

- **E2E happy path** — a complete, successful flow through the system for each key scenario, given a defined input and expected output
- **E2E unhappy path** — complete failure and edge-case flows through the system, given defined inputs that should produce error or boundary outputs
- **System design conformance** — behavior matches the contracts, data models, and component interactions defined in the architecture
- **Business rule conformance** — outcomes match the business rules and requirements defined in the spec

Integration tests are not at the method level — they treat the system (or a significant slice of it) as a black box, driving it from an external boundary and asserting on observable outputs. The integration test framework is a **runtime parameter**.

> **Note:** Integration test agents also do not have adversarial reviewers. Like unit test agents, they verify their own work using facts/policies/constitutions, tool outputs, and design guidelines before emitting their output artifacts.

##### Check Tier 2 — BDD & E2E Testing (`bdd-cucumber`, `e2e-playwright`)

The Cucumber and Playwright agents run **after** Tier 1 completes and consume the unit and integration test agents' output artifacts as inputs. They test the application at the same high-level, E2E perspective as the integration tests — happy path and unhappy path — but through the lens of BDD feature scenarios (Cucumber) and browser/UI automation (Playwright).

- **Cucumber Agent** — drives BDD scenarios written in Gherkin against the running application, asserting happy and unhappy path outcomes per feature and business rule
- **Playwright Agent** — drives browser-level or API-level E2E flows, asserting correct behavior for a given input/expected output as defined by the system design and business rules

Both agents treat the application as a black box and do not test at the method level. The target language is a **runtime parameter** for both.

| Check ID | Agent | Tier | Key Runtime Parameters |
|---|---|---|---|
| `unit-test` | Tech stack Unit Test Agent | 1 | Unit test framework, linter, coverage tool, coverage reporter |
| `integration-test` | Tech stack Integration Test Agent | 1 | Integration test framework |
| `bdd-cucumber` | Cucumber Agent | 2 (after Tier 1) | Target language |
| `e2e-playwright` | Playwright Agent | 2 (after Tier 1) | Target language |
| `pr-review` | PR Review Agent | 3 — deferred | *(TBD — scoped in a future iteration)* |

**Inputs available at this stage:**
- `.orc-test/outputs/dev/{slug}/` (includes dev-log from developer agents)
- `.orc-test/outputs/specs/{slug}/spec.md`
- `.orc-test/outputs/architecture/{slug}/architecture.md`
- `.orc-test/outputs/tasks/{slug}/tasks.md`

**Outputs expected:**
- Unit Test Agent: `.orc-test/outputs/quality/{slug}/unit-test-report.json`
- Integration Test Agent: `.orc-test/outputs/quality/{slug}/integration-test-report.json`
- Cucumber Agent: `.orc-test/outputs/quality/{slug}/cucumber-report.json`
- Playwright Agent: `.orc-test/outputs/quality/{slug}/playwright-report.json`
- All test outputs feed into the existing `.orc-test/outputs/quality/{slug}/report.md`

**Persistence:** Test code, configuration changes, and any repository changes produced by agents in this stage must be committed to **GitHub on the same branch as the work item** they correspond to. Decision rationale and traceability records for all agents in this stage must be persisted to the corresponding Jira story/task via the **Jira Avro MCP**.

---

### Full Pipeline with Custom Agent Placements

| Stage | Stage Type | Custom Agents Placed Here |
|---|---|---|
| `feature` | adversarial | *(none — human intake)* |
| `specify` | adversarial | BA Agent, NFR Agent, BA Reviewer Agent, NFR Reviewer Agent |
| `plan` | adversarial | Architect Agent, ADR Agent, Architect Reviewer Agent, ADR Reviewer Agent |
| `stories` | adversarial | *(none — existing story writer)* |
| `tasks` | adversarial | *(none — existing tasks orc-test)* |
| `development` | worker-pool | Java / React / Angular Developer Agents |
| `code-quality` | check-pool | Unit Test Agents, Integration Test Agents, Cucumber Agent, Playwright Agent |
| `pr-review` | single-review | PR Review Agent *(deferred — runs after all developer and tester agent self-checks are complete)* |

---

## Artifact & Schema Design Notes

All agents use **JSON** for schema definitions and output artifacts. Schemas must be designed with the upstream and downstream stage context in mind, as described in the placement section above.

### BA Agent & NFR Agent (→ `specify`)
Input schema should accept: `feature.md` content, tech stack hint (optional at this stage). Output schema must be compatible with the `specify` stage's expected `spec.md` format.

**BA Reviewer Agent** — Input schema accepts the BA Agent's `spec.md` output and the original `feature.md`. Output schema must emit a confidence score and a findings artifact noting any gaps in completeness, clarity, or alignment with the feature brief.

**NFR Reviewer Agent** — Input schema accepts the NFR Agent's output (standalone `nfr.md` or NFR section of `spec.md`) and the `feature.md`. Output schema must emit a confidence score and a findings artifact noting any NFRs that are missing, unmeasurable, or architecturally infeasible.

**Persistence requirement:** All output artifacts and findings from the BA, NFR, and their reviewer agents must be persisted to the corresponding Jira feature or epic via the **Jira Avro MCP**. The schema must include a `jira_target` field identifying the work item to attach to.

### Architect Agent & ADR Agent (→ `plan`)
Input schema must accept: `spec.md`. Both agents will need tech stack as a runtime input since it influences architectural patterns and ADR content. ADR Agent output should follow a standard ADR format (title, status, context, decision, consequences) serialized to JSON and written to the `adrs/` output path.

**Architect Reviewer Agent** — Input schema accepts `architecture.md`, `data-model.md`, `contracts/`, and `spec.md`. Output schema must emit a confidence score and a findings artifact covering architectural soundness, spec alignment, tech stack appropriateness, and any gaps or risks identified.

**ADR Reviewer Agent** — Input schema accepts the `adrs/` output and `architecture.md`. Output schema must emit a confidence score and a findings artifact assessing each ADR for clarity of context, strength of decision justification, and completeness of stated consequences.

**Persistence requirement:** All architecture artifacts, ADR documents, and reviewer findings must be persisted to the corresponding Jira feature or epic via the **Jira Avro MCP**. The schema must include a `jira_target` field identifying the work item to attach to.

### Developer Agents (→ `development`)
Developer agents check their own work — there is no adversarial reviewer. The self-check uses facts/policies/constitutions, linter/code style tool output, code coverage tool output, and design guidelines. Output artifact (dev-log) must include:
- Execution status (including compile and build result)
- Tech stack and framework versions used
- List of modules, classes, components, and entry points created
- Test hooks or context required by the Unit Test and Integration Test agents downstream
- Linter and code style tool results (tools are runtime parameters; must be clean)
- Code coverage report (tool is a runtime parameter; must confirm 100% path coverage)
- Self-check result against facts/policies/constitutions and design guidelines
- GitHub branch reference for the work item commit

**Persistence requirement:** All code and repository changes must be committed to a **GitHub branch scoped to the work item**. Decision rationale and traceability records must be persisted to the corresponding Jira story or task via the **Jira Avro MCP**. The schema must include a `github_branch` field and a `jira_target` field.

### Unit Test Agents (→ `code-quality`, Tier 1)
Unit test agents check their own work — there is no adversarial reviewer. Self-checking uses facts/policies/constitutions, linter/code style tool output, code coverage tool output, and design guidelines. Input schema must accept the developer agent's dev-log plus runtime parameters for the unit test framework, linter, coverage checker, and coverage reporter. Output artifacts must include:
- Test execution status and compile/build result
- Pass/fail counts per class, interface, and component
- Coverage report confirming 100% path coverage (branches, conditions, all paths)
- Happy path and unhappy path scenario results
- Business rule and requirement assertion results (traced to `spec.md`)
- Any context needed by the Cucumber and Playwright agents (e.g., test suite names, module paths, entry points)
- GitHub branch reference for the work item commit

**Persistence requirement:** Test code and any repository changes must be committed to the **same GitHub branch as the work item**. Decision rationale and traceability records must be persisted to the corresponding Jira story or task via the **Jira Avro MCP**. The schema must include a `github_branch` field and a `jira_target` field.

### Integration Test Agents (→ `code-quality`, Tier 1)
Integration test agents check their own work — there is no adversarial reviewer. Self-checking uses facts/policies/constitutions, tool outputs, and design guidelines. Input schema must accept the developer agent's dev-log plus a runtime parameter for the integration test framework. Output artifacts must include:
- Test execution status
- E2E happy path scenario results (input → expected output assertions)
- E2E unhappy path scenario results (error/boundary input → expected error output assertions)
- System design conformance result (traced to `architecture.md`)
- Business rule conformance result (traced to `spec.md`)
- Context needed by the Cucumber and Playwright agents
- GitHub branch reference for the work item commit

**Persistence requirement:** Test code and any repository changes must be committed to the **same GitHub branch as the work item**. Decision rationale and traceability records must be persisted to the corresponding Jira story or task via the **Jira Avro MCP**. The schema must include a `github_branch` field and a `jira_target` field.

### Cucumber Agent & Playwright Agent (→ `code-quality`, Tier 2)
Cucumber and Playwright agents check their own work — there is no adversarial reviewer. Self-checking uses facts/policies/constitutions, tool outputs, and design guidelines. Input schema must accept output artifacts from both the Unit Test and Integration Test agents (Tier 1 must complete first). The target language is a runtime parameter and must be declared in the input schema. Both agents treat the application as a black box and must cover:
- E2E happy path scenarios per feature/business rule (given a defined input and expected output)
- E2E unhappy path scenarios (given inputs that should produce error or boundary outputs)
- Assertions are based on system design (`architecture.md`) and business rules (`spec.md`), not implementation internals

**Persistence requirement:** Test code and any repository changes must be committed to the **same GitHub branch as the work item**. Decision rationale and traceability records must be persisted to the corresponding Jira story or task via the **Jira Avro MCP**. The schema must include a `github_branch` field and a `jira_target` field.

### PR Review Agent (→ `pr-review`, Tier 3 — Deferred)
The PR Review Agent runs at the end of the `code-quality` stage after all developer and tester agent self-checks are complete. It is **deferred and will be scoped in a future iteration**. At a high level, this agent will perform a final review of the full changeset — code, test results, coverage reports, and linting output — before the pipeline transitions to the `pr-review` stage. Input and output schema details will be defined when this agent is scoped.

---

## Summary of Runtime Parameters

| Agent | Runtime Parameter | Example Values |
|---|---|---|
| Unit Test agents (`^`) | Unit test framework | JUnit, TestNG, Jest, Jasmine, Mocha, Karma |
| Unit Test agents (`^`) | Linter | Checkstyle, SpotBugs, ESLint, TSLint |
| Unit Test agents (`^`) | Code coverage checker | JaCoCo, Istanbul/nyc, Vitest coverage |
| Unit Test agents (`^`) | Coverage reporter | JaCoCo HTML, lcov, Cobertura |
| Integration Test agents (`^`) | Integration test framework | REST Assured, Spring Boot Test, Supertest, HttpClient |
| Developer agents (`^`) | Linter | Checkstyle, SpotBugs, ESLint, TSLint |
| Developer agents (`^`) | Code coverage checker | JaCoCo, Istanbul/nyc, Vitest coverage |
| Developer agents (`^`) | Coverage reporter | JaCoCo HTML, lcov, Cobertura |
| Cucumber Agent (`*`) | Target language | Java, C#, TypeScript, Python, JavaScript |
| Playwright Agent (`*`) | Target language | Java, C#, TypeScript, Python, JavaScript |
| Architect Agent | Tech stack | Java/Spring, React, Angular, etc. |
| ADR Agent | Tech stack | Java/Spring, React, Angular, etc. |

---

## Open Items

- [x] ~~Provide the default BMAD workflow for agent placement analysis~~ *(orc-test SDLC Pipeline provided)*
- [ ] Confirm whether the `code-quality` check-pool supports ordered/dependent checks (needed for Cucumber/Playwright after unit/integration tests)
- [ ] Define JSON schema templates for input/output artifacts per agent
- [ ] Confirm unit and integration test framework options per tech stack group
- [ ] Finalize priority order for tech stack groups (Java, React, Angular)
- [ ] Decide whether NFR Agent runs as a standalone `post_generate` step in `specify` or is embedded within the BA Agent
- [ ] Decide whether ADR Agent is a standalone `post_generate` step in `plan` or a capability of the Architect Agent
- [ ] Confirm new output paths (e.g., `adrs/`, per-agent test reports) are compatible with the orc-test SDLC artifact conventions
- [ ] Define the Jira Avro MCP schema for attaching artifacts to epics, features, stories, and tasks — confirm field names and attachment format
- [ ] Define the GitHub branch naming convention for work item branches used by developer and tester agents
- [ ] Confirm which Jira work item types (epic, feature, story, task) map to which pipeline stages for the `jira_target` field

---

## Final Notes

### Iterative Development Approach

These agents will be built and refined through **multiple iterations using Claude**. Throughout the development process, Claude will also be used as a feedback and review mechanism — agents, schemas, and workflow configurations can be submitted to Claude for suggestions on improving agent quality, responsibilities, input/output design, and overall workflow cohesion.

### Expect Change

This document reflects the current state of requirements and should be treated as a **living document**. The following are expected to evolve as we iterate and refine:

- Input and output artifact schemas (JSON)
- Agent responsibilities and scope boundaries
- Runtime input parameters
- Check ordering and dependencies within the `code-quality` stage
- Artifact paths and naming conventions

Changes should be reviewed in the context of the full pipeline — since agents are designed around the artifacts produced by the stage before them and consumed by the stage after them, a change to one agent's schema or responsibilities may have upstream or downstream impacts that need to be assessed.

### Testability

Agents must be designed to be **testable in isolation or as a group within their respective stage**. This is essential for validating agent behavior, debugging issues, and iterating quickly without needing to run the full pipeline.

To support this:

- Every agent's input schema must be self-contained enough that it can be exercised independently by supplying a mock input document
- Input schema instances can be **mocked with sample scenario values** — real pipeline execution is not required to test an agent
- Agents within the same stage (e.g., the Tier 1 check-pool agents, or the developer + unit test + integration test group) should also be testable together as a group using mocked inputs and outputs passed between them
- Mock input examples should cover at least one happy path scenario and one unhappy path scenario per agent, using realistic but synthetic data derived from the agent's schema definition
- Mock artifacts should be version-controlled alongside the agent definitions so they can be reused across iterations and shared with Claude for feedback and review

This testability requirement applies during development and across all future iterations. When schemas or responsibilities change, the mock inputs should be updated to reflect the new contract before the agent is considered ready for pipeline integration.

### Auditability and Traceability

Auditability and traceability must be considered in the design of every agent. Agents should produce a record of the decisions they make and the rationale behind those decisions as part of their output artifacts. This is distinct from execution status — it is a human-readable (and machine-parseable) account of *why* the agent produced the output it did, not just *what* it produced.

Considerations for agent designers:

- Each agent's output schema should include a dedicated section for **decision records** — a log of significant choices made during execution (e.g., which architectural pattern was selected and why, which test cases were generated to cover a specific business rule, which linting violations were flagged and how they were resolved)
- Decision records should reference the inputs that drove the decision (e.g., a specific requirement in `spec.md`, a constraint in `architecture.md`, a rule in the agent's constitution) so the rationale is traceable back to its source
- Where an agent evaluates multiple options and selects one, the alternatives considered and the reason for rejection should also be recorded
- Audit artifacts should be versioned and stored alongside the primary output artifacts in the `.orc-test/outputs/` path structure so they are available for review at any stage gate, by human reviewers, and by downstream agents

This becomes particularly important when iterating — if an agent's output is questioned or a bug is found downstream, the decision record is what allows the team (and Claude) to understand what the agent was reasoning from and where the logic may need to be corrected.

### Security

All agents must be designed with a **default posture of least privilege and least access**. An agent should be granted only the permissions, data access, and responsibilities it strictly requires to perform its defined role — nothing more. Access, privilege, and responsibility are expanded only as explicitly needed and justified.

Considerations for agent designers:

- **Data access** — agents should only receive the input artifacts they need for their specific task; they should not have broad access to the full artifact store unless that access is required and documented
- **Tool and MCP access** — agents should only be granted access to the tools and MCP integrations (e.g., `mcp:jira`, `mcp:vcs`) that their role requires; access to write or transition resources should be more tightly scoped than read access
- **Scope of responsibility** — agent constitutions and policies should be written to constrain the agent to its defined role and prevent scope creep; an agent should not take actions or make decisions outside its stated responsibility
- **Secrets and credentials** — agents must not have access to secrets, credentials, or sensitive configuration beyond what is required; secrets should never appear in output artifacts or decision records
- **Privilege escalation** — if an agent determines it needs access or capabilities beyond what it was granted, it should surface this as a finding or flag rather than attempting to acquire additional access autonomously

Security posture should be reviewed at each iteration alongside schema and responsibility changes. When expanding an agent's access or privileges, the justification should be documented.

### Guiding Goal

The overarching goal for all agents and workflows being built is **determinism** — given the same inputs, agents should consistently produce the same quality of output. Schema design, clearly scoped agent responsibilities, explicit runtime parameters, and well-defined input/output contracts are all in service of this goal. Every iteration should be evaluated against how well it moves the system toward more predictable, repeatable, and verifiable results.
