# BICO — dimension reference

Read this when analyzing a security path in detail. `SKILL.md` carries the operational summary
(anchor, tracing framing, the control test); this file carries the probing questions and
patterns for each dimension.

---

## Behaviour

What the system is trying, expected, or induced to do. It is probabilistic and
attacker-influenceable. Sources of influence include: system prompts, developer instructions,
user prompts, objectives, plans, model configuration and selection, fine-tuning, alignment,
guardrails, retrieved context / RAG, memory, stored summaries, model output, tool results,
API responses, routing, orchestration, agent planning, tool selection, and agent-to-agent
messages.

Probing questions:

- What influences the model or agent on this path?
- Which inputs are trusted, partially trusted, or untrusted?
- Can user-controlled input affect a consequential decision?
- Can retrieved documents, webpages, email, files, images, database content, memory, API
  responses, tool results, MCP output, or other agents influence behaviour?
- Can attacker-controlled data affect plans, routing, tool selection, or tool arguments?
- Is model output later interpreted as code, commands, SQL, configuration, URLs, policy,
  authorization, or executable instructions?
- Does the architecture assume prompting or alignment will reliably prevent a dangerous
  action?

The central Behaviour question: **what happens when behaviour fails?** Never treat refusal
behaviour, prompts, natural-language guardrails, or model alignment as equivalent to
deterministic security enforcement.

---

## Identity

The security context of an action. Always three concepts together:

- **Authentication (AuthN):** who or what is acting?
- **Authorization (AuthZ):** what is that identity permitted to do?
- **Delegation:** on whose behalf is it acting?

Agentic systems have many principals — human users, applications, agents, sub-agents,
workloads, service accounts, cloud roles, API credentials, OAuth principals, tools, MCP
servers, internal and external services. Do not collapse them into a generic "agent."

For each consequential action:

- Who initiated the workload?
- Which principal actually performs the action?
- What permissions does that principal possess?
- Whose authority is being exercised?
- Is authorization checked at the *actual action boundary* (not merely at the UI or the model)?
- Is the acting identity more privileged than the user or the data influencing it?
- Can low-trust input cause a high-trust identity to act? (**confused deputy**)
- Can one user cause actions under another user's authority?
- Does delegated identity survive agent → sub-agent → service → tool transitions?
- Are credentials broadly scoped or unnecessarily long-lived?
- Can provenance be reconstructed?

**A model deciding an action is appropriate is not authorization.** The key questions: *Who is
acting? What are they allowed to do? On whose behalf?*

**Confused deputy** — a recurring, high-value pattern: a privileged component performs an
action requested (directly or indirectly) by a less-privileged or untrusted party, using the
privileged component's own authority instead of the requester's. In AI systems this appears
when untrusted retrieved/tool content steers a privileged agent, or when delegated user
identity is dropped and a shared service identity is used instead.

---

## Controls

Hard or independently enforced technical restrictions on what the system can actually do.
Examples: filesystem permissions, process isolation, containers, sandboxes, network
segmentation and egress restriction, narrowly scoped APIs and tools, IAM and resource
policies, policy engines, schema validation, typed interfaces, deterministic output
validation, transaction and rate limits, credential brokers, short-lived credentials,
capability tokens, approval workflows, isolation between agents/workloads, externally enforced
human authorization.

The test, applied repeatedly: **if the model completely ignored its instructions, would this
restriction still exist?** If not, it is Behaviour, not a hard Control.

For each important capability:

- What *independently* prevents abuse?
- Does the control fail closed?
- Can the model modify or disable it?
- Can another path bypass it? Does another tool expose the same underlying capability?
- Is privilege narrowly scoped? Is there defense in depth?
- What happens if the control fails?
- Can the model reach a lower-level primitive that bypasses the intended abstraction (e.g. a
  shell tool that subsumes a "safe" purpose-built tool)?
- Has convenience produced broad shell / filesystem / network / database / cloud / admin
  access?

Prefer `default deny → explicit enablement → continuous review` over `grant broad capability →
ask the model to behave`.

**Control decay:** controls lose effectiveness as systems change — permissions expand,
exceptions accumulate, integrations change, old assumptions expire. Consider decay only where
the repository provides evidence for it; do not claim decay merely because something is old.

---

## Observability

The evidence needed to reconstruct what actually happened. Prompt/response logging alone is
generally insufficient for agentic systems. The bar is whether defenders could reconstruct a
causal chain such as:

`human → workload → agent → sub-agent → tool → authorization decision → API → resource
modification`

Relevant signals: session/workload identifier, initiating user, delegated authority, model
invocation, agent and sub-agent identity, instruction provenance, retrieved-context
provenance, tool invocation with arguments and results, authorization decisions and policy
allow/deny events, approvals, API calls, network activity, filesystem/resource changes,
failures, retries, and downstream effects.

Ask: What happened? Who initiated it? Which identity performed it? On whose behalf? Which
control permitted or rejected it? What resource was affected? What happened afterward? **Can
those events be correlated end to end?**

Two disciplines:

- **Unknown ≠ absent.** Missing logging *code* does not mean observability is missing —
  telemetry may come from cloud infrastructure or another system outside the repo. When the
  evidence isn't in the repo, mark it unknown.
- **Do not recommend indiscriminate logging** of secrets, credentials, private data, sensitive
  prompts, protected model context, or hidden reasoning / chain-of-thought. Observability
  should improve security without creating a new sensitive-data liability.

---

## Fail-securely lens (applies across Identity and Controls)

Failures should reduce capability, not expand it. Look for: missing authorization defaulting
to allow; failed authentication creating a privileged fallback; unavailable policy systems
permitting execution; malformed model output falling back to a dangerous default; validation
failures that bypass validation; missing identity information causing use of a shared
privileged identity; unavailable safety systems resulting in unrestricted operation.
