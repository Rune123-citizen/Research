# Agentic Workflow — Masterclass Technical Report

> **Repository analyzed:** `Agentic_workflow(1).zip`  
> **Project type:** Provider-agnostic multi-agent planning system  
> **Primary language:** Python 3.11+  
> **Architecture style:** Deterministic orchestration + LLM agents + typed artifacts + human-in-the-loop gates  
> **Core idea:** Use multiple specialized AI agents to turn a natural-language feature request into a validated, reviewable implementation plan without directly modifying the target repository.

---

# 1. Executive Overview

## 1.1 What is this project?

**Agentic Workflow** is an AI-powered software-engineering planning system.

Instead of asking one LLM:

> "Add authentication to this repository. Tell me what files to change."

the system decomposes the problem into several specialized stages:

```text
User Feature Request
        │
        ▼
┌──────────────────────┐
│ A01 Codebase Agent   │
│ Understand repo      │
└──────────┬───────────┘
           │
           ├──────────────────┐
           ▼                  ▼
┌──────────────────┐  ┌─────────────────────┐
│ A02 Requirements │  │ Code Graph Analysis │
│ What must happen │  │ What depends on what│
└────────┬─────────┘  └──────────┬──────────┘
         │                       │
         └──────────┬────────────┘
                    ▼
          ┌──────────────────┐
          │ A03 Tech Stack   │
          │ What to reuse/add│
          └────────┬─────────┘
                   ▼
          ┌────────────────────┐
          │ A04 Architecture  │
          │ Change options     │
          │ + blast radius     │
          └─────────┬──────────┘
                    │
                 HUMAN
                 DECISION
                    │
                    ▼
          ┌────────────────────┐
          │ A05 Implementation │
          │ Plan               │
          └─────────┬──────────┘
                    ▼
          ┌────────────────────┐
          │ A06 Review         │
          │ Critic → Verifier │
          └─────────┬──────────┘
                    │
             confirmed issue?
               /          \
             yes           no
              │             │
              ▼             ▼
          revise A05      approve
              │
              └──► A06 again
                         │
                         ▼
                   Human Approval
                         │
                         ▼
                     plan.md
```

The important distinction is:

> **This is not primarily a coding agent. It is a planning and reasoning system.**

It analyzes an existing repository and produces an implementation plan another engineer or coding agent can execute.

---

# 2. The Problem It Solves

Traditional LLM coding workflows have several problems.

### Problem 1 — The LLM doesn't really understand the repository

A model might invent:

```text
src/auth/service.py
```

even though the repository uses:

```text
app/security/authentication.py
```

### Problem 2 — Requirements get silently guessed

A request such as:

> "Add notifications."

could mean:

- email notifications
- browser notifications
- mobile push
- in-app notifications
- all of them
- user-configurable notifications

Guessing wrong contaminates every subsequent decision.

### Problem 3 — LLMs hallucinate dependencies

An LLM might recommend:

```text
Install library X.
```

when the repository already contains an equivalent library.

### Problem 4 — Architectural changes can become unnecessarily large

An LLM often prefers introducing:

```text
new service
new abstraction
new repository
new event bus
new framework
```

instead of extending an existing component.

### Problem 5 — LLM-generated plans aren't necessarily executable

A plan saying:

```text
Update authentication.
Add tests.
Verify everything works.
```

is not an implementation plan.

The project therefore demands:

```text
exact files
exact symbols
exact dependencies
exact changes
runnable checks
blast radius
review points
```

---

# 3. Core Design Philosophy

The project is built around a very important engineering principle:

> **Use LLMs for reasoning and language, but use deterministic code for facts.**

This appears throughout the repository.

For example:

### LLM decides

```text
What architecture option makes sense?
```

### Code determines

```text
How many files does this option touch?
How many dependency communities does it cross?
Does it touch a hub?
```

Similarly:

### LLM critic

```text
"I think this file doesn't exist."
```

### Deterministic checker

```python
(repo / path).exists()
```

This is one of the strongest architectural decisions in the project.

---

# 4. High-Level Architecture

The repository has five major layers.

```text
                         ┌────────────────────┐
                         │      CLI / UI      │
                         └─────────┬──────────┘
                                   │
                                   ▼
                         ┌────────────────────┐
                         │   Orchestrator     │
                         │                    │
                         │ pipeline.py        │
                         │ run.py             │
                         │ checks.py          │
                         │ graph.py           │
                         └─────────┬──────────┘
                                   │
                    ┌──────────────┼──────────────┐
                    ▼              ▼              ▼
               ┌────────┐    ┌──────────┐    ┌──────────┐
               │ Agents │    │ Harness  │    │Artifacts │
               └────┬───┘    └────┬─────┘    └────┬─────┘
                    │             │               │
                    └─────────────┼───────────────┘
                                  ▼
                         ┌──────────────────┐
                         │ Provider Layer   │
                         │ OpenAI-compatible│
                         │ Anthropic        │
                         │ wrappers         │
                         └────────┬─────────┘
                                  │
                  ┌───────────────┼────────────────┐
                  ▼               ▼                ▼
               Gemini           Ollama           vLLM
```

---

# 5. Directory Structure

The extracted repository contains approximately 60 files including tests and configuration.

```text
Agentic_workflow/
│
├── pyproject.toml
├── .gitignore
│
├── config/
│   ├── default.yaml
│   └── ladder.yaml
│
├── agentic_workflow/
│   ├── __init__.py
│   ├── cli.py
│   │
│   ├── agents/
│   │   ├── a01_codebase.py
│   │   ├── a02_requirements.py
│   │   ├── a03_techstack.py
│   │   ├── a04_arch_delta.py
│   │   ├── a05_impl_plan.py
│   │   └── a06_review.py
│   │
│   ├── harness/
│   │   ├── loop.py
│   │   ├── tools.py
│   │   ├── sandbox.py
│   │   ├── structured.py
│   │   └── prompted_tools.py
│   │
│   ├── orchestrator/
│   │   ├── pipeline.py
│   │   ├── run.py
│   │   ├── artifacts.py
│   │   ├── checks.py
│   │   └── graph.py
│   │
│   ├── providers/
│   │   ├── base.py
│   │   ├── registry.py
│   │   ├── openai_compat.py
│   │   └── wrappers.py
│   │
│   └── ui/
│       ├── server.py
│       ├── view.py
│       └── report.py
│
└── tests/
    ├── test_agents.py
    ├── test_pipeline.py
    ├── test_loop.py
    ├── test_sandbox.py
    ├── test_checks.py
    ├── test_review.py
    ├── test_wrappers.py
    ├── test_wire_format.py
    ├── test_context.py
    ├── test_blast_radius.py
    ├── test_prompted_tools.py
    ├── test_new_tools.py
    ├── test_server.py
    ├── test_run.py
    └── test_step_verify.py
```

---

# 6. Architectural Pattern

This project combines several patterns rather than using traditional MVC.

## 6.1 Primary pattern: Pipeline / workflow orchestration

The core execution structure is:

```text
Stage 1
   ↓
Stage 2
   ↓
Stage 3
   ↓
Stage 4
   ↓
Stage 5
   ↓
Stage 6
```

implemented through `pipeline.py`.

---

# 7. Blackboard Architecture

One of the most important concepts in this repository is the **blackboard architecture**.

Agents don't directly communicate with one another.

Instead:

```text
A01 ──► architecture.json
              │
              ▼
A03 ──► techstack.json
              │
              ▼
A04 ──► arch_delta.json
```

The artifacts act as a shared blackboard.

The project explicitly follows:

> Agents read each other's artifacts, never each other.

This gives several advantages.

### Loose coupling

A01 does not need to know anything about A04.

### Persistence

The artifacts exist on disk.

### Resume support

If the program crashes:

```text
architecture.json
requirements.json
techstack.json
```

can remain available.

### Debugging

An engineer can inspect every intermediate decision.

### Testing

Each artifact can be independently validated.

---

# 8. The Six-Agent Pipeline

## A01 — Codebase Understanding

File:

```text
agents/a01_codebase.py
```

Responsibility:

> Determine what the repository currently is.

It does **not** ask the LLM to blindly crawl the repository.

Instead:

```text
Repository
    │
    ▼
graphify
    │
    ▼
AST analysis
    │
    ▼
graph.json
    │
    ▼
GRAPH_REPORT.md
    │
    ▼
LLM
    │
    ▼
Architecture artifact
```

This is an excellent design.

### Output

```json
{
  "summary": "...",
  "languages": [],
  "entry_points": [],
  "components": [],
  "hubs": [],
  "cycles": [],
  "file_count": 0
}
```

---

# 9. Why AST/Graph Analysis?

Suppose a repository has:

```text
500 files
20,000 symbols
100,000 lines
```

Sending everything to an LLM would be:

- expensive
- slow
- context-heavy
- noisy
- prone to hallucination

Instead, graphify extracts structural information.

Conceptually:

```text
A.py ─imports─► B.py
B.py ─calls───► C.py
D.py ─imports─► B.py
```

becomes a dependency graph.

This allows the system to calculate:

```text
node count
edge count
communities
cycles
high-connectivity nodes
```

without using an LLM.

---

# 10. A01 Incremental Reuse

An especially strong feature is:

```python
reuse_prior(...)
```

The architecture artifact can be reused if:

```text
same repository
+
same git commit
```

Therefore:

```text
Run 1
repo @ abc123
      ↓
architecture generated

Run 2
repo @ abc123
      ↓
reuse architecture
```

No expensive model call is required.

But:

```text
Run 1
abc123

Run 2
def456
```

does not reuse it.

This is the correct conservative behavior.

---

# 11. A02 — Requirements Agent

File:

```text
agents/a02_requirements.py
```

Its job is to answer:

> What exactly should the feature do?

It intentionally does **not** discuss:

- Python
- frameworks
- libraries
- APIs
- database schemas
- file names

This separation is extremely important.

---

# 12. Requirements Ambiguity Gate

Suppose the user asks:

```text
Add notifications.
```

The agent may produce:

```text
clear = false
```

with:

```text
approaches:
  - email
  - in-app
  - push
```

The workflow then stops.

```text
A02
 │
 ├── clear
 │      │
 │      ▼
 │   continue
 │
 └── unclear
        │
        ▼
    HUMAN GATE
        │
        ▼
     choice
```

This prevents **silent assumption propagation**.

---

# 13. Why Human Gates Matter

Agentic systems often try to automate everything.

This project deliberately does not.

The system recognizes:

```text
Some decisions are facts.
Some decisions are reasoning.
Some decisions are preferences.
```

For example:

```text
Does file X exist?
```

→ machine-checkable.

But:

```text
Should notifications be email or in-app?
```

→ product decision.

Therefore the human remains responsible for product intent.

---

# 14. A03 — Tech Stack Agent

File:

```text
agents/a03_techstack.py
```

Question:

> What technology should be used to implement the feature?

Its strongest bias is:

```text
REUSE > ADD
```

If the repository already contains:

```text
SQLAlchemy
```

the model should not casually recommend:

```text
Django ORM
```

or another persistence library.

The artifact contains:

```text
existing
additions
notes
```

and every addition has:

```text
name
purpose
why
risk
```

---

# 15. Research Only When Necessary

If:

```text
stack.additions == []
```

external research is unnecessary.

If:

```text
stack.additions != []
```

then the agent researches:

```text
Does the library still exist?
Is it maintained?
What version is current?
Is it deprecated?
Is it already installed?
```

This is an example of **conditional agentic computation**.

---

# 16. A04 — Architecture Delta

File:

```text
agents/a04_arch_delta.py
```

Question:

> How should the existing architecture change?

It creates multiple architectural options.

For example:

```text
Option A:
Extend existing notification service

Option B:
Create new notification subsystem

Option C:
Introduce event-driven notification architecture
```

But it does something particularly important:

> It measures the proposed architecture rather than trusting the model's description of its size.

---

# 17. Blast Radius

The project defines a score similar to:

```text
score =
    files_touched
    + 3 * communities_crossed
    + 5 * hubs_touched
    + 8 * public_api_changed
    + 8 * creates_cycle
```

This is a very useful example of combining:

```text
LLM qualitative reasoning
+
deterministic quantitative measurement
```

---

# 18. Example Blast Radius

Suppose an option:

```text
files touched = 8
communities crossed = 3
hubs touched = 1
public API changed = true
creates cycle = false
```

Then:

```text
8
+ 3 × 3
+ 5 × 1
+ 8 × 1
+ 8 × 0

= 30
```

If:

```text
30 <= 40
```

it remains plannable.

If:

```text
score > 40
```

the system escalates it as:

```text
TooLargeToPlan
```

---

# 19. Why Blast Radius Matters

Without a blast-radius mechanism, an LLM can say:

> "This is a relatively small change."

while proposing:

```text
12 files
4 modules
2 public APIs
new abstraction layer
new dependency
```

The project prevents that by calculating the risk independently.

---

# 20. A05 — Implementation Plan

File:

```text
agents/a05_impl_plan.py
```

This is the main deliverable stage.

The plan must specify:

```text
step number
title
files
dependencies
change
check
review point
blast radius
verification
```

The project's philosophy is:

> A plan should be executable without having to rediscover the reasoning.

---

# 21. Good vs Bad Implementation Plan

Bad:

```text
Add caching to the repository.
```

Good:

```text
FeedRepository.__init__ gains cache: CacheStore as a third
parameter.

In load(key), before self._client.get(...), perform:

hit = self.cache.get(key) if self.cache else None

Return hit when non-null.

On cache miss, retain the existing network call and call
self.cache.set(key, result) before returning.
```

The second version is operationally useful.

---

# 22. Step Dependency Graph

Every implementation step can declare:

```json
{
  "n": 3,
  "depends_on": [1, 2]
}
```

This creates a DAG:

```text
Step 1
   │
   ├────► Step 2
   │          │
   └─────────► Step 3
                  │
                  ▼
               Step 4
```

The deterministic checker verifies:

```text
dependency exists
dependency is earlier
no invalid forward dependency
```

---

# 23. Every Step Must Have a Check

This is another major engineering principle.

Bad:

```text
check: "Verify it works"
```

Good:

```text
pytest tests/test_auth.py
```

or:

```text
python -c "from package import function; assert ..."
```

The system checks for vague checks.

Therefore the plan becomes:

```text
Change
+
Proof
```

rather than just:

```text
Change
```

---

# 24. Sandbox Verification

A particularly interesting component is:

```text
harness/sandbox.py
```

The system can create a copy of the repository and test assumptions about an implementation step.

Flow:

```text
Original repo
     │
     ▼
temporary copy
     │
     ▼
inject verification program
     │
     ▼
Bubblewrap
     │
     ├── network disabled
     ├── host read-only
     ├── CPU limit
     ├── file-size limit
     ├── timeout
     └── isolated process/session
     │
     ▼
run probe
     │
     ▼
PASS / FAIL
```

---

# 25. Why Verify Assumptions Instead of Implementing?

Suppose the plan says:

```text
Call function:

Repository.load(key, timeout=5)
```

But the actual function is:

```python
Repository.load(key)
```

A normal coding agent might discover this only after editing.

This project instead creates a probe:

```text
Does Repository.load accept timeout?
```

If the probe fails:

```text
actual evidence
      ↓
rewrite plan
      ↓
verify again
```

This is much safer.

---

# 26. A06 — Review Agent

File:

```text
agents/a06_review.py
```

This stage attacks the plan.

The review intentionally uses:

```text
Critic
   ↓
Verifier
```

rather than trusting one model.

---

# 27. Critic → Verifier Architecture

```text
                   Implementation Plan
                          │
            ┌─────────────┴─────────────┐
            ▼                           ▼
    Architecture-fit              Testability
       critic                       critic
            │                           │
            └─────────────┬─────────────┘
                          ▼
                       findings
                          │
                ┌─────────┴─────────┐
                ▼                   ▼
             verifier            verifier
                │                   │
                ▼                   ▼
             evidence            evidence
                │                   │
                └─────────┬─────────┘
                          ▼
                  CONFIRMED / REJECTED
```

---

# 28. Why Two-Pass Review?

An LLM critic may say:

> `tests/test_auth.py` does not exist.

But the plan may contain:

```text
Step 2 creates tests/test_auth.py
```

Therefore the critic is wrong.

The verifier gets repository tools and can determine reality.

This is an important agentic principle:

> **A criticism is not a fact until independently verified.**

---

# 29. Review Dimensions

The active model-based dimensions are:

### Architecture fit

Does the plan match the existing architecture?

### Testability

Are the proposed checks actually runnable and meaningful?

Other dimensions are deliberately deterministic:

```text
file references
sequencing
blast radius
```

because the system has better non-LLM mechanisms for them.

---

# 30. Finding State Machine

Every finding can be:

```text
UNVERIFIED
CONFIRMED
REJECTED
```

The semantics are important.

### CONFIRMED

Evidence proves the criticism.

### REJECTED

Evidence disproves it.

### UNVERIFIED

The system couldn't establish the truth.

Crucially:

```text
UNVERIFIED != CONFIRMED
```

A verifier crash cannot accidentally turn into a plan-changing finding.

---

# 31. Bounded Review Loop

The project intentionally avoids:

```python
while findings:
    revise()
    review()
```

because an LLM can oscillate forever.

Instead:

```text
maximum rounds = 3
```

and it also detects repeated findings.

Conceptually:

```text
Review
  ↓
Finding A
  ↓
Revision
  ↓
Review
  ↓
Finding A again
  ↓
No progress
  ↓
STOP
```

This is a critical production-agent design principle.

---

# 32. Orchestrator

The orchestrator consists primarily of:

```text
pipeline.py
run.py
artifacts.py
checks.py
graph.py
```

---

# 33. `pipeline.py`

The workflow is represented as waves:

```python
WAVES = [
    [a01, a02],
    [a03],
    [a04],
    [a05],
    [a06],
]
```

This means:

```text
Wave 1:
A01 ─┐
     ├── parallel
A02 ─┘

Wave 2:
A03

Wave 3:
A04

Wave 4:
A05

Wave 5:
A06
```

A01 and A02 can run concurrently because they have independent inputs.

---

# 34. Why Parallelism Is Limited

The architecture doesn't blindly parallelize everything.

For example:

```text
A04 requires:
architecture
requirements
techstack
```

Therefore A04 must wait.

Likewise:

```text
A05 requires A04.
```

This is dependency-aware concurrency.

---

# 35. `run.py` — Persistent Run State

Every run receives a directory like:

```text
runs/
└── 20260910-005500-ab12/
    ├── run.json
    ├── events.jsonl
    ├── architecture.json
    ├── requirements.json
    ├── techstack.json
    ├── arch_delta.json
    ├── impl_plan.json
    ├── review.json
    ├── transcript.jsonl
    ├── plan.md
    └── gates/
```

This is effectively a lightweight workflow database.

---

# 36. Why Files Instead of a Database?

For this project's scale, files are a reasonable choice.

Advantages:

- zero database setup
- human-readable
- easy debugging
- easy backup
- naturally persistent
- easy replay
- easy testing

Trade-offs:

- poor concurrent write semantics at scale
- no transactional guarantees
- difficult querying
- unsuitable for many thousands of simultaneous runs

For a local developer tool, filesystem persistence is sensible.

For SaaS-scale deployment, it should probably move toward:

```text
PostgreSQL
+
object storage
+
event store
```

---

# 37. Event Log

`events.jsonl` is append-only.

Example conceptually:

```json
{"kind":"stage_start","stage":"a01"}
{"kind":"graph_built","stage":"a01"}
{"kind":"response","stage":"a01"}
{"kind":"artifact","name":"architecture"}
{"kind":"stage_end","stage":"a01"}
```

This provides observability and replay context.

---

# 38. Event Sourcing Concepts

This isn't full event sourcing, but it borrows an important idea:

> State changes leave an immutable-ish history.

The UI reconstructs run state from:

```text
run.json
+
events.jsonl
+
artifact files
```

This is significantly easier to debug than keeping only:

```text
current_state.json
```

---

# 39. Human Gate Mechanism

A gate is represented by:

```text
gates/
    approach.json
    approach.answer.json
```

The workflow does:

```python
run.gate(...)
```

If no answer exists:

```text
GateOpen
```

is raised.

The process stops.

The UI or CLI writes:

```json
{
  "choice": "option-a",
  "notes": "Prefer minimal change."
}
```

Then the workflow resumes.

---

# 40. Why This Is a Good Decoupling

The orchestrator doesn't know whether the answer came from:

```text
CLI
browser
TUI
future mobile app
API
```

All it knows is:

```text
answer file exists
```

Therefore:

```text
Orchestrator
      │
      ▼
   gate contract
      │
      ├── CLI
      ├── Web UI
      └── future UI
```

This is a clean interface boundary.

---

# 41. Deterministic Checks

`checks.py` contains the project's guardrails.

Examples:

```python
(repo / path).exists()
```

```text
Does dependency exist?
```

```text
Is dependency earlier?
```

```text
Is a check actually runnable?
```

```text
Does clear=True have requirements?
```

This is the **anti-hallucination layer**.

---

# 42. Provider Abstraction

The provider architecture is one of the most important parts.

`providers/base.py` defines a neutral model representation.

```text
              Provider Protocol
                     │
        ┌────────────┼────────────┐
        ▼            ▼            ▼
 OpenAI-compatible Anthropic   future provider
        │
        ▼
 Gemini / Ollama / vLLM / etc.
```

Agents don't care which provider they're using.

---

# 43. Why a Neutral Representation?

Different LLM APIs disagree on:

- message structure
- tool calls
- tool results
- finish reasons
- JSON schema
- metadata
- hidden provider state

The project normalizes them.

Core abstractions include:

```python
Message
TextBlock
ToolCall
ToolResult
ToolSpec
ModelResponse
Usage
ModelCapabilities
StopReason
```

---

# 44. Tool Call Representation

Instead of provider-specific structures:

```python
ToolCall(
    id="abc",
    name="read_file",
    arguments={"path": "main.py"}
)
```

the rest of the application uses that representation.

The provider adapter converts it to the appropriate wire format.

This is the **Adapter Pattern**.

---

# 45. OpenAI-Compatible Adapter

File:

```text
providers/openai_compat.py
```

This supports services exposing an OpenAI-style endpoint.

Conceptually:

```text
Agent
 │
 ▼
neutral Message
 │
 ▼
OpenAICompatProvider
 │
 ▼
HTTP POST /chat/completions
 │
 ▼
provider
```

This allows OpenAI-compatible services such as local inference servers and hosted gateways to use the same provider implementation.

---

# 46. Wire Format Translation

Internally:

```text
ToolResult(call_id="a", ...)
ToolResult(call_id="b", ...)
```

can be represented as one message.

OpenAI-compatible APIs may require:

```text
role=tool, tool_call_id=a
role=tool, tool_call_id=b
```

The adapter performs that transformation.

Agents don't need to care.

---

# 47. Provider Capability Registry

`registry.py` contains capabilities such as:

```text
context_window
max_output
supports_tools
supports_parallel_tools
supports_json_schema
supports_caching
cost_in
cost_out
```

This is critical.

The system does not assume:

> "Every OpenAI-compatible model behaves exactly like OpenAI."

Instead it explicitly models capabilities.

---

# 48. Capability Ladder

One of the project's strongest concepts is the **degradation ladder**.

Suppose a model supports:

```text
native tools
native JSON schema
```

Great.

But suppose a local model doesn't support tools.

The registry detects:

```text
supports_tools = false
```

and automatically wraps it with:

```text
PromptedTools
```

So the architecture becomes:

```text
Native tool model
      │
      ▼
normal tool calling


No-tool model
      │
      ▼
PromptedTools
      │
      ▼
tool calls encoded in text
      │
      ▼
same agent loop
```

This is portability through graceful degradation.

---

# 49. Why Conservative Defaults?

Unknown models default to:

```text
supports_tools = false
supports_json_schema = false
```

This is safer than assuming support.

Why?

Because:

```text
false positive capability
```

can cause a runtime API failure.

Whereas:

```text
false negative capability
```

usually causes a slower but functioning fallback.

That is a very good production engineering trade-off.

---

# 50. Provider Wrappers

`providers/wrappers.py` contains major infrastructure wrappers.

## 50.1 Cached

Records provider calls to disk.

```text
Agent
 ↓
Cache
 ├── hit → return recorded response
 └── miss → provider → store response
```

This enables:

- replay
- offline testing
- cheap development
- deterministic debugging

---

# 51. Replay Mode

The CLI supports:

```text
--replay
```

In replay mode:

```text
cache miss
    ↓
ReplayMiss
```

The system does not accidentally make an external model call.

This is particularly valuable for tests and expensive agent workflows.

---

# 52. Retrying Wrapper

The `Retrying` wrapper handles transient statuses such as:

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

with retries.

It also respects:

```text
Retry-After
```

when provided.

This is much better than blindly using fixed sleep intervals.

---

# 53. Rate Limiting

The wrapper also supports:

```text
min_interval
```

For example:

```text
RPM = 10
```

means approximately:

```text
60 / 10 = 6 seconds/request
```

This matters because multiple agents can otherwise overwhelm a free-tier provider.

---

# 54. Metered Wrapper

The `Metered` wrapper records:

```text
input tokens
output tokens
cached tokens
latency
model
stage
tool calls
```

This is an excellent cross-cutting-concern implementation.

Instead of every agent doing:

```python
log_tokens(...)
```

the provider wrapper does it centrally.

---

# 55. Why Wrappers Are Better Than Modifying Every Agent

This follows the **Decorator Pattern**.

Conceptually:

```text
Metered(
    Cached(
        Retrying(
            PromptedTools(
                OpenAIProvider
            )
        )
    )
)
```

Each layer adds one responsibility.

This is significantly cleaner than creating one giant provider class.

---

# 56. Agent Harness

The harness is the mechanism that transforms:

```text
LLM completion API
```

into:

```text
agent
```

The most important file is:

```text
harness/loop.py
```

---

# 57. The Agent Loop

The loop is essentially:

```text
send prompt
   ↓
LLM
   ↓
tool calls?
 ┌─┴─────────┐
no           yes
│             │
▼             ▼
finish     execute tools
              │
              ▼
          append results
              │
              ▼
             LLM
              │
              └──────► ...
```

This is the fundamental agentic loop.

---

# 58. Why Tool Results Are Returned Together

If the model produces:

```text
call A
call B
call C
```

the system executes them and puts the results into the following turn.

This preserves the model's mental structure:

```text
assistant:
I need A, B, C

tool:
result A

tool:
result B

tool:
result C
```

rather than splitting the calls across unrelated turns.

---

# 59. Parallel Tool Execution

Tools are only parallelized if:

```text
supports_parallel_tools
```

and:

```text
all tools are parallel_safe
```

This prevents unsafe operations from running concurrently.

For example:

```text
read_file + grep
```

can potentially execute concurrently.

But:

```text
write operation A
write operation B
```

should not automatically run in parallel.

---

# 60. Tool Safety

The toolbox provides:

```text
read_file
grep
glob
list_dir
```

and optional:

```text
web_search
web_fetch
bash
graph_query
```

---

# 61. Path Confinement

The toolbox resolves paths relative to a root:

```python
candidate = (root / raw).resolve()
```

Then checks that the candidate remains under the root.

Therefore:

```text
../../etc/passwd
```

is rejected.

This prevents a repository agent from escaping its assigned repository.

---

# 62. Tool Result Limits

Tool outputs are capped at approximately:

```text
20,000 characters
```

and matches at:

```text
200
```

This is **context-window protection**.

Without limits, a broad grep could return tens of thousands of lines and destroy the model's useful context.

---

# 63. Context Trimming

`loop.py` includes history trimming.

When history becomes too large:

```text
old tool result
```

is replaced by:

```text
[trimmed: an earlier tool result, dropped to free context]
```

The tool call itself remains, so the model still knows that a previous operation occurred without carrying its full output forever.

---

# 64. Structured Output

`harness/structured.py` converts model output into Pydantic objects.

Flow:

```text
LLM response
     │
     ▼
extract JSON
     │
     ▼
Pydantic validation
     │
   valid?
  /     \
yes      no
│         │
▼         ▼
return   repair prompt
            │
            ▼
           LLM
```

There is a bounded repair attempt.

---

# 65. Why Pydantic?

Pydantic is used for:

### Validation

```python
Requirements.model_validate_json(...)
```

### Schema generation

```python
model.model_json_schema()
```

The generated schema can also be passed to providers supporting structured output.

This avoids maintaining two separate schema systems.

---

# 66. Prompted Tools

For models without native function calling, `prompted_tools.py` provides a fallback.

Conceptually:

```text
tool definitions
      ↓
prompt
      ↓
LLM writes structured tool call
      ↓
parser extracts call
      ↓
Toolbox executes it
```

Therefore the same agent loop can work with weaker/local models.

---

# 67. Sandbox Security Model

The sandbox uses Linux Bubblewrap (`bwrap`).

The design includes:

```text
host root filesystem → read-only
repository copy → writable
network → disabled
PID namespace → isolated
CPU → limited
file size → limited
execution → timeout
```

This is much safer than simply executing generated code against the real repository.

---

# 68. Important Security Property

The actual repository is never modified during step verification.

Instead:

```text
real repo
   │
   ▼
temporary copy
   │
   ▼
probe
```

This is the correct model for AI-generated verification programs.

---

# 69. UI Architecture

The UI consists of:

```text
ui/server.py
ui/view.py
ui/report.py
```

The web server uses:

```text
FastAPI
```

and:

```text
Uvicorn
```

---

# 70. Why Polling Instead of WebSockets/SSE?

The project intentionally uses polling:

```text
browser
   │
   ├── GET /api/run/id
   ├── wait
   ├── GET /api/run/id
   └── ...
```

rather than maintaining another event-delivery infrastructure.

This works because the actual source of truth is already:

```text
events.jsonl
artifact files
```

Advantages:

- simpler
- survives server restart
- fewer moving parts
- easy debugging

Trade-offs:

- more HTTP requests
- less real-time
- poor scaling to thousands of browsers

For a local tool, polling is reasonable.

---

# 71. UI Data Flow

```text
events.jsonl
     │
     ▼
ui.view.load()
     │
     ▼
RunView
     │
     ├── CLI/report representation
     │
     └── web JSON representation
```

The important design decision is:

> The UI is a projection of run state, not another state machine.

---

# 72. HTML Reports

`report.py` generates a self-contained HTML file.

No React, Webpack, Vite, or external CDN is required.

The report contains:

```text
stages
decisions
review findings
implementation plan
token usage
cost information
```

This is useful for sharing an analysis artifact.

---

# 73. CLI

`cli.py` exposes commands conceptually equivalent to:

```text
plan-mode ask
plan-mode run
plan-mode resume
plan-mode answer
plan-mode ui
plan-mode report
```

### `ask`

Single-agent repository question.

```bash
plan-mode ask /path/to/repo "Where is authentication handled?"
```

### `run`

Full pipeline:

```bash
plan-mode run /path/to/repo "Add OAuth login"
```

### `resume`

Continue a paused workflow:

```bash
plan-mode resume RUN_ID
```

### `answer`

Answer a gate:

```bash
plan-mode answer RUN_ID approach option-a
```

### `ui`

Start local web UI:

```bash
plan-mode ui
```

### `report`

Generate HTML:

```bash
plan-mode report RUN_ID
```

---

# 74. Technology Stack

## Python

Primary implementation language.

Why?

- excellent filesystem tooling
- mature HTTP ecosystem
- Pydantic
- subprocess/sandbox integration
- concurrency support
- strong AI/ML ecosystem
- rapid AI-agent development

TypeScript would be attractive for a web-heavy platform, but Python is well suited to repository analysis and AI infrastructure.

---

# 75. Pydantic

Used for:

```text
typed artifacts
validation
JSON schema
```

Why Pydantic over raw dictionaries?

Without Pydantic:

```python
artifact["requirements"]
artifact["clear"]
```

can silently fail.

With Pydantic, the artifact itself becomes a contract.

---

# 76. NetworkX

Used for graph analysis.

This is appropriate because graph concepts are central:

```text
nodes
edges
connectivity
communities
cycles
```

A custom graph implementation could be faster for a narrow use case, but NetworkX dramatically reduces development complexity.

---

# 77. HTTPX

Used for HTTP communication.

Advantages:

- modern Python API
- timeout handling
- exception model
- headers
- redirects
- clean client interface

For this primarily sequential/small-concurrency workflow, synchronous HTTPX is sufficient.

---

# 78. PyYAML

Used for human-maintained configuration.

YAML is appropriate for nested provider and agent configuration.

---

# 79. FastAPI + Uvicorn

Used for the local browser UI/API.

FastAPI provides:

```text
routing
request parsing
JSON responses
HTTP error handling
```

Uvicorn provides the ASGI server.

---

# 80. Rich

Used by the CLI for:

```text
panels
formatted output
status
developer-friendly logs
```

---

# 81. JSONL

Used for:

```text
events.jsonl
transcript.jsonl
```

JSON Lines is well suited to append-only logs because every line is independently parseable.

---

# 82. Pyproject Configuration

The declared core dependencies include:

```text
httpx
pydantic
pyyaml
jsonschema
rich
networkx
```

Python requirement:

```text
>= 3.11
```

Build backend:

```text
hatchling
```

CLI entry point:

```text
plan-mode = agentic_workflow.cli:main
```

---

# 83. Repository Issue: `graphify`

The code calls an external command resembling:

```python
subprocess.run(
    ["graphify", "update", ...]
)
```

but `graphify` is not declared in the Python dependencies.

Therefore this is an external runtime prerequisite.

It should be:

1. declared/documented explicitly,
2. installed by a setup script, or
3. packaged as a proper dependency if appropriate.

This is one of the first things I would fix.

---

# 84. Repository Issue: Anthropic Provider

The provider registry references an Anthropic implementation, while the uploaded project does not appear to contain the corresponding provider module.

Therefore the Anthropic configuration should be treated as incomplete until the adapter is implemented or the configuration is corrected.

The default configuration may still work if another provider is selected.

---

# 85. Configuration

`default.yaml` contains provider configuration for providers such as:

```text
gemini
gemini-notools
vllm
ollama
claude
```

Conceptually:

```text
                 provider registry
                        │
          ┌─────────────┼─────────────┐
          ▼             ▼             ▼
       Gemini         Ollama         vLLM
          │             │             │
       hosted         local          local
```

---

# 86. Environment Variables

Credentials are referenced through environment variables such as:

```text
${GEMINI_API_KEY}
${ANTHROPIC_API_KEY}
```

The registry should refuse to silently substitute missing variables.

Conceptually:

```text
missing credential
      ↓
clear configuration error
```

rather than allowing an empty authorization header.

---

# 87. Complete End-to-End Example

Suppose the user asks:

```text
Add a search filter to the application's user list.
```

### Step 1 — A01

The repository is analyzed.

Graph extraction determines:

```text
backend/
frontend/
user module
API module
database layer
```

Architecture artifact is written.

### Step 2 — A02

Requirements agent determines:

```text
User can filter users by name.
User can clear the filter.
Empty results are shown clearly.
```

### Step 3 — A03

It discovers existing frontend and backend capabilities and prefers reuse.

### Step 4 — A04

Possible options:

```text
A: Filter in browser
B: Filter through existing API
C: Create new search subsystem
```

Graph analysis calculates different blast-radius scores.

### Step 5 — A05

The selected architecture becomes an implementation plan with:

```text
files
dependencies
changes
checks
review points
```

### Step 6 — A06

Critics inspect the plan, verifiers test factual claims, and confirmed findings cause revision.

### Step 7 — Human Approval

Once review passes, the workflow produces:

```text
plan.md
```

No production code is modified.

---

# 88. Scalability Analysis

The project is well designed for a **local developer tool**, but substantial changes would be needed for a multi-tenant production service.

## Bottleneck 1 — Filesystem State

Current state is stored in run directories.

At high concurrency, thousands of file operations and concurrent writes become difficult to manage.

### Production direction

```text
PostgreSQL
+
object storage
+
event stream
```

---

# 89. Bottleneck 2 — Provider Rate Limits

Six agents can create many LLM calls, especially the review stage.

Current mitigation:

```text
Retrying
+
min_interval
+
cache
```

Production evolution:

```text
central rate limiter
provider quotas
job queue
adaptive concurrency
```

---

# 90. Bottleneck 3 — A06 Fan-Out

Multiple critics/verifiers can execute concurrently.

This improves latency but increases provider pressure.

A centralized scheduler would provide better control at scale.

---

# 91. Bottleneck 4 — Graph Construction

Repository graph extraction can become expensive for large repositories.

Potential improvement:

```text
persistent graph cache
incremental updates
background indexing
```

---

# 92. Bottleneck 5 — Context Windows

Large repositories generate:

```text
large graph reports
large artifacts
large tool histories
```

The project mitigates this with:

```text
graph summarization
tool-output limits
history trimming
capability-aware contexts
```

But context management remains a fundamental constraint of agentic systems.

---

# 93. Bottleneck 6 — Synchronous HTTP

Provider calls are synchronous.

Threads provide some concurrency, but a very large deployment could benefit from:

```text
asyncio
httpx.AsyncClient
async workers
task queues
```

The current approach is reasonable for a local developer workflow.

---

# 94. Security Limitations

The sandbox is strong for local verification but should not automatically be treated as a complete hostile-code execution environment.

Potential concerns include:

```text
kernel vulnerabilities
container/runtime vulnerabilities
resource exhaustion
symlink edge cases
host configuration
privilege configuration
```

For highly hostile workloads, stronger isolation such as microVMs would be preferable.

---

# 95. Current Architecture Strengths

```text
✓ specialized agents
✓ blackboard artifacts
✓ Pydantic contracts
✓ provider abstraction
✓ capability negotiation
✓ prompted-tool fallback
✓ deterministic checks
✓ graph-based blast-radius measurement
✓ sandbox verification
✓ critic/verifier separation
✓ human gates
✓ replay/cache
✓ bounded loops
✓ persistent event history
```

---

# 96. Current Architecture Weaknesses

```text
⚠ graphify dependency packaging
⚠ incomplete Anthropic adapter/configuration
⚠ filesystem persistence at scale
⚠ centralized concurrency/rate management
⚠ production-grade sandbox isolation
⚠ authentication for non-local deployment
⚠ distributed workflow execution
⚠ stronger artifact/version management
```

---

# 97. Testing Strategy

The repository has a strong test suite for a project of this size.

There are tests covering:

```text
agent behavior
pipeline behavior
provider behavior
wire serialization
sandbox isolation
context trimming
blast radius
human gates
web UI
tool security
replay
retry
structured output
```

This indicates that the project tests **behavioral contracts**, not merely individual functions.

---

# 98. Particularly Important Tests

## `test_blast_radius.py`

Protects the invariant that the model cannot understate structural change.

## `test_sandbox.py`

Tests isolation properties such as:

```text
network unavailable
real repo unchanged
host filesystem protection
timeouts
copy isolation
write confinement
```

## `test_loop.py`

Tests:

```text
tool round trips
parallel tool calls
tool failures
unknown tools
loop limits
usage accumulation
path escape
```

## `test_review.py`

Tests:

```text
confirmed findings
rejected findings
unverified findings
revision
bounded review loops
```

---

# 99. Engineering Principles Demonstrated

## Principle 1

**Don't ask an LLM to determine something deterministic.**

Bad:

```text
"Does this file exist?"
```

Better:

```python
Path.exists()
```

## Principle 2

**Don't force every model to support the same capabilities.**

Use capability negotiation and fallbacks.

## Principle 3

**Don't trust generated output until it crosses a validation boundary.**

```text
LLM
 ↓
schema
 ↓
validated object
```

## Principle 4

**Don't let agents run forever.**

Use:

```text
max_turns
max_rounds
timeouts
retry limits
```

## Principle 5

**Human decisions should remain explicit.**

Don't hide product ambiguity behind an LLM guess.

## Principle 6

**Persist intermediate state.**

A multi-step AI workflow without persistence is extremely difficult to debug.

---

# 100. Recommended Setup

From WSL/Linux:

```bash
cd Agentic_workflow
```

Create environment:

```bash
python3 -m venv .venv
source .venv/bin/activate
```

Install:

```bash
pip install -e .
```

Set required credentials, for example:

```bash
export GEMINI_API_KEY="[YOUR_GEMINI_API_KEY]"
```

Verify the external graph command:

```bash
which graphify
```

Run tests:

```bash
pytest
```

---

# 101. First Test I Recommend

Before running the full workflow:

```bash
plan-mode ask /path/to/test/repository "Where is the main entry point?"
```

This validates:

```text
CLI
 ↓
Config
 ↓
Provider
 ↓
Agent loop
 ↓
Toolbox
 ↓
LLM
 ↓
Answer
```

Only after this succeeds should you run the complete pipeline.

---

# 102. Full Workflow

Example:

```bash
plan-mode run /path/to/repository \
  "Add a search filter to the user management page"
```

Expected conceptual flow:

```text
A01
 ↓
A02
 ↓
A03
 ↓
A04
 ↓
human choice
 ↓
A05
 ↓
A06
 ↓
human approval
 ↓
plan.md
```

---

# 103. Running the Web UI

```bash
plan-mode ui
```

The UI is intended for local workflow interaction and allows users to:

```text
select repository
start run
watch stages
inspect transcripts
answer gates
send steering notes
resume runs
open reports
```

---

# 104. Suggested Learning Order

For mastery, don't read the files alphabetically.

Use this order:

```text
1. artifacts.py
2. run.py
3. pipeline.py
4. base.py
5. loop.py
6. tools.py
7. structured.py
8. registry.py
9. wrappers.py
10. a01 → a06
11. graph.py
12. checks.py
13. sandbox.py
14. CLI
15. UI
16. tests
```

This follows the system's conceptual dependencies.

---

# 105. What to Understand at Each Phase

## Phase 1 — Artifacts

Understand:

```text
What information moves through the pipeline?
```

## Phase 2 — Run State

Understand:

```text
How does the workflow survive interruption?
```

## Phase 3 — Pipeline

Understand:

```text
Who runs when?
Why are A01/A02 parallel?
Why is A06 last?
```

## Phase 4 — Provider

Understand:

```text
How can different LLM APIs look identical to agents?
```

## Phase 5 — Agent Harness

Understand:

```text
How does a simple LLM API become an autonomous tool-using loop?
```

## Phase 6 — Agents

Understand:

```text
What reasoning responsibility belongs to each agent?
```

## Phase 7 — Verification

Understand:

```text
How does the system prevent LLM reasoning from becoming unchecked fact?
```

---

# 106. Future Roadmap

## Phase 1 — Fix Repository Completeness

Immediately address:

```text
graphify dependency/documentation
Anthropic provider implementation
installation documentation
```

Add a reproducible setup process.

---

# 107. Phase 2 — Stronger Artifact Versioning

Add metadata such as:

```text
schema_version
stage_version
model
git_commit
timestamp
```

This makes long-lived runs safer.

---

# 108. Phase 3 — Database Backend

For production:

```text
PostgreSQL
```

could store:

```text
runs
stages
artifacts
events
gates
provider calls
review findings
```

Object storage could hold large:

```text
transcripts
graph files
reports
```

---

# 109. Phase 4 — Distributed Workers

A scalable architecture could be:

```text
                    API
                     │
                     ▼
                  Queue
                     │
          ┌──────────┼──────────┐
          ▼          ▼          ▼
       Worker 1   Worker 2   Worker 3
          │          │          │
          └──────────┼──────────┘
                     ▼
                 PostgreSQL
```

Possible technologies:

```text
Redis
Celery
RQ
Temporal
Kafka
```

Temporal is particularly interesting because the workflow already has durable stages, retries, human gates, and resumability.

---

# 110. Phase 5 — Better Model Routing

Instead of one model for everything:

```text
A01 → cheap model
A02 → cheap model
A03 → cheap model
A04 → strong reasoning model
A05 → strong implementation/planning model
A06 → independent strong critic
```

This can reduce cost while improving quality.

---

# 111. Phase 6 — Independent Review Models

A future architecture could use independent models for different roles:

```text
Planner model
      ↓
Critic model
      ↓
Verifier model
      ↓
Final judge
```

This reduces correlated model failures.

---

# 112. Phase 7 — Repository-Specific Memory

The system could remember:

```text
repository architecture
coding conventions
past feature decisions
previous plans
known risky modules
```

Future runs could then start from an existing repository knowledge base.

A vector database should only be added if semantic retrieval provides measurable value; structural graph data should remain the source of truth for repository topology.

---

# 113. Phase 8 — Stronger Graph Intelligence

Blast radius could incorporate:

```text
weighted dependency edges
runtime dependencies
API consumers
test coverage
change frequency
git history
ownership
criticality
```

For example:

```text
blast_score =
structural_coupling
+
API_risk
+
change_frequency
+
runtime_criticality
+
test_coverage_penalty
```

This would provide a more realistic engineering risk model.

---

# 114. Phase 9 — Actual Coding Agent Integration

The natural extension is:

```text
Agentic Workflow
       │
       ▼
implementation plan
       │
       ▼
coding agent
       │
       ▼
code changes
       │
       ▼
tests
       │
       ▼
review agent
       │
       ▼
human approval
```

This would transform the system from a planning tool into a broader AI software-engineering workflow platform.

---

# 115. Ideal Future Architecture

```text
                         ┌───────────────┐
                         │ Web / CLI / API│
                         └───────┬───────┘
                                 │
                                 ▼
                         ┌───────────────┐
                         │ Workflow API  │
                         └───────┬───────┘
                                 │
                                 ▼
                         ┌───────────────┐
                         │ Workflow Engine│
                         │  Temporal/etc │
                         └───────┬───────┘
                                 │
                    ┌────────────┼────────────┐
                    ▼            ▼            ▼
                 A01-A02      A03-A04      A05-A06
                    │            │            │
                    └────────────┼────────────┘
                                 ▼
                         ┌───────────────┐
                         │ Provider Router│
                         └───────┬───────┘
                                 │
              ┌──────────────────┼──────────────────┐
              ▼                  ▼                  ▼
           OpenAI             Anthropic           Local
              │                  │                  │
              └──────────────────┼──────────────────┘
                                 ▼
                         ┌───────────────┐
                         │ PostgreSQL    │
                         └───────────────┘
                                 │
                    ┌────────────┴────────────┐
                    ▼                         ▼
              Object Storage             Event Stream
```

---

# 116. The Most Important Concept to Master

If you remember only one thing from this project, remember this:

```text
                LLM
                 │
          probabilistic reasoning
                 │
                 ▼
        ┌─────────────────┐
        │ typed artifact  │
        └────────┬────────┘
                 │
                 ▼
        deterministic check
                 │
          ┌──────┴──────┐
          ▼             ▼
        valid         invalid
          │             │
          ▼             ▼
       continue       repair
```

This is the fundamental architecture.

The system is **not** saying:

> "LLMs are reliable."

It is saying:

> "LLMs are useful reasoning engines, so surround them with deterministic contracts, validation, persistence, bounded execution, and human decision points."

That is the senior-level lesson behind the entire codebase.

---

# 117. Final Architectural Assessment

### Overall design quality

**Strong for a local AI engineering-planning tool.**

Its strongest decisions are:

```text
✓ specialized agents
✓ blackboard artifacts
✓ Pydantic contracts
✓ provider abstraction
✓ capability negotiation
✓ prompted-tool fallback
✓ deterministic checks
✓ graph-based blast-radius measurement
✓ sandbox verification
✓ critic/verifier separation
✓ human gates
✓ replay/cache
✓ bounded loops
✓ persistent event history
```

The most important improvements are:

```text
⚠ graphify dependency packaging
⚠ missing/incomplete Anthropic adapter
⚠ filesystem persistence at scale
⚠ centralized concurrency/rate management
⚠ production-grade sandbox isolation
⚠ authentication for non-local deployment
⚠ distributed workflow execution
⚠ stronger artifact/version management
```

---

# 118. Master Mental Model

You can understand the entire repository with this single model:

```text
                    USER
                     │
                     │ feature request
                     ▼
              ┌───────────────┐
              │ A01 + A02     │
              │ understand    │
              │ reality + need│
              └───────┬───────┘
                      │
                      ▼
              ┌───────────────┐
              │ A03           │
              │ technology    │
              └───────┬───────┘
                      │
                      ▼
              ┌───────────────┐
              │ A04           │
              │ architecture  │
              │ + graph math  │
              └───────┬───────┘
                      │
                  HUMAN GATE
                      │
                      ▼
              ┌───────────────┐
              │ A05           │
              │ implementation│
              │ plan          │
              └───────┬───────┘
                      │
                      ▼
              ┌───────────────┐
              │ A06           │
              │ critic        │
              │     ↓         │
              │ verifier      │
              └───────┬───────┘
                      │
                confirmed issue?
                 /          \
               yes           no
                │             │
                ▼             ▼
             revise         approve
                │             │
                └──────┐      │
                       ▼      ▼
                       A06   plan.md
                              │
                              ▼
                        ENGINEER / CODING
                           AGENT EXECUTES
```

## Final takeaway

This repository is best understood as a **durable, provider-independent, human-in-the-loop agentic planning engine**.

Its sophistication is not primarily in the individual LLM prompts. It is in the engineering surrounding the models:

**typed contracts + deterministic verification + dependency graphs + capability fallbacks + sandboxing + persistence + replay + rate limiting + bounded decision loops + human gates.**

That combination makes the project particularly valuable for learning how to move from simple LLM applications toward **production-grade agentic systems**.
