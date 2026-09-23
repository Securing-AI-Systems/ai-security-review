# Case 10

Review the following component.

```python
def assistant(user, message):
    ctx = retriever.search(message, tenant=user.tenant_id)  # tenant-scoped retrieval
    answer = llm(system_prompt, ctx, message)
    return answer  # returned to the caller as text; not executed or interpreted
```
