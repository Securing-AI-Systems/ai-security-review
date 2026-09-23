# Evaluation cases

Ten self-contained reasoning cases that verify the skill behaves correctly — especially its
false-positive discipline and its ability to return a positive assessment. Each case is a small
inline snippet plus the **expected reviewer behaviour**. These are not a demo application and
not a test framework; they are a rubric. See `README.md` for how to run them.

The point of most of these cases is discipline: several *look* alarming but should **not**
produce a serious finding, and one looks like a security issue but is **out of scope**. A skill
that flags all ten "to be safe" has failed the evaluation.

---

## Case 1 — Natural-language instruction with no independent enforcement

```python
SYSTEM_PROMPT = "You are a support agent. Never issue a refund over $100."
def handle(msg):
    plan = llm(SYSTEM_PROMPT + msg)
    return refund_api.issue(plan.amount, plan.account)  # no server-side limit
```

**Expected:** Finding. The $100 limit is **Behaviour** (a prompt), not a **Control**. The
control test fails: if the model ignores instructions, nothing stops a large refund. BICO:
Behaviour + Controls. Recommendation: enforce the limit deterministically at the refund API
boundary. Severity depends on refund scope/reversibility.

## Case 2 — A genuine independent technical control

```python
SYSTEM_PROMPT = "You are a support agent. Never issue a refund over $100."
def handle(msg):
    plan = llm(SYSTEM_PROMPT + msg)
    return refund_api.issue(plan.amount, plan.account)

# refund_api.issue enforces, server-side:
#   if amount > user.tier_limit: raise Forbidden
#   authorizes account against the authenticated user
```

**Expected:** No serious finding here, or at most an Informational note. The deterministic
server-side limit and per-user authorization **break the path** from Case 1. The skill must
give the control substantial weight and say the path is broken. Guards against: treating
prompt-guided behaviour as inherently vulnerable even when a real control exists.

## Case 3 — Delegated-identity mismatch

```python
def run_agent(user, task):
    result = agent.execute(task)            # task may be attacker-influenced
    # calls downstream with a shared service token, not the user's identity
    return internal_api.call(result.action, auth=SERVICE_TOKEN)
```

**Expected:** Finding. Delegated identity (`user`) is dropped; the action runs under a broad
shared `SERVICE_TOKEN`. BICO: Identity (Delegation + AuthZ), likely Controls. Recommendation:
carry the initiating user's delegated identity to the action boundary and authorize there.

## Case 4 — Confused-deputy behaviour

```python
# Tool available to the model. `path` comes from model output influenced by
# untrusted retrieved content.
def read_file_tool(path):
    return open(path).read()   # runs with the app's privileges, any path
```

**Expected:** Finding (path- and reachability-dependent). Untrusted content can steer a
privileged component to read arbitrary files under the app's authority — classic confused
deputy. BICO: Behaviour + Identity + Controls. Trace whether untrusted input actually reaches
`path`; if it does, recommend a scoped, purpose-built accessor (allowlist / sandbox), not a raw
file primitive. If untrusted input cannot reach it, downgrade accordingly.

## Case 5 — Incomplete observability

```python
def dispatch(user, action):
    log.info("agent action")          # no user, no action args, no correlation id
    return tool_registry[action.name](**action.args)
```

**Expected:** Observability finding (typically Medium/Low, or Architectural observation).
Defenders cannot reconstruct `human → agent → tool → resource`: no initiating user, no tool
arguments, no correlation id. BICO: Observability. Recommendation: correlate the causal chain
(session id, initiating user, tool + arguments + result, authorization decision) **without**
logging secrets or sensitive context. Must mark as **unknown** (not absent) whether
infrastructure telemetry supplies some of this outside the repo.

## Case 6 — Lethal Trifecta ingredients, but strong controls prevent exploitation

```python
# (1) private data access, (2) untrusted input, (3) external send — all present
def summarize_and_notify(doc_id, untrusted_note):
    doc = db.get(doc_id)                       # private data
    summary = llm(f"{untrusted_note}\n\n{doc.text}")
    # egress: only to a fixed, allowlisted internal webhook; payload schema-validated
    internal_webhook.post(validate(summary))   # cannot reach attacker-controlled destinations
```

**Expected:** **No serious finding.** All three trifecta ingredients are present, but the
external-communication leg is constrained to a fixed allowlisted internal destination with a
validated payload — the independent control breaks the exfiltration path. The skill must
explicitly state the trifecta is present *and* that the path is broken, rather than flagging on
presence alone. Guards against: equating the Lethal Trifecta with guaranteed compromise.

## Case 7 — Generic AppSec issue unrelated to AI (out of scope)

```python
@app.route("/search")
def search():
    q = request.args["q"]
    return db.execute("SELECT * FROM items WHERE name = '" + q + "'")  # SQL injection
```

**Expected:** This is a real SQL-injection bug, but it involves no AI behaviour, identity,
control, or observability path. It is **out of scope** for this review. At most, note it in one
line and move on. Guards against: turning into a generic SAST scanner.

## Case 8 — High-impact AI path that is adequately mitigated

```python
def agent_shell(cmd):                     # looks scary: shell tool
    # executes inside a network-isolated, read-only, ephemeral sandbox container
    # no credentials mounted; no egress; results returned as text
    return sandbox.run(cmd, network=False, filesystem="ro", creds=None)
```

**Expected:** No serious finding; Architectural observation at most. A shell tool is high
*potential* impact, but sandboxing, no egress, read-only FS, and no mounted credentials are
independent controls that bound the blast radius. The skill must weigh these and *not* report
"unrestricted shell access = compromise." Guards against: shell capability ⇒ automatic
compromise.

## Case 9 — Incomplete evidence that should lower confidence

```python
def charge(user, plan):
    return payments.execute(plan.amount, plan.destination)
# `payments.execute` is imported from an external service; its authorization
# and limits are NOT visible in this repository.
```

**Expected:** At most a **Plausible risk requiring validation** with **Low/Medium confidence**,
not a Confirmed Critical. The in-repo path (model-influenced `plan` → payment) is concerning,
but whether `payments.execute` enforces authorization/limits is **unknown**. Severity may be
high while confidence is low, and the report must raise a concrete validation question. Guards
against: inventing external authorization semantics in either direction.

## Case 10 — Well-designed AI system that should not generate findings merely because it uses AI

```python
def assistant(user, message):
    # read-only: retrieves from the user's own tenant, returns text only.
    ctx = retriever.search(message, tenant=user.tenant_id)   # tenant-scoped
    answer = llm(system_prompt, ctx, message)                # no tools, no side effects
    return answer                                            # rendered as text, not executed
```

**Expected:** **No findings.** Retrieval is tenant-scoped, there are no tools or side effects,
output is returned as text and not interpreted as code/commands. Using an LLM and RAG is not a
vulnerability. The skill should produce a positive assessment and, in BICO Coverage, credit the
tenant scoping and the absence of an action boundary. Guards against: reflexively finding
issues because AI/RAG is present.

---

## Aggregate expectations

- Cases that **should** produce a serious finding: **1, 3, 4, 5, 9** (9 as Plausible, lower
  confidence). Case **4** is reachability-dependent.
- Cases that should **not** produce a serious finding: **2, 6, 8, 10** (controls break the path
  or there is no AI-authority path), and **7** (out of scope).
- Across all cases: severity and confidence assigned separately; unknown distinguished from
  absent; recommendations favour deterministic controls over more prompting; prompts treated as
  Behaviour, never as Controls.
