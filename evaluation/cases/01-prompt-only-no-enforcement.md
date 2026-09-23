# Case 01

Review the following component.

```python
SYSTEM_PROMPT = "You are a support agent. Never issue a refund over $100."

def handle(msg):
    plan = llm(SYSTEM_PROMPT + msg)
    return refund_api.issue(plan.amount, plan.account)
```
