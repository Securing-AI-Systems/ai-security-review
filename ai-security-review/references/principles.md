# Supporting principles

BICO (`${CLAUDE_SKILL_DIR}/references/bico.md`) is the primary methodology. Use these principles when they
materially improve an analysis. None of them is a finding on its own — each must connect to a
real path with real consequence.

---

## Untrusted input

Treat external input as potentially malicious until validated *within the intended context*.
For AI systems, input is far more than chat messages: retrieved documents, webpages, email,
files, images, database content, vector stores, stored memory, prior summaries, API responses,
tool output, MCP responses, third-party skills, agent-to-agent messages, generated code, and
structured model output. Trace where untrusted input enters and what downstream authority it
can influence. **Untrusted input alone is not a vulnerability.**

## Trust boundaries

A trust boundary exists where assumptions about behaviour, identity, permission, provenance, or
security guarantees change. Examples: `user → application`, `external content → model context`,
`model → tool`, `agent → sub-agent`, `tool → OS`, `agent → credential broker`, `application →
internal API`, `service account → production resource`, `agent → Internet`. **Prioritize
boundaries where authority increases.**

## Least privilege

Components should receive only the capabilities they need. Look for overly broad tools,
unrestricted shell/filesystem/network access, wildcard IAM, unnecessary production access,
shared privileged identities, cross-user/cross-tenant capability, and agents that both decide
and perform high-impact operations without an independent boundary. **Do not call something
over-privileged simply because it has meaningful permissions** — determine whether those
permissions are necessary and appropriately constrained.

## Separation of duties

Prefer focused tools/agents over one component with unrelated responsibilities. Ask whether
capabilities can be separated, whether separate responsibilities could use separate identities,
whether a narrower abstraction would reduce blast radius, and whether probabilistic reasoning
is unnecessarily coupled directly to privileged execution. **More agents are not automatically
safer** — separation must create a meaningful security boundary.

## Verify before trust

Treat AI-generated artifacts as untrusted when consequences matter: source code, commands, SQL,
URLs, structured output, configuration, tool arguments, plans, policy recommendations,
authorization claims. Validate outside the model where practical. Human approval is an effective
control only if it is *independently enforced* and the reviewer receives enough information to
understand the action.

## Defense in depth

Assume individual controls may fail. Identify important paths where the failure of one control
immediately creates serious compromise, and prefer independent overlapping controls when impact
warrants. **Do not mistake duplicated controls that share the same failure mode for true
defense in depth.**

## Fail securely

Failures should reduce capability rather than expand it. (See the fail-securely lens in
`${CLAUDE_SKILL_DIR}/references/bico.md` for the concrete patterns.)

## Lethal Trifecta

Determine whether an AI component has all three of: (1) access to private/sensitive
information; (2) exposure to untrusted/attacker-controlled input; (3) a mechanism to
communicate data externally. **Presence of all three is not itself a vulnerability.** Trace
whether a real path connects them and what independent control breaks that path (e.g. egress
restriction, scoped data access, output validation, tenant isolation). Report a finding only
when the path is real and unbroken.

## Prompt, skill, and context provenance

Prompts, skills, memory, summaries, repository instructions, and reusable context are part of
the AI software supply chain because they influence Behaviour. When relevant, determine where
instructions originate, whether they are version-controlled, who can alter them, whether trusted
and untrusted instructions are mixed, whether provenance is retained, whether persistent memory
can preserve adversarial influence, and whether third-party skills/instructions can materially
change execution. These influence Behaviour — they are not themselves hard enforcement.

---

## Prioritizing large repositories

For very large repositories, do not attempt exhaustive coverage. Prioritize AI-relevant
components and the paths with the highest potential blast radius:

1. Entry points where untrusted input enters (APIs, webhooks, chat, ingestion, crawlers, email).
2. Model and agent integrations, orchestration, routing, tool/skill definitions, MCP servers.
3. Retrieval/RAG, memory, and any store whose content re-enters model context.
4. The action boundaries where the system affects the world: tool execution, internal APIs,
   databases, cloud/IaC, filesystem, network egress, payments, and irreversible operations.
5. Identity and credential handling around those boundaries.
6. Telemetry around those boundaries.

Skip vendor directories, generated output, caches, binaries, and lockfile internals unless a
specific path requires them. In the report's **Review Coverage** section, state what was
materially reviewed and any blind spots.
