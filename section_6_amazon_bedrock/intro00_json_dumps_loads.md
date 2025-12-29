# json.dumps() and json.loads() in Python

In Python, `json.dumps` and `json.loads` are used to **convert between
dict Python objects and JSON-formatted strings**. They are opposites of each
other.

------------------------------------------------------------------------

## `json.dumps()` --- Python → JSON string

**What it does:**\
Takes a dict Python object and **serializes (dump it)** it into a JSON-formatted **string**.

### Supported type conversions

-   `dict` → JSON object\
-   `list` / `tuple` → JSON array\
-   `str` → JSON string\
-   `int`, `float` → JSON number\
-   `True` / `False` → `true` / `false`\
-   `None` → `null`

### Example

``` python
import json

data = {
    "name": "Alice",
    "age": 30,
    "is_active": True,
    "skills": ["Python", "SQL"],
    "spouse": None
}

json_string = json.dumps(data)
print(json_string)
```

**Output:**

``` json
{"name": "Alice", "age": 30, "is_active": true, "skills": ["Python", "SQL"], "spouse": null}
```

📌 The result is a **string**, not a dictionary.

------------------------------------------------------------------------

## `json.loads()` --- JSON string → Python

**What it does:**\
Takes a JSON-formatted **string** and **deserializes (load it)** it back into a
dict Python object.

### Example

``` python
json_string = '{"name": "Alice", "age": 30, "is_active": true}'
data = json.loads(json_string)

print(data)
print(type(data))
```

**Output:**

``` python
{'name': 'Alice', 'age': 30, 'is_active': True}
<class 'dict'>
```

------------------------------------------------------------------------

## Visual summary

    dict Python object  --json.dumps()-->  JSON string
    JSON string   --json.loads()-->   dict Python object

------------------------------------------------------------------------

## Common pitfalls

-   `json.dumps()` does **not** write to a file → use `json.dump()`
-   `json.loads()` expects a **string** → use `json.load()` for files
-   Not all dict Python objects are JSON-serializable

------------------------------------------------------------------------

## Quick comparison

  Function         Input           Output
  ---------------- --------------- ---------------
  `json.dumps()`   dict Python object   JSON string
  `json.loads()`   JSON string     dict Python object