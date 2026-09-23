# Reporting

Read this before writing the report. The goal is a report an experienced security engineer
could defend from the cited evidence. Prefer a few defensible findings over many speculative
ones. A secure repository may legitimately receive few or zero significant findings — report
that outcome plainly and still fill in the other sections.

## Structure

Produce these sections, in this order.

### 1. Architecture Summary

Concise description of: the relevant AI architecture; important principals (human, app, agents/
sub-agents, service accounts, tools, MCP servers, external services); consequential
capabilities; sensitive assets where known; important trust boundaries; and the meaningful
*independent* controls that already exist. Give credit for good design here.

### 2. Security Paths

The important trust/execution paths you analyzed, as arrows, e.g.:

`external content → retrieval → model context → privileged tool → internal API → resource change`

List the paths you traced (including ones that turned out to be safe and why), so the reader
sees the shape of the analysis, not just the failures.

### 3. Findings

Order by severity. Each finding is one of the three classifications (Confirmed finding /
Plausible risk requiring validation / Architectural observation). Start with a one-line summary
table or list, then expand each finding. A compact summary line:

> **[Severity] · [Confidence] · [Classification] · BICO: [dimensions]** — one-sentence finding.

Expand each finding with:

- **Classification, Severity, Confidence** — and *why* severity and confidence differ if they do.
- **BICO dimension(s)** — only the ones that actually apply; do not force all four.
- **Evidence** — repository-relative paths with narrow line ranges (`path/file.py:41-58`).
- **Security path** — the `influence → decision → identity → capability → control →
  consequence` chain, with the specific files at each step.
- **What actually happens** — the concrete failure, including the acting identity where known.
- **Realistic security impact** — blast radius of one incorrect/manipulated decision.
- **Existing protections** — controls on the path and why they do or do not break it.
- **Smallest meaningful improvement** — the minimal deterministic change that breaks or
  constrains the path (prefer controls over more prompting).

Mark any step not visible in the repo as **unknown**; do not assume deployed IAM, network
policy, cloud controls, centralized logging, runtime behaviour, or external-API authorization.

### 4. BICO Coverage

Summarize strengths *and* weaknesses across all four dimensions — Behaviour, Identity,
Controls, Observability. Name the good design decisions, not only the gaps.

### 5. Unknowns / Validation Questions

Only unknowns that could materially change the assessment (e.g. production IAM lives outside the
repo; egress policy unavailable; external authorization semantics unverifiable; centralized
telemetry configured elsewhere). Frame each as a concrete question a maintainer can answer.

### 6. Review Coverage

For non-trivial repositories, state what was materially reviewed and any relevant blind spots.

## Worked finding (illustrative format, not a template to copy verbatim)

> **High · Medium · Confirmed finding · BICO: Behaviour, Identity, Controls** — Untrusted web
> content retrieved into agent context can drive a privileged internal API call under a shared
> service identity.
>
> **Security path:** external webpage → `ingest/fetch.py:22-40` (fetched text stored
> unlabeled) → `rag/retriever.py:88-105` (retrieved text concatenated into the system/context
> block) → agent plan → `agents/tools.py:41-58` (`admin_update` tool selected with
> model-supplied arguments) → `clients/internal_api.py:12-30` (call authenticated with a
> shared `SERVICE_TOKEN`, broad scope).
>
> **What actually happens:** Retrieved page content is placed in the model's context with the
> same trust as developer instructions. A crafted page can induce the agent to select
> `admin_update` and supply attacker-chosen arguments. The call executes under a shared service
> token rather than the initiating user's delegated identity, so per-user authorization is not
> enforced at the action boundary (**confused deputy**).
>
> **Realistic impact:** One manipulated retrieval can modify records for any tenant the service
> token can reach. Blast radius = the service token's scope.
>
> **Existing protections:** Input is length-capped (`ingest/fetch.py:31`) and the tool
> validates argument *shape* via a schema (`agents/tools.py:47`). Neither constrains *which*
> records may be changed or *on whose behalf*, so the path is not broken. No egress or network
> control is visible in the repo (**unknown** whether one exists in deployment).
>
> **Confidence rationale:** Medium — the in-repo path is clear, but the deployed scope of
> `SERVICE_TOKEN` and any external authorization on the internal API are not visible.
>
> **Smallest meaningful improvement:** Perform the internal API call under the initiating
> user's delegated identity and authorize at the action boundary (per-record, per-tenant),
> rather than a shared broad-scope token. Optionally, treat retrieved content as data (clearly
> delimited, never as instructions) — but that is a Behaviour mitigation and does not replace
> the authorization control.

Note how the example: cites narrow line ranges; separates severity from confidence; names the
acting identity; credits existing controls while explaining why they don't break the path;
marks deployment facts as unknown; and recommends a deterministic control, not more prompting.
