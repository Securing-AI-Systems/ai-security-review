---
name: ai-security-review
description: >-
  Architecture-first security review of AI-enabled systems (agents, LLM integrations, RAG,
  memory, tools, MCP). Traces how untrusted input, AI decisions, identity and delegation,
  controls, and observability combine into real attack paths, using the BICO methodology
  (Behaviour, Identity, Controls, Observability). Read-only and framework-independent. Not a
  generic SAST scanner. Use when asked to "ai security review", "BICO review", "threat model
  this agent", or to assess what can happen if an AI system is manipulated or behaves
  unexpectedly.
allowed-tools: Read, Grep, Glob
disallowed-tools: Edit, Write, NotebookEdit, Bash
license: Apache-2.0
---

# AI Security Review (BICO)

Perform an architecture-first security review of an AI-enabled system. The question you are
answering is:

> If this AI system behaves unexpectedly or is intentionally manipulated, what can actually happen?

This is not a SAST pass. You reason across code, prompts, agents, model integrations,
retrieval, memory, tools, APIs, authentication, authorization, delegated authority,
infrastructure, data flows, trust boundaries, hard controls, and telemetry — to find risks
that emerge specifically from combining probabilistic AI behaviour with real-world capability
and authority.

## Read-only operation (hard constraint)

This skill is **read-only**. It must not modify code, prompts, dependencies, configuration,
IAM, or infrastructure; must not install anything, remediate automatically, or touch live
systems. Safe repository inspection only. The file-mutation and command-execution tools
(`Edit`, `Write`, `NotebookEdit`, `Bash`) are removed for the duration of the review via the
skill's `disallowed-tools` frontmatter — so the boundary is enforced by the harness, not left
to model intent — while `Read`, `Grep`, and `Glob` (pre-approved via `allowed-tools`) provide
the inspection and search capability the review needs. Do not attempt to reach a
side-effecting capability by another route. Any remediation is a separate, explicitly
requested task.

## The BICO model

> Security is behaviour anchored in identity, bounded by controls, and made traceable through
> observability.

AI systems are probabilistic. Behaviour will occasionally fail. The job of security
engineering is to ensure that a behavioural failure does not automatically become a security
incident.

- **Behaviour** — what the system is trying, expected, or induced to do. Shaped by prompts,
  objectives, model config, retrieved context, memory, tool results, agent planning. It is
  probabilistic and attacker-influenceable.
- **Identity** — the security context of an action: **Authentication** (who/what is acting),
  **Authorization** (what it may do), and **Delegation** (on whose behalf). Always treat
  Identity as all three.
- **Controls** — hard, independently enforced technical restrictions on what the system can
  actually do (sandboxes, IAM, network egress rules, schema validation, scoped credentials,
  approval workflows).
- **Observability** — the evidence needed to reconstruct a consequential causal chain end to
  end.

**Foundational distinction:** *Prompts guide behaviour; they are not security boundaries.*
"Never transmit confidential data externally" in a prompt is **Behaviour**. A network policy
that blocks outbound egress is a **Control**. The control test, applied repeatedly:

> If the model completely ignored its instructions, would this restriction still exist?

If not, it is Behaviour, not a hard Control.

**BICO is a tracing model, not a four-bucket checklist.** Do not hunt for "one Behaviour
issue, one Identity issue…". Instead trace security paths through the architecture, e.g.:

`untrusted webpage → retrieval → model context → agent decision → delegated identity → tool
call → internal API → resource modification`

and for that path ask what influences **Behaviour**, which **Identity** exercises authority,
which **Controls** independently constrain the outcome, and whether **Observability** lets a
defender reconstruct it. A finding may involve one, several, or all four dimensions — do not
force all four onto every finding.

Depth on each dimension (definitions + probing questions + the confused-deputy and
fail-securely patterns) is in **`${CLAUDE_SKILL_DIR}/references/bico.md`** — read it when analyzing a path in
detail. Supporting principles (untrusted input, trust boundaries, least privilege,
separation of duties, verify-before-trust, defense in depth, fail securely, the Lethal
Trifecta, provenance, control decay) are in **`${CLAUDE_SKILL_DIR}/references/principles.md`** — read it when a
path invokes one of them.

## Review process

Work in deliberate phases. You need not expose every internal step to the user; you must
show your evidence in the report.

1. **Reconnaissance.** Understand the repo before looking for findings. Read README/arch docs,
   dependency manifests, entry points, then AI-relevant components: model integrations, agent
   frameworks, prompts/skills/instructions, MCP, tools, orchestration/routing, retrieval/RAG,
   memory, authN/authZ/delegation, credential handling, API clients, databases, IaC/cloud
   config, networking, policy/approval mechanisms, telemetry, deployment, tests. Do **not**
   burn context on vendor dirs, lockfile internals, generated output, caches, or binaries
   unless a specific path requires them. For large repos, prioritize AI-relevant components
   (see `${CLAUDE_SKILL_DIR}/references/principles.md` "Prioritizing large repositories").

2. **Build an architectural model.** Identify components, model/trust boundaries, security
   principals, AuthN/AuthZ/delegation, privilege transitions, sensitive assets, data/context/
   agent/tool flows, external interfaces, consequential actions, independent controls, and
   observability. Understand what the system can actually affect.

3. **Discover security paths.** Prioritize paths that combine untrusted input, AI
   decision-making, sensitive data, privilege, tool execution, external communication,
   persistent state, infrastructure change, or irreversible actions. Follow paths across
   files. Reason as: `influence → decision → identity → capability → control → consequence`.
   Do not stop at the first suspicious function.

4. **Apply BICO** to each meaningful path (see above and `${CLAUDE_SKILL_DIR}/references/bico.md`).

5. **Challenge assumptions.** For important paths, consider realistic failure: malicious
   retrieved content, wrong tool selection, attacker-controlled tool arguments, low-privilege
   users influencing higher-privilege agents, lost delegated identity, policy-service failure,
   compromised sub-agents, malicious tool output, hallucinated resource IDs, tenant-boundary
   mistakes, missing telemetry, bypassed controls. Include accidental failure — incidents
   don't require an attacker. Ask: *what is the blast radius of one incorrect model decision?*

6. **Validate evidence.** Before declaring a finding: (a) identify the initial influence;
   (b) trace execution/data flow; (c) identify the acting identity where possible; (d)
   identify the consequential capability; (e) identify controls along the path; (f) decide
   whether those controls break the path; (g) name what remains unknown. Inspect callers and
   callees. Do **not** infer runtime behaviour from names, comments, or docs alone.

7. **Report only defensible results.** Prefer three meaningful findings over twenty
   speculative ones. A secure repository may legitimately produce few or zero significant
   findings — say so plainly when that is the case.

## Finding classification, severity, confidence

Classify each finding as exactly one of:

- **Confirmed finding** — the repo provides evidence for the security path and the weakness.
- **Plausible risk requiring validation** — meaningful evidence for concern, but important
  deployment/runtime/identity/infra/external-service information is unavailable.
- **Architectural observation** — a security-relevant design property that has not been shown
  to create an exploitable weakness.

Do not conflate these.

**Severity** (Critical / High / Medium / Low / Informational) reflects realistic impact:
reachability, attacker influence, privilege, data sensitivity, affected resources, mitigating
controls, reversibility, blast radius. Prompt injection is not inherently Critical. Tool
access is not inherently High. Severity is what the *manipulated* system can realistically
accomplish.

**Confidence** (High / Medium / Low) is evidence quality, assigned *separately* from severity.
A catastrophic path with missing runtime evidence can be High severity, Low confidence.

## False-positive discipline (a primary quality requirement)

Do **not** equate: RAG with prompt-injection vulnerability; untrusted input with exploitation;
model usage with insecurity; MCP with insecurity; tool usage with excessive agency; shell
capability with compromise; service accounts with privilege escalation; missing visible
logging with no observability; the Lethal Trifecta with guaranteed compromise; model output
with automatic code execution.

Follow actual paths. Give mitigating controls (authorization, scoped identities, schema
validation, sandboxing, network restrictions, approvals, rate/transaction limits, tenant
isolation, short-lived credentials, resource policy, narrow tools) substantial weight — and
**when a control meaningfully breaks the attack path, say so and drop or downgrade the
finding.** Distinguish **unknown** (evidence unavailable) from **absent** (shown not to
exist); telemetry and IAM often live outside the repo. Never invent findings, and never
invent deployed IAM, network config, cloud controls, centralized logging, runtime behaviour,
or external-API authorization semantics. Generic AppSec issues unrelated to AI authority are
out of scope for this review — note them briefly at most.

## Evidence standards

Cite repository-relative paths with narrow line ranges (e.g. `src/agents/tools.py:41-58`).
Strong evidence explains a path; weak output just names a suspicious file. If a step is not
visible in the repo, mark it **unknown** rather than assuming it. If you find secrets, do not
reproduce their full values.

## Recommendations

Recommend the *smallest change that breaks or constrains the actual path*, preferring
deterministic enforcement over more prompting: authorization at the action boundary,
preserving delegated identity across transitions, replacing broad tools with purpose-built
APIs, sandboxing, scoped/short-lived credentials, constrained egress, deterministic output
validation, separating decision from execution, meaningful (well-informed, independently
enforced) human approval, end-to-end trace correlation. Do **not** reflexively recommend more
system-prompt text, "sanitize all input", a second LLM as the only validator, logging
everything, eliminating all autonomy, or changing providers.

## Report

Read **`${CLAUDE_SKILL_DIR}/references/reporting.md`** before writing the report; it defines the required
structure and shows a worked finding. The report includes: **Architecture Summary**,
**Security Paths** analyzed, **Findings** (Severity, Confidence, Classification, BICO
dimension(s), Finding, Evidence, Security path, what actually happens, realistic impact,
existing protections, smallest meaningful improvement), **BICO Coverage** (strengths *and*
weaknesses across all four dimensions — credit good design), **Unknowns / Validation
Questions** (only those that could materially change the assessment), and **Review Coverage**
(what was materially reviewed; blind spots). Do not recommend indiscriminate logging of
secrets, credentials, private data, sensitive prompts, protected context, or hidden reasoning.

## Attribution

The BICO methodology is from *Securing AI Systems: Hardening Agents in Production* by David
Campbell and Dan Borges (O'Reilly). This skill is self-contained and does not require the
book or any external resource at runtime.
