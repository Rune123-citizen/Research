# Agentic Workflow — Master-Level Technical Masterclass Report

## 0. Scope and Evidence

This report is based on the uploaded `Agentic_workflow.zip` and the source/configuration/tests contained in it.

### Repository facts observed

- Primary language: **Python**
- Configuration: **YAML**
- Packaging: **PEP 621 `pyproject.toml` + Hatchling**
- Core source: approximately **4,500 lines of Python** across the application modules.
- Main package: `agentic_workflow/`
- Test suite: `tests/`
- Persistent execution state: `runs/`
- Configuration: `config/default.yaml`, `config/ladder.yaml`
- CLI entry point: `plan-mode = agentic_workflow.cli:main`
- No application database schema or ORM is present. Run state is intentionally **file-backed**.
- The project depends on an external **Graphify** executable/module for deterministic code-graph extraction.
- The registry contains an `anthropic` provider branch, but the uploaded source tree does **not** contain `agentic_workflow/providers/anthropic.py`; therefore that configured provider cannot currently be constructed from this archive.

A local test execution in the analysis environment produced **134 passed, 14 failed, 15 skipped**. The failures were concentrated around `graphify`: the test process could not import the external `graphify` module. One concurrency-related pipeline test also consequently failed because `a01` could not persist its architecture artifact. This is primarily an environment/dependency packaging issue rather than evidence that the corresponding pipeline logic is intrinsically incorrect.

---

# 1. Executive Overview

## 1.1 What problem does this project solve?

`agentic_workflow` is a **provider-agnostic, multi-agent planning system for software repositories**.

Its purpose is not to autonomously modify a repository. Instead, it takes:

1. an existing codebase,
2. a natural-language feature request,

and produces a **structured, reviewable implementation plan**.

The key design philosophy is:

> **LLMs propose; deterministic code measures and verifies. Humans decide ambiguous or high-impact choices.**

This is significantly more disciplined than simply asking an LLM:

> "Look at this repository and tell me how to implement feature X."

The system decomposes planning into specialized stages:

```text
Feature Request
      |
      v
+------------------+
| A01 Codebase     |----> Architecture artifact
+------------------+
      |
      |       +----------------------+
      |       | A02 Requirements     |----> Requirements artifact
      |       +----------------------+
      |                  |
      +------------------+
               |
               v
      +------------------+
      | A03 Tech Stack   |----> TechStack artifact
      +------------------+
               |
               v
      +------------------+
      | A04 Architecture |----> Options + measured blast radius
      | Delta            |
      +------------------+
               |
          Human Gate
               |
               v
      +------------------+
      | A05 Implementation|
      | Plan             |----> ordered executable steps
      +------------------+
               |
               v
      +------------------+
      | A06 Review       |----> critique + verification
      +------------------+
               |
          Revision loop
               |
               v
      +------------------+
      | Human Approval   |
      +------------------+
               |
               v
          plan.md
```

## 1.2 Target use case

The project is especially useful for:

- planning changes to unfamiliar repositories,
- generating implementation plans for coding agents,
- architectural impact analysis,
- feature planning before code modification,
- repository-aware AI assistants,
- offline/replayable LLM workflows,
- comparing provider/model behavior,
- controlling LLM hallucination with deterministic checks.

It is best understood as a **software-engineering planning copilot**, not an autonomous coding agent.

---

# 2. The Most Important Architectural Idea

The strongest architectural choice is the **blackboard architecture**.

Agents do not directly call one another.

Instead:

```text
Agent A
   |
   v
architecture.json
   |
   v
Agent B reads artifact
   |
   v
requirements.json
   |
   v
Agent C reads artifacts
```

The agents communicate through typed artifacts persisted to disk.

The `Run` object is therefore simultaneously:

- a workflow state manager,
- a persistent blackboard,
- an event journal,
- a human-gate coordinator,
- a resume boundary.

This dramatically reduces coupling.

### Why this matters

A naive multi-agent design might look like:

```text
A01 -> A02 -> A03 -> A04 -> A05 -> A06
```

where each Python agent directly invokes another agent.

That creates temporal and implementation coupling.

This project instead does:

```text
A01 -> architecture.json
A02 -> requirements.json
A03 -> techstack.json
A04 -> arch_delta.json
A05 -> impl_plan.json
A06 -> review.json
```

Every stage has a clear contract.

That means an agent can be replaced without requiring every other agent to know its internal implementation.

---

# 3. System Architecture

## 3.1 Architectural pattern

The project combines several patterns:

### A. Pipeline / staged workflow

The orchestrator defines waves:

```python
WAVES = [
    [("a01", a01_codebase.run), ("a02", a02_requirements.run)],
    [("a03", a03_techstack.run)],
    [("a04", a04_arch_delta.run)],
    [("a05", a05_impl_plan.run)],
    [("a06", a06_review.run)],
]
```

So the execution model is:

```text
Wave 1: A01 || A02
           |
           v
Wave 2: A03
           |
           v
Wave 3: A04
           |
           v
Wave 4: A05
           |
           v
Wave 5: A06
```

A01 and A02 can execute concurrently because:

- A01 needs the repository.
- A02 primarily needs the user's feature request.

Everything after them has dependencies.

### B. Blackboard architecture

Artifacts stored in the run directory are the shared state.

### C. Adapter architecture

LLM providers implement a common `Provider` contract.

### D. Decorator/wrapper architecture

Providers are wrapped:

```text
OpenAICompatProvider
        |
        v
PromptedTools       (when native tools unavailable)
        |
        v
Retrying
        |
        v
Cached
        |
        v
Metered
```

The caller sees the same `complete()` interface.

### E. Human-in-the-loop architecture

The pipeline deliberately stops at decision gates.

### F. Event-sourced-ish execution history

`events.jsonl` records lifecycle events.

It is not a full event-sourcing system, but it uses an append-only event log to reconstruct important execution state.

---

# 4. End-to-End Data Flow

## 4.1 CLI flow

```text
plan-mode run REPO "FEATURE REQUEST"
          |
          v
       cli.py
          |
          v
      Run.create()
          |
          v
 pipeline.execute()
          |
          +------------------------------+
          |                              |
          v                              v
       Config                         Run state
          |                              |
          v                              v
 provider construction             runs/<run-id>/
          |                              |
          +--------------+---------------+
                         |
                         v
                    A01 + A02
                         |
                         v
             typed Pydantic artifacts
                         |
                         v
                 A03 -> A04 -> gate
                         |
                         v
                        A05
                         |
                         v
                        A06
                         |
                         v
                approval/revision
                         |
                         v
                      plan.md
```

## 4.2 LLM tool loop

The core agent loop is:

```text
Prompt
  |
  v
Provider.complete()
  |
  v
Model response
  |
  +---- normal answer ----> return
  |
  +---- tool calls -------> Toolbox.execute()
                                  |
                                  v
                            ToolResult list
                                  |
                                  v
                        append ALL results
                                  |
                                  v
                         Provider.complete()
                                  |
                                  v
                              repeat
```

This is the fundamental mechanism that converts a stateless LLM completion API into an agent.

---

# 5. Component Breakdown

# 5.1 `agentic_workflow/providers/base.py`

This file defines the provider-neutral domain model.

Important abstractions:

- `Role`
- `StopReason`
- `TextBlock`
- `ToolCall`
- `ToolResult`
- `Message`
- `ToolSpec`
- `Usage`
- `ModelResponse`
- `ModelCapabilities`
- `ProviderError`
- `Provider`

## Why this layer exists

Different model providers disagree about:

- message formats,
- tool-call representation,
- tool-result representation,
- stop reasons,
- structured output,
- provider-specific metadata.

Instead of spreading provider-specific conditionals throughout the system, this project creates a neutral intermediate representation.

Conceptually:

```text
Provider A wire format
        |
        v
    normalize
        |
        v
Neutral Message/ToolCall/Response
        |
        v
application
        |
        v
Neutral representation
        |
        v
    encode
        |
        v
Provider B wire format
```

This is classic **anti-corruption/adaptor-layer design**.

## `ToolCall.extra`

One particularly mature design detail is preservation of unknown provider metadata.

For example, Gemini-style reasoning/tool metadata may need to be echoed back exactly.

Instead of trying to understand every provider-specific field:

```python
extra: dict[str, Any]
```

is carried through the system.

This is a form of **lossless passthrough**.

That is important because provider APIs can reject requests if opaque metadata disappears between turns.

---

# 5.2 `OpenAICompatProvider`

This adapter talks to:

```text
/v1/chat/completions
```

It can therefore cover many servers/providers exposing an OpenAI-compatible API.

Examples configured in the repository include:

- Gemini OpenAI-compatible endpoint
- vLLM
- Ollama

The adapter performs:

```text
neutral messages
       |
       v
OpenAI-compatible JSON
       |
       v
HTTP POST
       |
       v
wire response
       |
       v
neutral ModelResponse
```

## Tool encoding

A neutral:

```text
ToolSpec
```

becomes:

```json
{
  "type": "function",
  "function": {
    "name": "...",
    "description": "...",
    "parameters": {}
  }
}
```

## Tool argument decoding

Model APIs often return tool arguments as a JSON string.

The adapter converts:

```text
"{\"path\":\"src/main.py\"}"
```

into:

```python
{"path": "src/main.py"}
```

If parsing fails, it raises a structured `ProviderError`.

---

# 5.3 Provider capability registry

`providers/registry.py` maintains static capability information.

This includes:

- context window,
- maximum output,
- tool support,
- parallel tool support,
- JSON schema support,
- caching support,
- pricing.

### Why static rather than probing?

A capability probe would cost an LLM request.

Also, self-hosted servers may advertise capabilities that the loaded model does not really support.

Therefore:

```text
model id
  |
  v
static capability registry
  |
  v
capability object
```

YAML can override the defaults.

This is a deliberate **conservative capability model**.

Unknown models default to:

```text
no native tools
no native JSON schema
no parallel tools
```

Then the degradation ladder provides compatibility.

---

# 5.4 Capability degradation ladder

One of the most important concepts is:

```text
Native tools available?
       |
     yes
       |
       v
use native tool calling

       no
       |
       v
PromptedTools
       |
       v
encode tool instructions into prompt
       |
       v
parse model-produced tool JSON
```

This makes the application more portable.

Instead of saying:

> "This model doesn't support tools, so the system fails."

the system says:

> "Use a lower capability mode."

That is a **graceful degradation strategy**.

---

# 5.5 `PromptedTools`

When native function calling is unavailable, tools are described in the prompt.

The model returns structured tool-call JSON, which is parsed into the same neutral `ToolCall` object.

Therefore the rest of the application cannot tell whether a tool call originated from:

```text
native API function calling
```

or:

```text
prompted tool protocol
```

This is an excellent example of abstraction preserving application-level invariants.

---

# 5.6 Provider wrappers

`providers/wrappers.py` contains three major cross-cutting concerns.

## `Cached`

Purpose:

- record LLM calls,
- replay them,
- enable deterministic development,
- reduce cost.

Cache key:

```text
SHA-256(
    model +
    serialized request payload
)
```

The response is serialized to disk.

This enables:

```text
live run
   |
   v
LLM
   |
   v
cache file

future run
   |
   v
same request
   |
   v
cache hit
   |
   v
no network request
```

### Why this is valuable

LLM systems are expensive and nondeterministic.

Caching makes testing much more practical.

It also enables:

```bash
--replay
```

for offline execution.

---

## `Retrying`

Handles:

- rate limits,
- timeouts,
- temporary HTTP failures,
- minimum request spacing.

Retryable statuses include:

```text
408
409
429
500
502
503
504
529
```

The backoff strategy uses:

```text
server Retry-After if available
otherwise exponential backoff
```

with jitter.

This is important for free-tier providers.

---

## `Metered`

Captures:

- input tokens,
- output tokens,
- cached input tokens,
- latency,
- stop reason,
- tool calls,
- transcript information.

This wrapper centralizes observability.

Instead of every agent remembering to log usage:

```text
Agent
  |
  v
Metered Provider
  |
  +--> usage ledger
  +--> transcript
  |
  v
actual provider
```

This is a strong example of the **Decorator pattern**.

---

# 6. Harness Layer

The harness turns LLM completions into actual agent behavior.

---

# 6.1 `harness/loop.py`

The core loop is:

```text
while turns < max_turns:

    call model

    if final response:
        return

    if tool calls:
        execute tools
        append tool results
        continue
```

## Critical invariant

If the model emits multiple tool calls:

```text
call A
call B
call C
```

the harness executes them and appends:

```text
result A
result B
result C
```

as one logical result turn.

This matters because many model APIs expect the assistant's complete tool-call turn to be followed by corresponding tool results.

Dropping a result can corrupt the conversation protocol.

---

# 6.2 Parallel tool execution

Parallel execution happens only if:

```text
multiple calls
AND
provider supports parallel tools
AND
every tool is marked parallel_safe
```

Then:

```text
Tool A \
Tool B  ---> ThreadPoolExecutor ---> results
Tool C /
```

This is a practical concurrency optimization.

The system does not blindly parallelize every tool because some tools may mutate state or have ordering dependencies.

---

# 6.3 Tool failures are model-visible

Instead of:

```python
raise Exception(...)
```

the toolbox returns:

```text
ToolResult(..., is_error=True)
```

Therefore:

```text
tool fails
   |
   v
error becomes model context
   |
   v
model can adapt
```

This is a core agentic design principle.

A tool failure is often recoverable by reasoning.

---

# 6.4 Loop bounds

`max_turns` prevents:

```text
model -> tool -> tool -> tool -> ...
```

from becoming an infinite loop.

If the limit is exceeded:

```text
LoopLimitExceeded
```

is raised.

This is an important safety and reliability property.

---

# 7. Tooling Layer

`harness/tools.py` defines the repository interaction surface.

Core tools include:

- `read_file`
- `grep`
- `glob`
- `list_dir`

Optional tools include:

- `web_search`
- `web_fetch`
- `bash_readonly`
- `graph_query`

---

# 7.1 Path containment

The toolbox resolves repository-relative paths and prevents path escape.

Conceptually:

```text
requested path
      |
      v
resolve()
      |
      v
inside repo?
   /       \
 yes       no
 |          |
 v          v
allow     reject
```

This is important because an LLM is untrusted input.

---

# 7.2 Result truncation

Tool results are capped.

Why?

Because tool output consumes model context.

Without limits:

```text
grep huge repository
      |
      v
100 MB output
      |
      v
context overflow
```

The system caps characters and matches.

This is **context-budget engineering**.

---

# 7.3 Read-only bash

The project intentionally restricts shell access.

Allowed executables include tools such as:

```text
git
ls
cat
head
tail
wc
find
python
python3
node
npm
pip
uv
ruff
mypy
tsc
pytest
```

Git is further restricted to read-only subcommands.

It also blocks:

```text
>
>>
|
;
&&
&
```

This prevents common shell injection/write patterns.

It is not a perfect security boundary by itself, but it is a meaningful reduction of the model's attack surface.

---

# 8. Deterministic Graph Analysis

`agentic_workflow/orchestrator/graph.py` consumes Graphify's graph.

It calculates:

- node degree,
- hubs,
- communities,
- files touched,
- communities crossed.

This is central to **blast-radius measurement**.

---

# 8.1 Why graph analysis exists

An LLM can say:

> "This is a small change."

But the LLM should not be trusted to quantify impact.

The project instead computes:

```text
files touched
communities crossed
hub files touched
public API change
cycle creation
```

The first three are graph-derived.

The last two are model-supplied semantic judgments.

This is a hybrid architecture:

```text
LLM judgment
     +
deterministic analysis
```

---

# 8.2 Hub detection

The system does not simply take the top 15 nodes.

It calculates average connectivity and requires a hub to exceed:

```text
average degree × HUB_FACTOR
```

where:

```text
HUB_FACTOR = 2.0
```

This avoids a small repository being incorrectly classified as having many hubs.

---

# 8.3 Blast radius formula

The score is:

```text
score =
    files_touched
  + 3 × communities_crossed
  + 5 × hubs_touched
  + 8 × public_api_changed
  + 8 × creates_cycle
```

A score above:

```text
40
```

is considered too large for ordinary planning.

This is an important engineering principle:

> Replace vague instructions such as "keep the change small" with an explicit measurable policy.

---

# 9. Typed Artifacts

`orchestrator/artifacts.py` contains Pydantic models.

The major artifacts are:

```text
Triage
Architecture
Requirements
TechStack
ArchDelta
ImplPlan
Review
```

These form the workflow's contracts.

---

# 9.1 Architecture

Contains:

- summary,
- languages,
- entry points,
- components,
- hubs,
- graph commit,
- cycles,
- file count.

---

# 9.2 Requirements

Contains:

- clear/ambiguous flag,
- requirements,
- out-of-scope items,
- open questions,
- behavior-level approaches.

A particularly important constraint is that A02 is **not allowed to decide technology**.

This separation prevents requirements from becoming polluted by implementation assumptions.

---

# 9.3 TechStack

Separates:

```text
existing
additions
notes
```

Each addition has:

```text
name
purpose
why
risk
```

The bias is:

> Reuse what already exists.

This reduces dependency sprawl.

---

# 9.4 Implementation Plan

Each `Step` contains:

- number,
- title,
- files,
- dependencies,
- exact change,
- runnable check,
- review point,
- blast radius,
- verification state.

This is much stronger than a simple bullet list.

It makes a plan executable by another engineer or coding agent.

---

# 9.5 Review

Findings have:

```text
dimension
claim
evidence
verdict
rebuttal
round
```

Verdicts:

```text
UNVERIFIED
CONFIRMED
REJECTED
```

This is effectively a small evidence/verification model.

---

# 10. Agent-by-Agent Deep Dive

# A01 — Codebase Analysis

File:

```text
agentic_workflow/agents/a01_codebase.py
```

A01 does not blindly ask an LLM to crawl the repository.

It first invokes:

```text
python -m graphify update <repo>
```

Graphify performs deterministic AST-based analysis.

Then the model receives:

```text
file list
+
graph report
```

and converts this into the typed `Architecture` artifact.

### Major optimization

Architecture results can be reused when the graph's commit matches a prior run.

Therefore:

```text
same repository commit
       |
       v
reuse architecture
       |
       v
avoid expensive model work
```

This is an important cost optimization.

---

# A02 — Requirements

A02 converts the feature request into product requirements.

It has two modes:

```text
clear request
     |
     v
requirements

ambiguous request
     |
     v
2–4 behavior options
     |
     v
human gate
     |
     v
selected behavior
     |
     v
requirements
```

The system deliberately refuses to silently guess.

That is a major human-in-the-loop design decision.

---

# A03 — Tech Stack

A03 reads:

```text
Architecture
Requirements
```

and determines what technologies are needed.

It strongly prefers existing dependencies.

If it proposes new dependencies, it uses web research to check:

- existence,
- maintenance,
- current version,
- deprecation,
- whether the dependency already exists in the repository.

This prevents the classic LLM failure mode:

> hallucinated or obsolete packages.

---

# A04 — Architecture Delta

A04 answers:

> How should the existing architecture change?

It proposes 2–3 alternatives.

Each option includes:

```text
files
changes
blast radius
escalation status
```

Then deterministic graph analysis recalculates the blast radius.

The model cannot simply claim:

> "blast radius = 3"

and have the system trust it.

The graph overwrites the measurable parts.

Finally the user chooses an architecture option.

---

# A05 — Implementation Plan

A05 produces the primary deliverable.

The system prompt demands implementation-level precision.

A good step must say:

```text
which file
which symbol
which signature
where the change occurs
what data shape is involved
what dependency it has
how success is tested
```

This is deliberately more detailed than:

> "Add caching."

The goal is that another engineer can execute the plan without rediscovering the design.

---

# A05 Optional Sandbox Verification

When enabled:

```text
--verify
```

the system asks the LLM to write a tiny verification program.

That program is run against a disposable copy of the repository.

Flow:

```text
planned step
    |
    v
LLM writes probe
    |
    v
sandbox copy
    |
    v
execute probe
    |
    +---- success ---> verified=True
    |
    +---- failure ---> rewrite step
                         |
                         v
                      re-probe
```

This is a powerful concept:

> Test the assumptions behind a plan before implementing the plan.

---

# A06 — Review

A06 attacks the plan.

The current implementation uses two principal critic dimensions:

```text
architecture_fit
testability
```

Other dimensions are deliberately moved to deterministic checks:

```text
file references
sequencing
blast radius
```

This is a sophisticated division of labor.

### Critic → verifier architecture

```text
Critic
  |
  v
Finding
  |
  v
Verifier
  |
  +---- CONFIRMED
  |
  +---- REJECTED
  |
  +---- UNVERIFIED
```

The verifier is instructed to try to **disprove** the claim.

This resembles adversarial testing.

---

# 11. Human Gates

There are several explicit decision points.

## Requirements gate

When requirements are ambiguous:

```text
Which behavior do you want?
```

The run pauses.

## Architecture gate

The user chooses:

```text
Which architecture change should the plan implement?
```

## Final approval

The user sees the plan and can:

```text
approve
revise
```

The approval cycle is bounded to:

```text
3 rounds
```

This prevents infinite revision loops.

---

# 12. Persistence Model

The project intentionally avoids a database.

A run looks approximately like:

```text
runs/
└── <run-id>/
    ├── run.json
    ├── events.jsonl
    ├── transcript.jsonl
    ├── architecture.json
    ├── requirements.json
    ├── techstack.json
    ├── arch_delta.json
    ├── impl_plan.json
    ├── review.json
    ├── plan.md
    ├── report.html
    ├── graphify-out/
    └── gates/
        ├── approach.json
        ├── approach.answer.json
        ├── arch_change.json
        └── ...
```

This is a **file-backed state machine**.

---

# 13. Why No Database?

For this workload, a database would initially add more complexity than value.

A run is:

- relatively low-volume,
- naturally hierarchical,
- mostly append/read,
- useful as inspectable files,
- easy to copy/archive,
- easy to replay.

JSON files therefore provide excellent transparency.

However, a database becomes attractive when the system evolves toward:

- many concurrent users,
- thousands of runs,
- remote workers,
- distributed orchestration,
- querying across historical runs,
- multi-user permissions,
- transactional state transitions.

At that point a PostgreSQL-backed run store would be a sensible modernization.

---

# 14. CLI

The CLI is `agentic_workflow/cli.py`.

Commands:

```text
plan-mode ask
plan-mode run
plan-mode resume
plan-mode answer
plan-mode ui
plan-mode report
```

## `ask`

Runs a single agent-style repository question.

Example:

```bash
plan-mode ask <repo> "where is authentication handled?"
```

## `run`

Runs the complete planning pipeline.

Conceptually:

```bash
plan-mode run <repo> "add feature X"
```

## `resume`

Continues a persisted run after a gate.

## `answer`

Writes a gate answer to disk.

## `ui`

Starts the FastAPI browser UI.

## `report`

Generates a self-contained HTML report.

---

# 15. Browser UI

The UI is implemented using:

- FastAPI
- vanilla JavaScript
- generated HTML/CSS

It intentionally uses **polling instead of SSE**.

Why?

Because run state already exists on disk.

Therefore:

```text
Browser
   |
   | poll
   v
FastAPI
   |
   v
run files
```

No second event-delivery infrastructure is required.

The UI can therefore survive server restarts more naturally.

---

# 16. Report Generation

`ui/report.py` generates a single self-contained HTML file.

It displays:

- request,
- repository,
- stage statuses,
- decisions,
- review findings,
- implementation plan,
- token usage.

No external frontend build is required.

This is a pragmatic choice for an internal developer tool.

---

# 17. Theoretical Foundations

# 17.1 Agentic systems

An LLM by itself is:

```text
input -> output
```

An agent is closer to:

```text
observe
  |
reason
  |
act
  |
observe result
  |
reason again
  |
...
```

This project implements that loop explicitly.

---

# 17.2 Tool calling

A tool-enabled agent follows:

```text
LLM
 |
 | tool call
 v
Tool
 |
 | result
 v
LLM
```

The model controls *what* to ask for, while deterministic code controls *how* the action is performed.

---

# 17.3 Multi-agent specialization

Instead of one giant prompt:

```text
"Understand repo, requirements, architecture, dependencies,
implementation and review."
```

the project creates specialized roles.

This reduces cognitive load and improves separation of concerns.

---

# 17.4 Structured generation

Pydantic schemas provide a bridge between:

```text
LLM output
```

and:

```text
typed program data
```

The flow is:

```text
Pydantic model
      |
      v
JSON Schema
      |
      v
LLM structured response
      |
      v
Pydantic validation
```

For models without native schema support, the schema is included in the prompt.

---

# 17.5 Retry theory

The retry mechanism is based on exponential backoff.

Conceptually:

```text
attempt 0 -> ~1s
attempt 1 -> ~2s
attempt 2 -> ~4s
attempt 3 -> ~8s
...
```

with server-provided retry delay taking precedence when available.

Jitter reduces synchronized retries.

---

# 17.6 Context-window management

Agent conversations grow over time.

The project handles this in two ways:

1. turn limits,
2. history trimming.

Older tool results can be dropped while retaining enough conversational structure.

This is an example of **bounded-memory reasoning**.

---

# 17.7 DAG/dependency reasoning

Implementation steps form a dependency graph.

A valid plan must satisfy:

```text
if step B depends on A:

A < B
```

The deterministic checks reject forward dependencies.

This is effectively a directed acyclic dependency constraint.

---

# 17.8 Graph centrality

The architecture graph uses connectivity to identify hubs.

A high-degree node is expensive to modify because many relationships converge on it.

Therefore:

```text
centrality ~= architectural leverage/risk
```

This is not a universal measure of software complexity, but it is a useful approximation for planning.

---

# 18. Tech Stack and Trade-Off Analysis

| Technology | Role | Why it fits | Trade-off |
|---|---|---|---|
| Python | Core language | Fast development, excellent AI ecosystem | Lower raw concurrency/performance than Go/Rust |
| Pydantic v2 | Typed artifacts | Strong validation + JSON Schema generation | Adds dependency and model semantics |
| HTTPX | Provider HTTP | Modern HTTP client, timeouts, good Python API | Another abstraction over urllib |
| PyYAML | Config | Simple human-readable configuration | YAML has complex parsing semantics |
| Rich | CLI UX | Excellent terminal rendering | Not needed for machine-only execution |
| FastAPI | Local UI API | Lightweight typed HTTP server | Adds web-server complexity |
| Uvicorn | ASGI server | Natural FastAPI runtime | Extra runtime dependency |
| NetworkX | Declared dependency | Useful for graph operations | Current local `graph.py` mostly processes Graphify JSON directly; dependency appears underused |
| Graphify | AST/dependency graph | Deterministic structural analysis | External dependency; packaging is currently incomplete |
| JSONL | Events/transcripts | Append-only, inspectable, simple | Weak transactional guarantees |
| File system | Run state | Transparent, portable, easy replay | Weak for distributed/high-concurrency workloads |
| OpenAI-compatible protocol | Model abstraction | One adapter covers many endpoints | Compatibility is not identical across providers |

---

# 19. Important Packaging/Repository Issues

## 19.1 Missing Graphify dependency declaration

`a01_codebase.py` invokes:

```bash
python -m graphify update ...
```

but `pyproject.toml` does not declare Graphify.

This is a significant reproducibility issue.

A fresh installation based only on:

```bash
pip install .
```

may not have Graphify.

### Recommended fix

Either:

```text
declare Graphify as an install dependency
```

or provide a documented extra:

```text
agentic-workflow[graph]
```

or make Graphify an explicit external prerequisite with a startup diagnostic.

---

## 19.2 Broken bundled `.venv`

The uploaded archive contains a `.venv`, but its `pyvenv.cfg` points to a machine-specific interpreter location.

Therefore the bundled environment is not portable.

A repository archive should generally exclude:

```text
.venv/
```

and document reproducible environment setup instead.

---

## 19.3 Missing Anthropic adapter

The registry contains:

```python
from .anthropic import AnthropicProvider
```

but `providers/anthropic.py` is absent from the uploaded source.

Therefore the configured:

```yaml
kind: anthropic
```

provider is incomplete.

This is particularly important because the configuration suggests Anthropic support but the implementation archive does not actually contain it.

---

## 19.4 Runtime-only dependencies

The main `pyproject.toml` dependencies do not include the UI runtime packages even though the UI imports:

```text
fastapi
uvicorn
```

The base dependencies also include some packages that are not central to the currently inspected implementation.

A production package should separate:

```text
core dependencies
UI extras
development dependencies
optional graph dependencies
```

---

# 20. Current Test Health

A test run in the analysis environment produced:

```text
134 passed
14 failed
15 skipped
```

The dominant failure:

```text
No module named graphify
```

This occurred in tests requiring real Graphify execution.

The failures cascade into pipeline tests because A01 is the first wave stage.

This reveals an important engineering lesson:

> Integration dependencies should be explicit and validated before the full test suite starts.

A useful improvement would be a startup/test fixture check:

```text
if graphify unavailable:
    report exactly how to install it
```

instead of allowing multiple downstream tests to fail through the same root cause.

---

# 21. Scalability Analysis

# 21.1 LLM rate limits

The largest immediate bottleneck is provider throughput.

Six agents plus:

- repair turns,
- research,
- critics,
- verifiers,
- revisions

can generate many model calls.

Even though A01/A02 run concurrently, `Retrying` deliberately serializes request pacing per wrapped provider.

At large scale:

```text
N users
 ×
6 stages
 ×
multiple turns
 ×
review fan-out
```

can produce substantial provider pressure.

---

# 21.2 A06 fan-out

A06 creates parallel critics/verifiers.

This is good for latency but increases:

- concurrency,
- provider load,
- memory,
- rate-limit pressure.

A paid/high-throughput provider benefits from it.

A free-tier model may actually run slower because requests become rate-limited.

---

# 21.3 File-backed concurrency

`Run.log()` appends directly to a file.

Multiple threads can call it.

The current implementation relies on ordinary append behavior rather than a dedicated event queue.

For higher concurrency, consider:

```text
agent threads
     |
     v
thread-safe event queue
     |
     v
single writer
     |
     v
events.jsonl
```

---

# 21.4 Shared in-memory `_active` UI state

The FastAPI server uses:

```python
_active: dict[str, str]
```

This is process-local.

It fails as a distributed coordination mechanism.

If multiple UI workers exist:

```text
Worker A knows run X
Worker B does not
```

A database or shared state store would be needed.

---

# 21.5 Polling

The browser polls approximately every 1.5 seconds.

For a small local developer tool this is excellent.

For thousands of clients it becomes wasteful.

At scale:

```text
WebSocket
or
Server-Sent Events
```

could reduce unnecessary polling.

---

# 21.6 LLM context growth

Large architecture and tool outputs can consume huge context windows.

The project mitigates this with:

- truncation,
- history trimming,
- Graphify summaries,
- typed artifacts.

Further improvement could add:

```text
artifact summarization
semantic retrieval
stage-specific context budgets
```

---

# 22. Reliability Risks

## 22.1 LLM hallucination

Mitigations already present:

```text
deterministic graph
file checks
dependency checks
Pydantic validation
sandbox verification
critic/verifier loop
human gates
```

This is one of the strongest aspects of the architecture.

---

# 22.2 Incorrect model capability assumptions

Static capability declarations can become stale.

A future provider/model version may behave differently.

Possible future solution:

```text
static registry
+
optional cached capability probe
```

rather than live probing on every request.

---

# 22.3 Provider compatibility

OpenAI-compatible does not always mean semantically identical.

Providers may differ in:

- JSON schema behavior,
- tool call metadata,
- reasoning fields,
- finish reasons,
- token accounting.

The passthrough metadata design helps, but provider-specific integration tests are still necessary.

---

# 23. Security Analysis

The most important security boundary is the LLM's access to the repository.

Good existing controls include:

- repository-relative file operations,
- path escape prevention,
- read-only shell allowlist,
- restricted Git commands,
- no shell chaining,
- sandbox verification,
- no network in the verification sandbox.

## Remaining risks

The web tools can access external URLs.

The local UI exposes filesystem browsing under the user's home directory.

If the server is bound beyond localhost, this becomes a serious security concern.

Production deployment should add:

```text
authentication
authorization
CSRF protection where applicable
strict bind-address policy
request validation
rate limiting
audit logging
```

---

# 24. Engineering Process Encoded in the Architecture

The project represents a useful software-development philosophy:

```text
1. Understand the existing system.
2. Understand what the user actually wants.
3. Decide technology only after requirements.
4. Measure architectural impact.
5. Ask the human about consequential choices.
6. Produce a precise implementation plan.
7. Verify the plan's assumptions.
8. Attack the plan.
9. Revise confirmed problems.
10. Ask for final approval.
```

This is much closer to senior-level engineering practice than:

```text
prompt -> code
```

---

# 25. Future Roadmap

## Phase 1 — Reproducibility

1. Remove `.venv` from repository archives.
2. Declare Graphify properly.
3. Add UI dependencies as extras.
4. Add Anthropic provider implementation or remove its configuration.
5. Add a documented setup script.
6. Add environment validation.

---

## Phase 2 — Better provider architecture

Introduce:

```text
providers/
    openai_compat.py
    anthropic.py
    gemini.py
    ...
```

with a common adapter contract.

Add provider conformance tests:

```text
provider test suite
    |
    +-- text response
    +-- tool calls
    +-- parallel tools
    +-- schema output
    +-- retries
    +-- metadata passthrough
```

---

## Phase 3 — Distributed execution

Replace local thread-based execution with a worker architecture:

```text
FastAPI
   |
   v
Job Queue
   |
   +--> worker A
   +--> worker B
   +--> worker C
   |
   v
PostgreSQL
```

Possible components:

```text
Redis
Celery/RQ/Arq
PostgreSQL
```

Only introduce these when workload justifies the operational complexity.

---

## Phase 4 — Persistent database

Move:

```text
runs/*.json
events.jsonl
gates/*.json
```

into a database.

Keep artifacts as immutable blobs or versioned records.

Benefits:

- transactional gates,
- concurrent users,
- queryable history,
- permissions,
- distributed workers.

---

## Phase 5 — Better observability

Add:

- OpenTelemetry,
- trace IDs,
- per-agent latency,
- provider latency,
- token cost,
- tool latency,
- retry count,
- failure classification.

A useful trace:

```text
run_id
 |
 +-- a01
 |    +-- provider call
 |    +-- graph build
 |
 +-- a03
 |    +-- provider call
 |    +-- web research
 |
 +-- a06
      +-- critic 1
      +-- critic 2
      +-- verifier 1
      +-- verifier 2
```

---

## Phase 6 — Smarter context management

Introduce an explicit context budget:

```text
stage budget
    |
    +-- architecture: 20%
    +-- requirements: 10%
    +-- tech stack: 15%
    +-- graph: 20%
    +-- plan: 25%
    +-- instructions: 10%
```

Then dynamically summarize/truncate artifacts.

---

# 26. Recommended Improved Architecture

For a production evolution:

```text
                    +----------------+
                    | Browser / CLI  |
                    +-------+--------+
                            |
                            v
                    +---------------+
                    | API / Control |
                    +-------+-------+
                            |
                            v
                    +---------------+
                    | Orchestrator  |
                    +-------+-------+
                            |
                 +----------+----------+
                 |                     |
                 v                     v
          +-------------+       +-------------+
          | Job Queue   |       | PostgreSQL  |
          +------+------+       +-------------+
                 |
        +--------+--------+
        |        |        |
        v        v        v
     Worker   Worker   Worker
        |        |        |
        +--------+--------+
                 |
                 v
        +-------------------+
        | Provider Gateway  |
        +-------------------+
          |       |       |
          v       v       v
       OpenAI  Gemini  Anthropic
                 |
                 v
        +-------------------+
        | Artifact Store    |
        +-------------------+
```

The current project is essentially the **single-process/local prototype of this architecture**.

That is not a criticism. It is a sensible starting point.

---

# 27. How to Learn This Project Properly

Do not read the 4,500 lines sequentially.

Use this order.

## Level 1 — Understand the core abstraction

Read:

```text
providers/base.py
```

Master:

- Message
- ToolCall
- ToolResult
- Provider
- ModelResponse
- ModelCapabilities

---

## Level 2 — Understand agent behavior

Read:

```text
harness/loop.py
harness/tools.py
```

Be able to explain:

```text
LLM -> tool call -> execution -> result -> LLM
```

without looking at the code.

---

## Level 3 — Understand persistence

Read:

```text
orchestrator/run.py
orchestrator/artifacts.py
```

Understand:

```text
Run
  |
  +-- artifacts
  +-- events
  +-- gates
  +-- sessions
```

---

## Level 4 — Understand orchestration

Read:

```text
orchestrator/pipeline.py
```

Understand:

- waves,
- concurrency,
- resume,
- approval,
- revision.

---

## Level 5 — Understand the six agents

Read in this order:

```text
a01
a02
a03
a04
a05
a06
```

For each agent ask:

1. What does it consume?
2. What does it produce?
3. Which decisions are deterministic?
4. Which decisions are delegated to the LLM?
5. What happens if it fails?
6. What gets persisted?
7. Can it resume?

---

## Level 6 — Understand the provider system

Read:

```text
registry.py
openai_compat.py
wrappers.py
prompted_tools.py
```

Then draw:

```text
Provider
  |
  +-- PromptedTools
  |
  +-- Retrying
  |
  +-- Cached
  |
  +-- Metered
```

---

## Level 7 — Understand verification

Finally study:

```text
checks.py
sandbox.py
a06_review.py
```

This is where the project's anti-hallucination philosophy becomes clear.

---

# 28. Senior-Level Questions You Should Be Able to Answer

After mastering the codebase, you should be able to answer:

### Architecture

- Why is `Run` a blackboard rather than a central message bus?
- Why are agents prevented from directly calling one another?
- Why is A01 deterministic before invoking an LLM?
- Why is blast radius partly computed and partly model-supplied?

### Agent design

- Why are tool errors returned as `ToolResult` instead of exceptions?
- Why must every tool call receive a corresponding result?
- Why are tools selectively exposed to different stages?
- Why is the loop bounded?

### Provider design

- Why use a neutral message representation?
- Why preserve unknown metadata?
- Why put `PromptedTools` before `Retrying` and `Cached`?
- Why cache the neutral response rather than provider wire JSON?

### Reliability

- Why does A06 use a verifier?
- Why does `ask_and_check()` perform repair in the same session?
- Why are revision rounds bounded?
- Why are deterministic checks intentionally conservative?

### Scalability

- What breaks first under 1,000 concurrent users?
- What happens when multiple workers write the same run?
- Why is polling acceptable locally but questionable at scale?
- When does PostgreSQL become justified?

If you can answer these from first principles, you understand the architecture rather than merely memorizing files.

---

# 29. Final Architectural Assessment

## Strengths

### 1. Excellent separation of concerns

The project cleanly separates:

```text
agents
orchestration
providers
tools
artifacts
UI
```

### 2. Strong anti-hallucination strategy

It combines:

```text
LLM reasoning
+
AST graph analysis
+
Pydantic validation
+
deterministic checks
+
sandbox probes
+
adversarial verification
+
human approval
```

### 3. Good provider abstraction

The neutral provider model and wrapper composition are particularly strong.

### 4. Good replayability

Disk-backed caching and run artifacts make debugging and experimentation practical.

### 5. Thoughtful human gates

Ambiguous or high-impact choices are surfaced rather than silently assumed.

### 6. Sensible local architecture

For a developer-focused tool, filesystem persistence, polling, threads, and a local FastAPI server are reasonable.

---

## Weaknesses

### 1. Packaging/reproducibility is incomplete

Graphify is operationally required but not declared in `pyproject.toml`.

### 2. Anthropic support is incomplete in the uploaded source

The registry expects an adapter that is not present.

### 3. File-backed state is not a distributed coordination mechanism

It will eventually need a database/queue architecture for multi-user production scale.

### 4. Capability metadata can become stale

Static capability registries trade startup cost for maintenance burden.

### 5. Security is designed for localhost/developer use

It should not be treated as a hardened internet-facing service without additional controls.

---

# 30. One-Sentence Mental Model

If you remember only one thing, remember this:

> **Agentic Workflow is a typed, resumable software-planning pipeline in which specialized LLM agents propose artifacts, deterministic repository analysis measures reality, verifiers attack assumptions, and humans approve consequential decisions.**

That is the architectural essence of the project.
