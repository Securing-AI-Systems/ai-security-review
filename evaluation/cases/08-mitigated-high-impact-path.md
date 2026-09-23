# Case 08

Review the following component.

```python
# Tool exposed to the model.
def agent_shell(cmd):
    # Runs inside an ephemeral, network-isolated container: read-only filesystem,
    # no egress, no credentials mounted. Only stdout/stderr text is returned.
    return sandbox.run(cmd, network=False, filesystem="ro", creds=None)
```
