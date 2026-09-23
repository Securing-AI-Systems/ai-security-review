# AI Security Review

An architecture-first security review for repositories containing **AI-enabled systems** —
agents, LLM integrations, RAG, memory, tools, MCP, and orchestration. Packaged as a native,
read-only [Claude Code Skill](https://docs.claude.com/en/docs/claude-code/skills).

It helps you answer one question:

> If this AI system behaves unexpectedly or is intentionally manipulated, what can actually happen?

This is **not** a generic SAST scanner. It reasons across code, prompts, agents, model
integrations, retrieval, memory, tools, APIs, authentication, authorization, delegated
authority, infrastructure, data flows, trust boundaries, hard controls, and telemetry — to find
risks that emerge specifically from combining **probabilistic AI behaviour** with **real-world
capability and authority**.

## About this repository

This is the public companion repository to *Securing AI Systems: Hardening Agents in Production*
by David Campbell and Dan Borges (O'Reilly). It exists to distribute one durable, standalone
artifact: the `ai-security-review` Claude Code Skill, which operationalizes the book's **BICO**
methodology (Behaviour, Identity, Controls, Observability) as a runnable, architecture-first
security review.

The skill is **fully self-contained** — it needs no part of the book and no external service, and
does not require network access to perform a repository review. The book is the source of the
methodology and the reason this repository exists; it is not a runtime dependency. Copy the skill
into any repository containing an AI-enabled system and it works on its own.

## BICO

The review is organized around **BICO**:

> Security is **behaviour** anchored in **identity**, bounded by **controls**, and made
> traceable through **observability**.

- **Behaviour** — what the system is trying, expected, or induced to do (prompts, objectives,
  retrieved context, memory, tool results, agent planning). Probabilistic and
  attacker-influenceable.
- **Identity** — the security context of an action: **Authentication** (who is acting),
  **Authorization** (what it may do), and **Delegation** (on whose behalf).
- **Controls** — hard, independently enforced technical restrictions on what the system can
  actually do (sandboxes, IAM, network egress rules, schema validation, scoped credentials,
  approvals).
- **Observability** — the evidence needed to reconstruct a consequential causal chain end to
  end.

Two ideas do most of the work:

- **AI systems are probabilistic; behaviour will occasionally fail.** Security engineering
  should ensure a behavioural failure does not automatically become a security incident.
- **Prompts guide behaviour; they are not security boundaries.** "Never send data externally"
  in a prompt is Behaviour. A network policy that blocks egress is a Control. The test:
  *if the model ignored its instructions, would this restriction still exist?*

BICO is used to **trace security paths** through an architecture, e.g.:

```
untrusted webpage → retrieval → model context → agent decision
  → delegated identity → tool call → internal API → resource modification
```

— not as a four-item checklist. A finding may involve one, several, or all four dimensions.

## Install

Copy the skill package into your Claude Code skills directory:

```bash
# Personal (all your projects):
cp -R ai-security-review ~/.claude/skills/ai-security-review

# Or per-project (checked in for a team):
mkdir -p .claude/skills && cp -R ai-security-review .claude/skills/ai-security-review
```

The portable unit is the `ai-security-review/` directory (`SKILL.md`, `references/`, and its own
`LICENSE`). It is self-contained — nothing else in this repository is required at runtime.

## Use

In Claude Code, from inside the repository you want to review:

```
/ai-security-review
```

or ask in natural language, e.g. *"do an AI security review of this repo"*, *"threat model this
agent"*, or *"BICO review"*. You can scope it to a subtree or component. The skill runs in
deliberate phases (reconnaissance → architectural model → path discovery → BICO analysis →
challenge assumptions → evidence validation → report) and produces:

- **Architecture Summary**, **Security Paths**, **Findings** (with severity, confidence,
  classification, BICO dimensions, evidence, and the smallest meaningful fix), **BICO
  Coverage**, **Unknowns / Validation Questions**, and **Review Coverage**.

## Scope and non-goals

**In scope:** security risks arising from AI behaviour combined with identity, controls, and
observability across the whole system.

**Non-goals:**

- Not a generic SAST/AppSec scanner. Generic bugs unrelated to AI authority are out of scope.
- Not a benchmark of the model, and not a live/active security test — it does not attack running
  systems.
- Not a remediation tool. It is **read-only** (see below); fixes are a separate, explicit task.
- Low false-positive rate is a design goal: a secure repository may receive few or zero
  significant findings, and the review will say so.

## Read-only

The skill does not modify code, prompts, dependencies, configuration, IAM, or infrastructure;
does not install anything; and does not remediate automatically. This boundary is enforced by the
harness, not left to model intent: the skill's `disallowed-tools` frontmatter **removes** the
file-mutation and command-execution tools (`Edit`, `Write`, `NotebookEdit`, `Bash`) for the
duration of the review, while `allowed-tools` pre-approves the read-only inspection tools it does
use (`Read`, `Grep`, `Glob`) so the review runs without permission prompts.

The enforced read-only boundary prevents file mutation and command execution — it is **not** a
network-isolation control. Network-capable tools (WebSearch, WebFetch, MCP integrations) are not
removed; the skill simply does not require network access to perform a repository review, so treat
the lack of network dependency as a design property, not a guaranteed isolation boundary.

## Framework independence

BICO is provider- and framework-independent. The skill works across Anthropic, OpenAI, Google,
AWS Bedrock, Azure, and local/open models; LangChain, LangGraph, CrewAI, AutoGen, MCP, direct
APIs, and custom agent frameworks; serverless, containers, and cloud-native infrastructure.

## Repository layout

```
.
├── README.md                     # this file
├── LICENSE                       # Apache-2.0
├── ai-security-review/           # the portable skill (copy this into .claude/skills/)
│   ├── SKILL.md                  # operational core: workflow, FP discipline, evidence, report
│   ├── LICENSE                   # Apache-2.0 (travels with the skill directory)
│   └── references/               # progressive disclosure — loaded on demand
│       ├── bico.md               # BICO dimension definitions + probing questions
│       ├── principles.md         # supporting principles + large-repo prioritization
│       └── reporting.md          # report structure + a worked finding
└── evaluation/                   # rubric-based eval (not shipped with the skill)
    ├── README.md                 # how to run + pass/fail criteria
    ├── cases/                    # 10 inputs, one per file (the only files the skill sees)
    └── expected.md               # evaluator-only answer key (kept away from the skill)
```

## Provenance

The BICO methodology is from *Securing AI Systems: Hardening Agents in Production* by
**David Campbell** and **Dan Borges** (O'Reilly). This repository is the public companion to the
book. The skill is self-contained: using it requires neither the book nor any external resource.

## License

Apache-2.0. See [LICENSE](LICENSE).
