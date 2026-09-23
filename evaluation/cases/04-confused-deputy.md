# Case 04

Review the following component.

```python
# Tool exposed to the model. `path` is taken from model output, which is
# influenced by untrusted retrieved content earlier in the pipeline.
def read_file_tool(path):
    return open(path).read()
```
