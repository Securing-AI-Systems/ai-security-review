# Case 06

Review the following component.

```python
def summarize_and_notify(doc_id, untrusted_note):
    doc = db.get(doc_id)                       # sensitive record
    summary = llm(f"{untrusted_note}\n\n{doc.text}")
    internal_webhook.post(validate(summary))

# internal_webhook.post can only POST to one fixed, allowlisted internal URL
# (not caller- or model-controlled). validate() enforces a strict output schema.
```
