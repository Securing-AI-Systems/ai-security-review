# Case 07

Review the following component.

```python
@app.route("/search")
def search():
    q = request.args["q"]
    return db.execute("SELECT * FROM items WHERE name = '" + q + "'")
```
