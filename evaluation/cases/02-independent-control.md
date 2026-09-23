# Case 02

Review the following component.

```python
SYSTEM_PROMPT = "You are a support agent. Never issue a refund over $100."

def handle(msg):
    plan = llm(SYSTEM_PROMPT + msg)
    return refund_api.issue(plan.amount, plan.account)

# refund_api.issue enforces, server-side:
#   if amount > user.tier_limit: raise Forbidden
#   authorizes `account` against the authenticated user
```
