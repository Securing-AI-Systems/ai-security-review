# Case 03

Review the following component.

```python
def run_agent(user, task):
    result = agent.execute(task)   # `task` may be attacker-influenced
    return internal_api.call(result.action, auth=SERVICE_TOKEN)

# SERVICE_TOKEN is a shared, broadly scoped service credential.
```
