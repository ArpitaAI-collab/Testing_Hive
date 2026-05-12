# 🐝 Testing Hive — Multi-Agent Quality Intelligence Platform

> **"The Hive thinks. The Hive remembers. The Hive tests."**

**Testing Hive** is a multi-agent AI platform built on the **BMAD Agent Builder** framework. It transforms how QA teams approach test generation, failure analysis, and regression intelligence — replacing manual effort with a coordinated hive of specialized AI agents commanded by **Aegis**, the Lead QA Strategist.

Every artifact is traceable: every `.feature` file carries a `@Forensic-[ID]` tag, permanently linking each test to the evidence that produced it through the **Forensic Trail**.

---

## Table of Contents

1. [System Architecture](#system-architecture)
2. [The 10 Agents](#the-10-agents)
3. [Intelligence Layer](#intelligence-layer)
4. [Command Center](#command-center)
5. [Qdrant Memory Model](#qdrant-memory-model)
6. [DOM-Aware Agent](#dom-aware-agent)
7. [Configuration Model](#configuration-model)
8. [Scoring and Reporting](#scoring-and-reporting)
9. [Classification and Healing Logic](#classification-and-healing-logic)
10. [Current Capabilities](#current-capabilities)
11. [Prerequisites](#prerequisites)
12. [Installation](#installation)
13. [Configuring Connectors](#configuring-connectors)
14. [Execution Guide](#execution-guide)
15. [Project Structure](#project-structure)
16. [Client-Side Deployment](#client-side-deployment)
17. [Development](#development)
18. [Troubleshooting](#troubleshooting)

---

## System Architecture

```
┌──────────────────────────────────────────────────────────────────┐
│           AEGIS — Maya Quinn (Central Brain)                      │
│           agent-test-architect · Lead QA Strategist               │
│           Risk Profiler · Forensic Analyst                        │
│                                                                   │
│   UI:        hive_app_v3.py (Streamlit)                          │
│   Memory:    Qdrant Vector DB (forensic risk records)            │
│   Connectors: Jira · GitHub · Jenkins · LLM Router · Qdrant     │
└───────────────────────────┬──────────────────────────────────────┘
                            │ Dispatches sub-agents
                            ▼
┌──────────────────────────────────────────────────────────────────┐
│                       THE TESTING HIVE                           │
│                                                                   │
│  Unit          API             Functional      UI                │
│  Test Agent    Test Agent      Test Agent      Test Agent        │
│  Jest/TS       Supertest       Playwright E2E  Playwright        │
│  (Ethan Cole)  (Priya Nair)    (Sofia Bennett) (Lena Hart)       │
│                                                                   │
│  Regression    Coverage        Classification  Self-Healing      │
│  Agent         Analysis Agent  Agent           Agent             │
│  Qdrant-Bkd    Jenkins Report  (Nina Rao)      (Elena Park)      │
│  (Marcus Vale) (Daniel Cross)                                    │
│                                                                   │
│  DOM-Aware Sub-Agent (Aria Chen)                                 │
│  Grounds UI-backed generation in DOM structure and selectors     │
└──────────────────────────────────────────────────────────────────┘
                            │
                            ▼
┌──────────────────────────────────────────────────────────────────┐
│                    INTELLIGENCE LAYER                             │
│                                                                   │
│  LLM Router              Qdrant Forensics                        │
│  OpenAI · Anthropic      Vector DB: historical risk,             │
│  Gemini · DeepSeek       failure patterns, selector              │
│  Local (Ollama)          ancestry, forensic records              │
│                                                                   │
│  Jira Connector          GitHub Connector                        │
│  ADF → plaintext         PR diffs, post comments                 │
│  AC extraction                                                   │
└──────────────────────────────────────────────────────────────────┘
```

### Orchestration Model

The current implementation is a **BMAD-style orchestrated multi-agent QA workflow** with Aegis as the control plane.

**How it works:**

1. **Aegis** runs through the Streamlit app entrypoint and owns ticket ingestion, connector calls, risk context, and artifact dispatch.
2. **Jira ingestion** fetches the real ticket, parses Atlassian Document Format (ADF), and extracts Acceptance Criteria.
3. **Qdrant forensics** adds historical context for risk framing and regression guidance.
4. **Aegis story-based routing** recommends which generation agents should run for the active story — it does not force every category every time.
5. **DOM-aware routing** is applied separately for UI-backed stories:
   - `UI` always receives DOM support when routed and DOM context exists.
   - `FUNCTIONAL` and `REGRESSION` also receive DOM support when the story is `UI-heavy` or `Flow-oriented`.
6. **Selected generation agents** are dispatched sequentially through the LLM router: `Unit → API → Functional → UI → Regression`.
7. Each generation agent returns a Gherkin `.feature` file and an executable TypeScript `.spec.ts` file.
8. **Classification**, **Coverage**, and **Self-Healing** interpret current session artifacts and the Forensic Trail in the post-run view.

> **Important:** The current model is a single orchestrator-driven workflow, not a distributed autonomous agent mesh. Aegis routes work to sub-agents, controls handoffs between generation, human review, classification, and repair suggestion — but agents do not independently negotiate or schedule work.

### Internal Beta Service Layer

The app launches through `hive_app_v3.py`, but the backend is no longer fully monolithic. An internal-beta service layer exists under:

- `testing_hive_services/routing_service.py`
- `testing_hive_services/generation_service.py`
- `testing_hive_services/memory_service.py`
- `testing_hive_services/run_state_service.py`

Streamlit remains the active UI shell while routing, generation, memory-role mapping, and run-state persistence are delegated through reusable service modules.

---

## The 10 Agents

### Agentic Orchestration Roster

| # | Agent Name | Role | Persona |
|---|------------|------|---------|
| 1 | **Aegis** | Lead QA Strategist — Central Brain | 🧑‍✈️ Maya Quinn |
| 2 | **Unit Test Agent** | Logic Validation Agent | 🧑‍🔬 Ethan Cole |
| 3 | **API Test Agent** | API Contract Agent | 👩‍💻 Priya Nair |
| 4 | **Functional Test Agent** | Business Flow Agent | 🧑‍💼 Sofia Bennett |
| 5 | **UI Test Agent** | Experience Assurance Agent | 👩‍🎨 Lena Hart |
| 6 | **DOM-Aware Sub-Agent** | DOM-Aware UI Sub-Agent | 🧭 Aria Chen |
| 7 | **Regression Agent** | Regression Intelligence Agent | 🧑‍🏫 Marcus Vale |
| 8 | **Classification Agent** | Story Routing and Classification Agent | 👩‍⚖️ Nina Rao |
| 9 | **Coverage Analysis Agent** | Risk and Coverage Agent | 🧑‍📊 Daniel Cross |
| 10 | **Self-Healing Agent** | Recovery Agent | 👩‍🔧 Elena Park |

### Agent Detail: Purpose, Inputs, and Outputs

| Agent | Overall Purpose | Main Inputs | Tool / Memory Use | Current Outputs |
|-------|-----------------|-------------|-------------------|-----------------|
| **Aegis** | Orchestrates the full QA flow | Jira story, ACs, Qdrant history, user profile config | Jira connector, Qdrant, LLM router, Jenkins, GitHub | Risk brief, routing decision, execution orchestration |
| **Unit** | Covers deterministic validation and logic-style checks | Jira ACs, story semantics, generation profile | LLM router | Scenario artifact + executable test artifact |
| **API** | Covers service, contract, and payload behavior | Jira ACs, linked service semantics, generation profile | LLM router | Scenario artifact + executable API-oriented test artifact |
| **Functional** | Covers business flow and end-to-end acceptance behavior | Jira ACs, workflow semantics, DOM context when UI-backed | LLM router, DOM memory when routed | Scenario artifact + executable functional test artifact |
| **UI** | Covers user interactions and selector-aware browser behavior | Jira ACs, DOM context, selector policy, UI target URL | LLM router, DOM memory | UI-focused Gherkin + executable browser test artifact |
| **DOM-Aware Sub-Agent** | Grounds UI-backed generation in actual page structure | `UI_TARGET_URL`, `UI_DOM_HTML_SNAPSHOT`, `UI_ACCESSIBILITY_NOTES`, current story | LLM router, DOM Qdrant collection, run store | DOM summary, stable selectors, field constraints, Gherkin guidance, TS guidance |
| **Regression** | Expands risky changes into replayable regression coverage | Jira ACs, Qdrant history, DOM context when UI-backed | LLM router, Qdrant history, DOM memory | Regression-oriented scenario/test artifact set |
| **Coverage** | Summarizes generated and executed coverage | Generated artifacts, Jira ACs, Jenkins results | Jenkins report data, session context | Coverage summary, AC coverage, selector stability summary |
| **Classification** | Explains what kind of failure occurred | Failed Jenkins case, evidence snippet, console block, artifacts, Qdrant context | Jenkins evidence, LLM router, Qdrant | Category, confidence, root cause type, failure layer, QA triage report |
| **Self-Healing** | Proposes what to do next after classification | Classification result, evidence, DOM context, Qdrant healing history | LLM router, Qdrant healing/DOM memory, Jenkins context | Repair suggestion, approval path, patch validation path when applicable |

### Agent Badge and Skill Directory Reference

| Agent | Badge | Skill Directory | Output |
|-------|-------|-----------------|--------|
| **Aegis** 🎯 | `badge-aegis` | `skills/agent-test-architect/` | Risk Briefs, Qdrant forensic memory |
| **Unit** 🧪 | `badge-unit` | `skills/unit-test-agent/` | Gherkin `.feature` + Jest `.spec.ts` |
| **API** 🔌 | `badge-api` | `skills/api-test-agent/` | Gherkin `.feature` + Supertest `.spec.ts` |
| **Functional** 📋 | `badge-func` | `skills/functional-test-agent/` | Gherkin `.feature` + Playwright `.spec.ts` |
| **UI** 🎨 | `badge-ui` | `skills/ui-test-agent/` | Gherkin `.feature` + Playwright `.spec.ts` |
| **Regression** 🔄 | `badge-reg` | `skills/regression-test-agent/` | Gherkin `.feature` + prioritized matrix |
| **Coverage** 🔍 | `badge-cov` | `skills/coverage-analysis-agent/` | Session-derived coverage summary; Jenkins-backed gap analysis |
| **Classification** 🏷️ | `badge-class` | `skills/test-classification-agent/` | Session-aware failure categorization (SEL/DATA/NET/LOGIC/ENV/TIMING) |
| **Self-Healing** 🩹 | `badge-heal` | `skills/self-healing-agent/` | Context-aware repair suggestions; automatic patch validation for safer cases |

> **Execution note:** `Unit`, `API`, `Functional`, `UI`, and `Regression` are coverage and generation categories, not separate CI runners. Their generated artifacts compile into the same Playwright-based Jenkins execution path and must remain compatible with the active runner contract.

---

## Intelligence Layer

All intelligence scripts live under `skills/agent-test-architect/scripts/`:

| Script | Purpose | CLI Entry Point |
|--------|---------|-----------------|
| `llm_router.py` | Unified gateway for OpenAI, Anthropic, Gemini, DeepSeek, generic OpenAI-compatible endpoints, and Local (Ollama) | `python3 skills/agent-test-architect/scripts/llm_router.py` |
| `qdrant_forensics.py` | Vector DB for failure history, selector ancestry, risk records | `python3 skills/agent-test-architect/scripts/qdrant_forensics.py` |
| `jira_connector.py` | Fetch tickets, post comments, list projects. Includes ADF-to-text parser for Atlassian Document Format | `python3 skills/agent-test-architect/scripts/jira_connector.py` |
| `github_connector.py` | Fetch PR diffs, post comment artifacts | `python3 skills/agent-test-architect/scripts/github_connector.py` |
| `jenkins_orchestrator.py` | Trigger builds, poll status | `python3 skills/agent-test-architect/scripts/jenkins_orchestrator.py` |
| `mock_gherkin_gen.py` | Demo pipeline: Jira → Gherkin → TypeScript | `python3 skills/agent-test-architect/scripts/mock_gherkin_gen.py` |

---

## Command Center

`hive_app_v3.py` is the Streamlit Command Center. It brings together orchestration, human review, CI execution, post-run classification, guarded healing, and Jira feedback in a single UI.

| Tab | Purpose |
|-----|---------|
| **🏠 Home** | Platform landing page with Aegis overview, named sub-agent roster, toolchain summary, and current operating context |
| **🔎 Forensic Dashboard** | Jira ingestion, Qdrant memory lookup, 5D risk framing, forensic trail creation, and pre-generation recommendations |
| **⚡ Live Execution Feed** | Progressive generation flow, per-agent artifacts, **human-in-the-loop review gate**, LLM-guided suggestions, export, branch push, and Jenkins trigger |
| **🔬 Post-Run Insights** | Jenkins sync, failed-case selection, evidence extraction, classification, coverage summary, human-reviewed repair guidance, targeted patch validation, and Jira update preview |

### Human-in-the-Loop (HITL) Review Gate

The Live Execution Feed tab includes a full **Human-in-the-Loop review gate** between artifact generation and export. This gate gives QA engineers control over every artifact before it reaches CI.

**During the review gate, users can:**
- Edit generated scenarios and code directly
- Accept or dismiss LLM-guided review suggestions
- Approve or reject individual artifacts
- Use bulk-approval actions (scenarios-only, code-only, or the entire run)

**After approval, users can:**
- Export approved artifacts to the runner repo
- Push a generated branch
- Trigger Jenkins from the UI

This gate is intentional — it prevents unreviewed LLM output from reaching CI and gives humans meaningful control over every step in the pipeline.

---

## Qdrant Memory Model

The Qdrant design is split into five logical collections:

| Memory Role | Env Variable | Default Collection | Current Use |
|-------------|--------------|--------------------|-------------|
| **Risk** | `QDRANT_RISK_COLLECTION` | `test-risk-history` | Story-level risk snapshots; future routing and risk similarity |
| **Knowledge** | `QDRANT_KNOWLEDGE_COLLECTION` | `jira-tester-knowledge` | Jira-derived requirement knowledge documents |
| **DOM** | `QDRANT_DOM_COLLECTION` | `dom-aware-memory` | DOM-aware summaries, selector/constraint findings, UI-structure guidance |
| **Healing** | `QDRANT_HEALING_COLLECTION` | `selector-ancestry` | Selector-healing suggestions, approvals, rejections, and targeted validation events |
| **Execution** | `QDRANT_EXECUTION_COLLECTION` | `execution-memory` | Post-run Jenkins/execution summaries for instability and trend analysis |

The app reads and writes across all five memory roles. The `Qdrant Memory Health` card in the Home tab shows record counts, last-write visibility, and whether a collection is actively receiving writes.

> The broader cross-run value of risk, DOM, healing, and execution memory improves as more real runs accumulate. The most fully exercised happy-path behavior remains around Jira knowledge sync plus the tested Playwright/Jenkins flow.

---

## DOM-Aware Agent

The **DOM-Aware UI Sub-Agent (Aria Chen)** grounds UI-backed generation and repair flows in actual page structure.

### What It Reads

- `UI_TARGET_URL` — the target application URL
- `UI_DOM_HTML_SNAPSHOT` — a snapshot of the DOM
- `UI_ACCESSIBILITY_NOTES` — accessibility and interaction notes

### What It Produces

- Stable selectors
- Field constraints (`maxlength`, `pattern`, `disabled`)
- Interaction notes
- Gherkin guidance
- TypeScript guidance

### When DOM Support Is Applied

| Agent | DOM Support Applied When |
|-------|--------------------------|
| `UI` | Always, when routed and DOM context exists |
| `FUNCTIONAL` | When Aegis marks the story as `UI-heavy` or `Flow-oriented` |
| `REGRESSION` | When the routed story is UI-backed |
| `UNIT` | Never by default |
| `API` | Never by default |

### Architecture Note

DOM findings are stored in both the SQLite run store and the dedicated Qdrant DOM collection. The app rehydrates DOM context in this order:

1. Live run context
2. Latest run report from SQLite
3. Qdrant DOM memory

This makes the DOM-aware flow portable for future non-Streamlit frontends.

### SCRUM-70 DOM Example

For the DemoQA `SCRUM-70` story, the DOM-aware agent supports:
- `Text Box` email validation
- `Web Tables` age and salary validation
- `Radio Button` disabled behavior
- Overlay/obstruction stability checks

**Recommended `UI_TARGET_URL`:**
```
https://demoqa.com/elements
```

**Example `UI_DOM_HTML_SNAPSHOT`:**
```html
[PAGE] https://demoqa.com/elements

[TEXT_BOX]
<input type="email" id="userEmail" autocomplete="off" placeholder="name@example.com" class="mr-sm-2 form-control" />
<button id="submit" type="button">Submit</button>
<div id="output" class="mt-4 row"></div>

[WEB_TABLES]
<button id="addNewRecordButton" type="button">Add</button>
<div class="modal-content">
  <input id="age" type="text" pattern="\d*" minlength="1" maxlength="2" placeholder="Age" />
  <input id="salary" type="text" pattern="\d*" minlength="1" maxlength="10" placeholder="Salary" />
  <button id="submit" type="button">Submit</button>
</div>

[RADIO_BUTTON]
<input id="noRadio" type="radio" disabled />
<label for="noRadio" class="form-check-label disabled">No</label>

[OVERLAY_BEHAVIOR]
Overlay may appear as iframe/div and obstruct interaction.
Recovery strategy should prefer visibility checks, scroll into view, and obstruction-aware interaction handling.
```

---

## Configuration Model

The app uses a three-layer override model:

```
Level 1  .env defaults                (startup baseline)
Level 2  OS environment overrides     (useful for CI/CD)
Level 3  Streamlit sidebar overrides  (session-only; empty values do NOT erase .env)
```

> **Key rule:** Empty sidebar fields never override valid `.env` values. Clearing a field in the sidebar preserves the original `.env` value and protects credentials from accidental clearing.

### Core Sidebar Variables

| Section | Variables | Purpose |
|---------|-----------|---------|
| **Jira** | `JIRA_URL`, `JIRA_EMAIL`, `JIRA_API_KEY` | Requirement source, ticket fetch, Jira comment pushback |
| **GitHub** | `GITHUB_TOKEN`, `GITHUB_REPO` | Runner repo export, branch push |
| **Jenkins** | `JENKINS_URL`, `JENKINS_USER`, `JENKINS_API_TOKEN`, `JENKINS_BRANCH_PARAM` | CI trigger, branch routing, build sync |
| **Qdrant** | `QDRANT_HOST`, `QDRANT_RISK_COLLECTION`, `QDRANT_KNOWLEDGE_COLLECTION`, `QDRANT_DOM_COLLECTION`, `QDRANT_HEALING_COLLECTION`, `QDRANT_EXECUTION_COLLECTION` | Memory layers for risk, knowledge, DOM, healing, and execution |
| **LLM Provider** | `LLM_PROVIDER`, `LLM_MODEL`, `LLM_API_KEY`, `OPENAI_COMPATIBLE_BASE_URL`, `OPENAI_COMPATIBLE_API_KEY` | Generation, classification, and healing route defaults, including OpenAI-compatible endpoints |
| **Embedding Provider** | `EMBEDDING_PROVIDER`, `EMBEDDING_MODEL`, `EMBEDDING_DIMENSION` | Vectorization and semantic memory search |
| **Agent Routing** | `USE_AEGIS_ROUTING`, `ENABLED_AGENTS`, plus per-agent provider/model overrides | Aegis story-based dispatch with optional manual override for `unit`, `api`, `functional`, `ui`, `regression`, `classification`, `healing`, `coverage` |

### Story-Based Agent Routing

When `USE_AEGIS_ROUTING=true`, Aegis recommends a story-specific subset of generation agents using:
- Jira summary, description, and acceptance criteria wording
- UI vs API vs validation/workflow signals
- Risk score
- Qdrant historical matches

The UI surfaces this as a recommended agent subset (e.g. `UNIT`, `FUNCTIONAL`, `UI`) and story-type labels:

- `UI-heavy`
- `API-heavy`
- `Validation-heavy`
- `Flow-oriented`
- `Regression-required`
- `Lean coverage`

If story-based routing is turned off in the sidebar, the app falls back to the manual `Enabled Generation Agents` multiselect.

### Generation Controls

| Variable | Purpose | Current Practical Capability |
|----------|---------|------------------------------|
| `TEST_AUTOMATION_FRAMEWORK` | Target automation framework | Best supported today with `playwright` |
| `TEST_SCRIPT_LANGUAGE` | Output script language | Best supported today with `typescript` |
| `TEST_SCENARIO_FORMAT` | Scenario artifact style | `gherkin` is the primary working path |
| `TEST_CASE_STYLE` | Test flavor emphasis | Supports patterns like negative, boundary, mixed |
| `TEST_DATA_POLICY` | Data generation strategy | Typically synthetic/demo-safe data |
| `TEST_SELECTOR_POLICY` | Selector preference strategy | Accessibility-first or DOM-oriented guidance |
| `TEST_STEP_DETAIL` | Verbosity of generated steps | Controls compact vs richer scenario wording |
| `TEST_GENERATION_STRICT_MODE` | Prompt strictness and structure expectation | Tightens generation toward structured output |

### Runner Controls

| Variable | Purpose | Current Practical Capability |
|----------|---------|------------------------------|
| `RUNNER_FRAMEWORK` | CI runner framework profile | Currently aligned to Playwright runner |
| `RUNNER_LANGUAGE` | CI runner language profile | Currently aligned to TypeScript runner |
| `JENKINS_BRANCH_PARAM` | Jenkins parameter for generated branch selection | Supports flexible branch parameter naming |

> **Configurable does not mean seamless.** Only the `Playwright + TypeScript + Gherkin + Jenkins` path has been tested end to end. Other enterprise toolchains (Selenium, Calypso, structured Excel) require dedicated integration, templates, and validation before they can be treated as plug-and-play.

---

## Scoring and Reporting

### 5D Risk Profile

The Forensic Dashboard frames risk across five dimensions:

| Dimension | Description |
|-----------|-------------|
| **Critical Path Score** | How central this feature is to core flows |
| **Integration Surface** | Breadth of service or dependency contact |
| **Data Complexity** | Volume and variety of data handling |
| **Edge Case Density** | Number and complexity of boundary conditions |
| **Historical Instability** | Past failure frequency from Qdrant memory |

These values are computed per story from forensic inputs and available memory context — not hardcoded.

### Overall Risk Score

The Overall Risk Score is a heuristic roll-up influenced by:
- Number and complexity of acceptance criteria
- Presence of integration-heavy or validation-heavy language
- Historical similarity hits from Qdrant memory
- Instability signals found in prior runs or related artifacts

### Coverage Summary

| Metric | Logic |
|--------|-------|
| **Tests Observed** | Count of generated scenarios and executable tests, plus Jenkins-backed totals when synced |
| **AC Coverage** | Covered acceptance criteria / total parsed acceptance criteria |
| **Untested ACs** | Acceptance criteria not mapped to any generated scenario or test |
| **Stable Selectors** | Safer selector candidates inferred from generated UI artifacts and selector policy |
| **Jenkins totals** | Real pass/fail/skip totals from Jenkins JUnit report when available |

> Before Jenkins sync, some coverage numbers are generation-derived. After Jenkins sync, pass/fail counts are taken from Jenkins aggregation.

---

## Classification and Healing Logic

### Classification Labels

| Label | Meaning | Typical Signals |
|-------|---------|-----------------|
| `SEL` | Selector / locator drift | Locator failures, missing element patterns, selector mismatch |
| `DATA` | Data validation or expected-value mismatch | Validation wording, payload/schema/value-state mismatches |
| `NET` | Network or dependency failure | `ECONNREFUSED`, unreachable service, dependency startup issues |
| `LOGIC` | Test-design or assertion-logic mismatch | Expected vs observed behavior divergence without selector failure |
| `ENV` | Environment or framework/runtime issue | Syntax, import, framework wiring, runtime setup issues |
| `TIMING` | Synchronization / obstruction / wait-state issue | Timeout, overlay obstruction, delayed state readiness |

Classification combines:
- Rule-based pattern matching on failed case and console evidence
- Artifact-aware context from the current run
- Qdrant historical similarity count
- Optional LLM enrichment for a structured triage result

The Classification Agent produces: category, confidence, root cause type, failure layer, recommended action, recommended fix type, healable/not healable, and a full QA report.

### Healing Logic

The repair layer distinguishes two paths:

**Repair suggestion (all failure categories)**
- Available for `SEL`, `NET`, `LOGIC`, `DATA`, `TIMING`, and `ENV`
- Produces LLM-guided, human-reviewed repair guidance
- Approved guidance is preserved in run memory and included in the Jira push

**Auto-patchable (intentionally narrow)**
- Only when the system can safely apply a concrete code/test patch and validate it
- Blocked when evidence points to: network/dependency problems, assertion mismatch, product behavior mismatch, requirement ambiguity, overlay/timing issues that are not true selector drift, weak confidence, or missing stable selector alternatives

When a repair **is patch-supported**, the app can:
- Propose a concrete patch
- Let a human approve or reject it
- Create a targeted healing branch
- Patch the failed spec
- Trigger Jenkins for only the failed spec/test

When a repair is **guidance-only**, the app can:
- Generate the repair guidance
- Let a human approve or reject it
- Preserve approved guidance in run memory
- Include the guidance in the Jira push

> The current healing model is best described as **guarded assisted recovery**, not autonomous self-repair.

---

## Current Capabilities

### What Is Working Today

- Real Jira fetch through Nexus-managed credentials
- ADF parsing and Acceptance Criteria extraction
- Qdrant lookup across separate risk, knowledge, healing, and execution memory layers
- LLM-driven generation for `Unit`, `API`, `Functional`, `UI`, and `Regression`
- Ollama fallback when the primary LLM provider fails
- Forensic Trail tagging across session artifacts
- Timeline-based UI with downloadable `.feature`, TypeScript, raw output, and ZIP bundles
- Human-in-the-loop scenario and code review before export
- LLM-guided review suggestions with accept/dismiss workflow
- Bulk approval actions for scenarios, code, or the entire run
- Export to runner repo, generated branch push, and Jenkins trigger from the UI
- Jenkins test report sync with pass/fail totals, failed-case selector, console evidence, and artifact links
- Post-run classification with recommended fix type, root-cause layer, healability, and QA report
- Repair suggestions for all failure categories with human review
- Targeted automatic patch validation only when the repair is patch-supported
- Jira comment pushback with QA-style run summaries

### What Needs More Refinement

- **Risk scoring** — still heuristic; should eventually become a learned or explicitly weighted model
- **True agent execution** — current roles are orchestrated sequentially, not autonomous parallel worker agents
- **Coverage fidelity** — generation-side coverage is partly inferred, though Jenkins-backed counts are used when available
- **Classification depth** — stronger than before, but still depends on rule extraction plus optional LLM summarization rather than full trace-native reasoning
- **Self-healing scope** — intentionally limited to safer selector-style failures; does not auto-fix assertion, requirement, or behavior mismatches
- **CI/CD hardening** — targeted reruns work, but production-grade persistence, promotion, and broader framework support still need hardening

### Capabilities Added But Not Fully Tested Yet

- Per-agent LLM routing for generation, classification, healing, and coverage
- Generic OpenAI-compatible LLM routing for Groq, OpenRouter, vLLM, and similar endpoints
- UI-configurable generation and runner profile controls
- Execution-memory, risk-memory, DOM-memory, and selector-ancestry population across Qdrant
- Targeted healing validation branch flow
- Role-based access and audit scaffolding
- Session persistence and run history through SQLite
- Broader story-based routing and recommendation logic
- Richer evidence extraction from Jenkins logs and artifacts

### What To Stabilize Before Broad Client Rollout

- Replace heuristic risk scoring with a governed weighted model
- Connect coverage, classification, and healing to real CI execution logs
- Harden Jenkins orchestration and retry behavior
- Add audit logging for connector actions
- Add authentication and role-based access around the Streamlit UI
- Validate self-healing suggestions against real execution history before patch application

---

## Prerequisites

| Dependency | Version | Purpose |
|------------|---------|---------|
| **Python** | 3.10+ | All agents, scripts, and the Streamlit app |
| **pip** | Latest | Python package management |
| **Docker** | 24+ | Qdrant vector DB (optional but recommended) |
| **Node.js** | 18+ (optional) | Running generated TypeScript tests |

### Optional Dependencies

| Dependency | Purpose |
|------------|---------|
| **Qdrant** (Docker) | Vector DB for forensic memory |
| **Jenkins** (Docker) | CI pipeline for test execution |
| **Ollama** (Docker/local) | Local LLM provider — pull `llama3.2` or similar |

### Python Virtual Environment

```bash
# Using venv
python3 -m venv .venv
source .venv/bin/activate   # Windows: .venv\Scripts\activate

# Using conda
conda create -n testing-hive python=3.11
conda activate testing-hive
```

---

## Installation

### 1. Clone

```bash
git clone <repository-url> bmad-test-builder
cd bmad-test-builder
```

### 2. Install Python Dependencies

```bash
pip install --upgrade pip
pip install streamlit python-dotenv requests
```

The core scripts (`llm_router.py`, `qdrant_forensics.py`, `jira_connector.py`, etc.) use **zero external dependencies** — only the Python standard library.

Optional cloud provider packages:

```bash
pip install openai anthropic google-generativeai
# Ollama: install separately via https://ollama.com
```

### 3. Start Qdrant

```bash
docker run -d --name qdrant \
  -p 6333:6333 \
  -v $(pwd)/qdrant_storage:/qdrant/storage \
  qdrant/qdrant
```

Verify:

```bash
curl http://localhost:6333/collections
# Expected: {"result":{"collections":[]},"status":"ok"}
```

### 4. Configure Environment

```bash
cp .env_example .env
```

Edit `.env` with your credentials. Gold standard template:

```env
# ── DevOps ──────────────────────────────────────────────────────────
JIRA_URL=https://your-domain.atlassian.net
JIRA_EMAIL=you@example.com
JIRA_API_KEY=your_jira_api_token
GITHUB_TOKEN=ghp_your_github_pat
GITHUB_REPO=owner/repo

# ── CI ──────────────────────────────────────────────────────────────
JENKINS_URL=http://jenkins:8080
JENKINS_USER=your_username
JENKINS_API_TOKEN=your_jenkins_token

# ── Intelligence ────────────────────────────────────────────────────
LLM_PROVIDER=openai          # openai | anthropic | gemini | deepseek | openai-compatible | local
LLM_API_KEY=sk-your-api-key
LLM_MODEL=gpt-4o
OPENAI_COMPATIBLE_BASE_URL=
OPENAI_COMPATIBLE_API_KEY=
OLLAMA_URL=http://localhost:11434
OLLAMA_MODEL=llama3.2

# ── Embedding ───────────────────────────────────────────────────────
EMBEDDING_PROVIDER=local
EMBEDDING_MODEL=mxbai-embed-large
EMBEDDING_DIMENSION=1024

# ── Qdrant ──────────────────────────────────────────────────────────
QDRANT_HOST=http://localhost:6333
QDRANT_RISK_COLLECTION=test-risk-history
QDRANT_KNOWLEDGE_COLLECTION=jira-tester-knowledge
QDRANT_DOM_COLLECTION=dom-aware-memory
QDRANT_HEALING_COLLECTION=selector-ancestry
QDRANT_EXECUTION_COLLECTION=execution-memory

# ── Session Defaults ────────────────────────────────────────────────
DEFAULT_JIRA_KEY=SCRUM-5
```

---

## Configuring Connectors

### Jira

Generate an API token at: https://id.atlassian.com/manage/api-tokens

```bash
# Verify connection
python3 skills/agent-test-architect/scripts/jira_connector.py projects

# Fetch a specific ticket (auto-parses ADF, extracts ACs)
python3 skills/agent-test-architect/scripts/jira_connector.py fetch --ticket SCRUM-5
```

### GitHub

Generate a Personal Access Token: **Settings → Developer settings → Personal access tokens → Fine-grained tokens** (scopes: `contents: read`, `pull_requests: write`)

```bash
python3 skills/agent-test-architect/scripts/github_connector.py --repo owner/repo
```

### Jenkins

```bash
docker run -d --name jenkins -p 8080:8080 -p 50000:50000 jenkins/jenkins:lts

# Verify and trigger
python3 skills/agent-test-architect/scripts/jenkins_orchestrator.py \
  trigger --job test-suite --params '{"BRANCH":"main"}'
```

### LLM Provider

| Provider | `LLM_PROVIDER` value | Notes |
|----------|----------------------|-------|
| OpenAI | `openai` | Requires `LLM_API_KEY` |
| Anthropic | `anthropic` | Requires `LLM_API_KEY` |
| Gemini | `gemini` | Requires `LLM_API_KEY` |
| DeepSeek | `deepseek` | Requires `LLM_API_KEY` |
| Generic OpenAI-Compatible | `openai-compatible` | Requires `OPENAI_COMPATIBLE_BASE_URL`; supports Groq, OpenRouter, vLLM |
| Local (Ollama) | `local` | `OLLAMA_URL` optional (default `http://localhost:11434`) |

> **Ollama caution:** Ollama is integrated and usable for experimentation, but local-model runs may time out more often, return partial output, or require parser recovery. It is **not the recommended primary path** for dependable artifact generation. Use it when API keys are unavailable, for offline exploration, or for low-cost demo setups.

```bash
python3 skills/agent-test-architect/scripts/llm_router.py list-models
```

**Local mode override:**

```env
LLM_PROVIDER=local
LLM_MODEL=llama3.2
OLLAMA_MODEL=llama3.2
EMBEDDING_PROVIDER=local
EMBEDDING_MODEL=mxbai-embed-large
```

### Qdrant

```bash
# Seed a forensic record
python3 skills/agent-test-architect/scripts/qdrant_forensics.py \
  store --id SCRUM-5 --payload '{"risk":7.5,"findings":"KYC compliance gaps detected"}'
```

---

## Execution Guide

### Launch the Command Center

```bash
streamlit run hive_app_v3.py
```

Open http://localhost:8501.

### Happy Flow Walkthrough

#### Step 1 — Home Tab
- Review the **Hive Mind** roster — all 10 agents displayed with roles and descriptions
- Check **Nexus Status** (🟢 Connected / 🔴 Disconnected)
- See **Qdrant Memory Health** card showing collection record counts and write activity

#### Step 2 — Sidebar: Connect Nexus
1. Click each expander: Jira, GitHub, Jenkins, Qdrant, LLM Provider, Embedding Provider
2. Fields are pre-populated from `.env`; typing overrides for this session; clearing does **not** override
3. Enter a Jira Ticket ID (default: `SCRUM-5`)
4. Click **"🔗 Connect Nexus"** → status changes to 🟢
5. Click **"🔎 Fetch & Forensics"** → fetches the Jira ticket, parses ADF, extracts ACs, queries Qdrant for historical risk records, and displays the Forensic Dashboard

#### Step 3 — Forensic Dashboard Tab
1. Enter a Jira ID and click **"🔎 Query Qdrant Memory"**
2. Aegis displays the **5D Risk Profile** with Critical Path Score, Integration Surface, Data Complexity, Edge Case Density, and Historical Instability
3. The **Overall Risk Score** (X/10) appears with recommendation (HIGH/MEDIUM/LOW) and a narrative finding with dispatched agent list

#### Step 4 — Live Execution Feed Tab
1. Ensure a ticket has been fetched
2. Click **"🚀 Engage Hive"**
3. Watch the progressive execution flow:
   - **Aegis** → parsed Jira context and risk framing
   - **Unit Agent** → Gherkin + executable test artifact
   - **API Agent** → Gherkin + executable test artifact
   - **Functional Agent** → Gherkin + executable test artifact
   - **UI Agent** → Gherkin + executable test artifact
   - **Regression Agent** → regression-focused artifact generation informed by Qdrant context
4. Each artifact is tagged with `@Forensic-{SessionID}`
5. Review artifacts in timeline or compact mode; download per-agent or full ZIP bundle
6. Use the **Human-in-the-Loop Review Gate** to:
   - Edit scenarios and code
   - Accept LLM-guided suggestions
   - Approve/reject individual artifacts or bulk-approve the entire run
7. After approval, export to the runner repo, push the generated branch, and trigger Jenkins

#### Step 5 — Post-Run Insights Tab
1. Click **"Sync Jenkins"** to refresh build status, failed cases, and report totals
2. Select a failed Jenkins case to inspect its specific evidence
3. **Classification Agent** (Nina Rao) analyzes the failure using console evidence, snippets, retry behavior, and session context
4. **Coverage Summary** (Daniel Cross) combines generated coverage with Jenkins-backed totals
5. **Recovery Decision** enables healing only when the failure is a safe selector-style candidate
6. If healing is approved, the app validates the patch by creating a targeted rerun branch and triggering Jenkins for only that spec/test
7. Push the QA-style summary back to Jira

### CLI Demo Pipeline

```bash
python3 skills/agent-test-architect/scripts/mock_gherkin_gen.py \
  --jira-key DEMO-001 \
  --forensic-id demo-run \
  --output ./demo-output
```

Produces 3 features × 2 scenarios = 6 test scaffolds in `./demo-output/`.

### Running Scripts Independently

```bash
# Qdrant: store a forensic record
python3 skills/agent-test-architect/scripts/qdrant_forensics.py \
  store --id SCRUM-5 --payload '{"risk":7.5,"findings":"KYC gap"}'

# Jira: fetch a ticket (auto-parses ADF)
python3 skills/agent-test-architect/scripts/jira_connector.py fetch --ticket SCRUM-5

# Jira: list all projects
python3 skills/agent-test-architect/scripts/jira_connector.py projects

# Jira: post a comment
python3 skills/agent-test-architect/scripts/jira_connector.py \
  comment --ticket SCRUM-5 --message "Risk analysis complete. @Forensic-A3F2"

# LLM: chat with a provider
python3 skills/agent-test-architect/scripts/llm_router.py \
  chat --provider openai --model gpt-4o --prompt "Analyze this test" \
  --system "You are the Classification Agent"

# LLM: list available local models
python3 skills/agent-test-architect/scripts/llm_router.py list-models
```

### ADF Parsing Pipeline

When Jira returns a ticket description in Atlassian Document Format (JSON):

```
description ADF JSON
    → _extract_text(adf_node)                      # recursive; handles all node types
        → text nodes concatenated                  # paragraphs, headings, lists, hardBreaks
        → block-level newlines preserved
    → Plain text description                       # ready for LLM consumption
    → _extract_acceptance_criteria_from_adf()      # finds ACs under heading
        → bulletList items extracted               # each listItem becomes one AC
    → Acceptance Criteria list                     # passed to LLM for artifact generation
```

The parser handles: `doc`, `paragraph`, `heading`, `bulletList`, `orderedList`, `listItem`, `text`, `hardBreak`, `inlineCard`, `mention`, `table` — all ADF block and inline node types.

### Jenkins Post Actions Reference

| Jenkins Output | Meaning |
|----------------|---------|
| `Archiving artifacts` | Jenkins is saving Playwright output, traces, screenshots, videos, logs, or generated reports |
| `Recording test results` | Jenkins is processing JUnit XML files to show pass/fail/skip totals in the job UI |
| `[Checks API] No suitable checks publisher found.` | Informational — Jenkins is not configured to publish SCM Checks annotations back to GitHub for this job. Not a test failure. |

> If Jenkins says `None of the test reports contained any result`, the run likely failed before real test execution and there will be no meaningful pass/fail totals.

---

## Project Structure

```
BMAD_Test_builder/
│
├── hive_app_v3.py                          # Streamlit Command Center
├── DEMO_WALKTHROUGH.md                     # Demo-facing product walkthrough
├── .env                                    # Your credentials (git-ignored)
├── .env_example                            # Gold standard template — copy to .env
├── README.md                               # This file
│
├── skills/
│   ├── agent-test-architect/               # Aegis — Central Brain (Maya Quinn)
│   │   ├── SKILL.md                        # BMAD agent definition
│   │   ├── customize.toml                  # BMAD customization config
│   │   ├── references/                     # System prompts / guidance docs
│   │   │   ├── analyze-requirements.md
│   │   │   ├── historical-risk-analysis.md
│   │   │   ├── memory-guidance.md
│   │   │   └── orchestrate-sub-agents.md
│   │   └── scripts/                        # All intelligence scripts
│   │       ├── jira_connector.py           # Jira API client (ADF parser built-in)
│   │       ├── qdrant_forensics.py         # Qdrant vector DB interface
│   │       ├── llm_router.py              # Unified LLM gateway (6 providers)
│   │       ├── github_connector.py         # GitHub API client
│   │       ├── jenkins_orchestrator.py     # Jenkins CI client
│   │       └── mock_gherkin_gen.py         # Demo Gherkin/TS generator
│   │
│   ├── unit-test-agent/                    # Ethan Cole — Logic Validation
│   ├── api-test-agent/                     # Priya Nair — API Contract
│   ├── functional-test-agent/              # Sofia Bennett — Business Flow
│   ├── ui-test-agent/                      # Lena Hart — Experience Assurance
│   ├── regression-test-agent/              # Marcus Vale — Regression Intelligence
│   ├── coverage-analysis-agent/            # Daniel Cross — Risk and Coverage
│   ├── test-classification-agent/          # Nina Rao — Story and Failure Classification
│   └── self-healing-agent/                 # Elena Park — Recovery Agent
│
├── testing_hive_services/                  # Internal-beta service layer
│   ├── routing_service.py
│   ├── generation_service.py
│   ├── memory_service.py
│   └── run_state_service.py
│
├── _bmad/                                  # BMAD framework config & memory
│   ├── config.toml
│   ├── config.user.toml
│   ├── memory/
│   │   └── agent-test-architect/           # Aegis's permanent memory
│   │       ├── INDEX.md                    # Memory index and cross-reference
│   │       ├── PERSONA.md                  # Aegis persona definition
│   │       ├── CREED.md                    # Core beliefs and principles
│   │       ├── BOND.md                     # Relationship protocols with other agents
│   │       ├── MEMORY.md                   # Session and long-term memory
│   │       ├── CAPABILITIES.md             # Capability manifest
│   │       ├── PULSE.md                    # Health and status tracking
│   │       └── sessions/                   # Session logs (e.g., 2026-05-03.md)
│   ├── custom/
│   │   └── agent-test-architect.toml
│   └── scripts/
│       ├── resolve_config.py
│       └── resolve_customization.py
│
├── docs/                                   # Additional documentation
└── _bmad-output/                           # BMAD output artifacts
    ├── planning-artifacts/
    └── implementation-artifacts/
```

---

## Client-Side Deployment

### Recommended Deployment Model

Start with a **single internal QA workstation or VM deployment**, then expand to shared infrastructure once Jira, Jenkins, Qdrant, and LLM access are stable.

**Minimum components:**
- Streamlit app host
- Python 3.10+
- Qdrant instance
- Optional Ollama runtime for local fallback
- Network access to client Jira, GitHub, and Jenkins

### Option 1: Single-Machine Pilot

Best for demos, pilots, and controlled validation.

```bash
streamlit run hive_app_v3.py --server.port 8501 --server.address 0.0.0.0
```

Expose the app only behind the client's internal VPN, jump host, or reverse proxy.

### Option 2: Internal Shared Service

Best for team-wide usage. Deploy:

- **Streamlit app** on an internal Linux VM or container host
- **Qdrant** as a persistent service with mounted storage
- **Ollama** on the same internal network if local inference is required

Recommended production add-ons: Nginx or Apache reverse proxy, HTTPS termination, process supervision via `systemd`, Docker Compose, or Kubernetes, and secrets injected through environment variables or vault tooling.

### Docker Compose

```yaml
version: "3.9"
services:
  hive:
    build: .
    command: streamlit run hive_app_v3.py --server.port 8501 --server.address 0.0.0.0
    ports:
      - "8501:8501"
    env_file:
      - .env
    depends_on:
      - qdrant

  qdrant:
    image: qdrant/qdrant
    ports:
      - "6333:6333"
    volumes:
      - ./qdrant_storage:/qdrant/storage
```

### Phase 2 Pilot Prototype

```bash
streamlit run hive_app_v3.py
```

Phase 2 adds:
- SQLite-backed run persistence (`testing_hive.db` by default)
- Jenkins-backed post-run signals for coverage/classification
- Approval tracking for healing suggestions
- Lightweight auth/audit scaffolding

Pilot deployment files: `Dockerfile.pilot`, `docker-compose.pilot.yml`, `requirements-pilot.txt`

Optional pilot auth env:

```env
HIVE_AUTH_REQUIRED=true
HIVE_USERS_JSON={"admin":{"password":"change-me","role":"admin"},"reviewer":{"password":"change-me-too","role":"reviewer"}}
HIVE_SESSION_DB=/app/data/testing_hive.db
```

### Client Deployment Notes

- Keep `.env` out of version control
- Prefer `JIRA_EMAIL + JIRA_API_TOKEN` for Jira Cloud
- Validate outbound access to: Atlassian, GitHub, Jenkins, Qdrant host, Ollama host (if local inference is enabled)

---

## Development

### Adding a New Agent

1. Create `skills/<agent-name>/` directory
2. Add `SKILL.md` with YAML frontmatter (`name:`, `description:`)
3. Add `customize.toml` with agent metadata
4. Add `references/` with capability documentation (e.g., `generate-*-tests.md`)
5. Add `scripts/` with executable implementations
6. Register the agent in the Hive Mind roster within `hive_app_v3.py`

### Running All Tests

```bash
python3 -m pytest tests/ -v
```

### Verifying Configuration

```bash
python3 -c "
from pathlib import Path
env = Path('.env')
if env.exists():
    print('.env file found')
    for line in env.read_text().splitlines():
        if '=' in line and not line.startswith('#'):
            key, val = line.split('=', 1)
            hidden = val[:4] + '****' if len(val) > 4 else '****'
            print(f'  {key.strip()}={hidden}')
else:
    print('.env file NOT FOUND — copy from .env_example')
"
```

### Verifying All Scripts Compile

```bash
python3 -c "
import py_compile
scripts = [
    'skills/agent-test-architect/scripts/llm_router.py',
    'skills/agent-test-architect/scripts/qdrant_forensics.py',
    'skills/agent-test-architect/scripts/jira_connector.py',
    'skills/agent-test-architect/scripts/github_connector.py',
    'skills/agent-test-architect/scripts/jenkins_orchestrator.py',
    'skills/agent-test-architect/scripts/mock_gherkin_gen.py',
]
for s in scripts:
    py_compile.compile(s, doraise=True)
    print(f'OK: {s}')
print('All scripts compile clean.')
"
```

### Aegis Memory Files

Aegis's permanent memory lives in `_bmad/memory/agent-test-architect/`:

| File | Purpose |
|------|---------|
| `INDEX.md` | Memory index and cross-reference |
| `PERSONA.md` | Aegis persona and behavior rules |
| `CREED.md` | Core beliefs and principles |
| `BOND.md` | Relationship protocols with other agents |
| `MEMORY.md` | Session and long-term memory |
| `CAPABILITIES.md` | Capability manifest |
| `PULSE.md` | Health and status tracking |
| `sessions/` | Individual session logs (e.g., `2026-05-03.md`) |

---

## Troubleshooting

### Nexus / Configuration

| Symptom | Likely Cause | Fix |
|---------|-------------|-----|
| Jira falls back to "Mock" | `JIRA_API_KEY` empty in `.env` or Nexus overrode with `""` | Check `.env` has valid key. Empty sidebar fields no longer override (fixed). |
| Sidebar field shows blank | No value in `.env` for that key | Add the key to `.env` and click "🧹 Reset" to reload |
| `JIRA_URL is not set` error | Missing `JIRA_URL` in `.env` | Add `JIRA_URL=https://your-domain.atlassian.net` to `.env` |

### Docker / Qdrant

| Symptom | Likely Cause | Fix |
|---------|-------------|-----|
| `Connection refused` querying Qdrant | Container not running | `docker start qdrant` or `docker run -d -p 6333:6333 qdrant/qdrant` |
| `collection not found` | First-time setup, no data seeded | Run a forensic scan from the Streamlit app to initialize |
| Port 6333 already in use | Another service on that port | `docker run -d -p 6334:6333 qdrant/qdrant` and update `.env` |

### LLM / API

| Symptom | Likely Cause | Fix |
|---------|-------------|-----|
| `HTTP 401 Unauthorized` | API key missing or invalid | Check `LLM_API_KEY` in `.env` |
| `HTTP 429 Rate Limit` | API rate limit exceeded | Wait 60 seconds or switch to `local` provider |
| `HTTP 502 Bad Gateway` (Local) | Ollama not running | `ollama serve` or `docker run -d -p 11434:11434 ollama/ollama` |
| `Model not found` (Local) | Model not pulled | `ollama pull llama3.2` |
| `Provider 'deepseek' not in list` | Typo or unregistered provider | `LLM_PROVIDER` must be one of: `openai`, `anthropic`, `gemini`, `deepseek`, `openai-compatible`, `local` |

### Jira

| Symptom | Likely Cause | Fix |
|---------|-------------|-----|
| `Jira API error 401` | API token invalid or expired | Regenerate at https://id.atlassian.com/manage/api-tokens |
| `Jira API error 404` | Ticket key doesn't exist in this project | Verify the ticket key (e.g., `SCRUM-5`) and project permissions |
| `Jira API error 403` | IP restricted or insufficient permissions | Check Jira IP allowlist and user permissions |
| Description appears as raw JSON | ADF parser not triggered | Ensure `fields['description']` is a dict (ADF format). The `from_api()` method auto-detects dict vs string. |
| Acceptance Criteria empty | ACs not nested under a heading in ADF | Check the ticket's ADF structure. The parser looks for a heading containing "Acceptance Criteria" followed by a bullet list. |

### Streamlit

| Symptom | Likely Cause | Fix |
|---------|-------------|-----|
| App doesn't load | Missing dependencies | `pip install streamlit` |
| `st.rerun` error | Old Streamlit version | `pip install --upgrade streamlit` |
| Session state resets | Page refresh | By design — each session gets a new Forensic Trail ID |
| Port 8501 in use | Another Streamlit app | `streamlit run hive_app_v3.py --server.port 8502` |
| Sidebar fields don't reflect `.env` changes | Stale session state | Click "🧹 Reset" to reload all values from `.env` |

---

## License

See [LICENSE](./LICENSE) and [NOTICE](./NOTICE).

---

<p align="center">
  <strong>🐝 The Hive thinks. The Hive remembers. The Hive tests.</strong><br>
  <em>Built on the BMAD Agent Builder framework.</em>
</p>
