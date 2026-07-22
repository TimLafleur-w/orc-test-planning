# orc-test Planning Repository

Design and planning artifacts for the **orc-test PDLC AI Harness** — an agentic SDLC pipeline that uses Claude Code (BMAD framework) to automate the full software development lifecycle from Jira epics through PR review.

---

## Where we're at 7-21-2026
Currently we have a first iteration of a design [doc](<target application/BrokerEdge_Design_Document.md>) for the target application and it's pretty exhaustive. This document was created with many iterations with claude. The epics were run through the epic-critic adversarial reviewer and the findings were addressed to where the epics have >= 90 confidence score. The epics can be found in section 11. of the document. The document links to the actual json instances of the epics found in the [epics folder](<target application/jira epics/>). We need to create a first iteration of our bare bones orc-test cli bmad workflow runner so we can test our default workflow against these epics and work on refining and ironing out issues in the agents, skills, and workflows.

## Repository Structure

```
orc-test-planning/
├── orc-test pdlc ai harness/          # Pipeline design, agent specs, and schemas
└── target application/       # BrokerEdge — the sample app the pipeline builds
```

### `orc-test pdlc ai harness/`
Everything under this folder is design and planning artifacts for the orc-test cli application. 

| Path | Description |
|---|---|
| `pdlc agent pipeline design.md` | 8-stage pipeline overview: Feature → Spec → Plan → Stories → Tasks → Dev → Quality → PR Review |
| `run-config.example.json` | Per-contributor runtime config (GitHub repo, Jira project). Copy to `run-config.json`. |
| `.env.example` | Secret template (`GITHUB_TOKEN`, `JIRA_API_TOKEN`). Copy to `.env`, never commit. |
| `agent designs/bmad-custom-agents-requirements.md` | Requirements for each custom BMAD agent (language-agnostic, Java, React, Angular, Cucumber, Playwright) |
| `jira schemas/` | JSON Schema (`orc-test.epic/v1`) and example instances for epic, feature, story, and task work items |
| `agent artifact schemas/` | Output schemas for each pipeline stage agent *(placeholders — in progress)* |

### `target application/`

| Path | Description |
|---|---|
| `BrokerEdge_Design_Document.md` | ~3,500-line comprehensive spec for BrokerEdge, the mock brokerage platform |
| `jira epics/` | 12 fully-authored epic JSON files (BE-1 through BE-12) conforming to `orc-test.epic/v1` |

---

## The Pipeline - subject to change

The PDLC AI Harness drives agents through these stages, consuming Jira epics as input and committing artifacts back to GitHub and Jira at each stage:

1. **Specify** — BA agent produces a structured feature spec
2. **Plan** — Architect agent produces an architecture decision + ADR
3. **Stories** — Story agent decomposes the spec into user stories
4. **Tasks** — Task agent breaks stories into developer tasks
5. **Developer** — Tech-stack-specific agent generates application code
6. **Code Quality** — Quality agent reviews and patches generated code
7. **PR Review** — PR reviewer agent scores the final diff

Each stage has a paired adversarial reviewer agent for language-agnostic stages.

Future goal:\
 We will later see and find that we will likely need agents in the pipeline that assist with the steps of creating the epics/features, the quality and details that go into creating the epics/features will help avoid problems that would lead to lower adversarial reviewer scores. So in the future there will be agents that go before specify agent that help walkthrough and ask questions to get all the details that would go into a SAD or design doc and eventually the epics/features. 

---

## Target Application: BrokerEdge

BrokerEdge is a fictional brokerage account management platform used to exercise every pipeline stage across a realistic full-stack codebase.

**Backend:** Java 21, Spring Boot 3.x, Spring Security (JWT), Spring Data JPA, Flyway, PostgreSQL 16, Apache Kafka, Spring Batch, Resilience4j, Testcontainers

**Frontend:** React 18 + TypeScript, Zustand, TanStack Query v5, shadcn/ui, Tailwind CSS, Playwright, Vitest

**Architecture patterns:** Clean Architecture, DDD, Transactional Outbox, CQRS, Event Sourcing (audit log), Circuit Breaker

### Epics (BE-1 – BE-12)

| ID | Title |
|---|---|
| BE-1 | Identity & Access Management — Registration, Login & JWT Sessions |
| BE-2 | Account Management — Open Account, Funding & Account Details |
| BE-3 | Order Management — Place, Clear, Cancel & Track Orders |
| BE-4 | Portfolio & Positions — Summary, Holdings, P&L & Performance |
| BE-5 | Market Data (Mock) — Security Catalogue, Quote Feed & Search |
| BE-6 | Batch Processing — Settlement, Valuation, Statements & Reconciliation |
| BE-7 | Event Bus & Audit Infrastructure — Outbox, Kafka Topics & Immutable Audit Trail |
| BE-8 | UI Portal Core — Auth Screens, Dashboard, Account UI & Navigation |
| BE-9 | UI Portal Trading — Order Entry, Order Book & Security Search |
| BE-10 | UI Portal Portfolio — Portfolio View, Transaction History & Statements |
| BE-11 | DevOps & CI/CD Foundation — Repo, Build, Containerisation & Local Orchestration |
| BE-12 | Testing Infrastructure — Testcontainers, Contract Tests, Playwright & Coverage |

---

## Local Setup

1. Copy `pdlc ai harness/run-config.example.json` → `pdlc ai harness/run-config.json` and fill in your GitHub and Jira settings.
2. Copy `pdlc ai harness/.env.example` → `pdlc ai harness/.env` and add your tokens.
3. Open this repo in Claude Code with the BMAD extension active.

---

## Status

| Area | State |
|---|---|
| Epic schema (`orc-test.epic/v1`) | Complete |
| BrokerEdge epics (BE-1 – BE-12) | Complete |
| BrokerEdge design document | Complete |
| BMAD agent requirements | Complete |
| Feature / story / task schemas | In progress (placeholders) |
| Agent artifact output schemas | In progress (placeholders) |
| Pipeline design document | Draft |
| Application code | Not started — generated by pipeline |
