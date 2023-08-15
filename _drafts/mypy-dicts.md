---
layout: post
title:  "Tip: (Ab)using mypy to catch KeyErrors"
description: "Or, how to get the type system to boss you around in a helpful manner."
date:   2023-01-29 02:00:00 -0400
categories: python mypy
author: Colin Atkinson
---

Python programmers, as many of you are likely aware, love dictionaries. Don't
want to make a class to store a bunch of values? Use a dict. Data
serialization? Use a dict and encode as JSON. Engine troubles? Not sure how,
but you can probably use a dict for that, too.

This is great and gives you that flexibility that dynamic languages crave.
However, in cases where you do want to do heavy validation, it can lead to lots
of runtime errors.

# The problem

Let's say I'm receiving untrusted user input as JSON. Maybe I expect the JSON
to look something like this:

```json
{
    "type": "update",
    "timestamp": 1675021143,
    "update_specific_key": "update_some_stuff"
}
```

Now let's say we want to operate on this data. You could do something like:

```python
import datetime
import json


def handle_update_msg(timestamp: datetime.datetime, usk: str) -> None:
    pass

def handle_unknown_msg(timestamp: datetime.datetime, msg_type: str) -> None:
    pass


def handle_msg(msg_serialized: str) -> None:
    msg = json.loads(msg_serialized)

    msg_type = msg["type"]
    timestamp = msg["timestamp"]
    dt = datetime.datetime.fromtimestamp(timestamp)

    if msg_type == "update":
        update_key = msg["update_specific_key"]
        handle_update_msg(dt, update_key)
        return

    handle_unknown_msg(dt, msg_type)
```

This is all well and good. But what if...

1. The value isn't a dict, and is instead e.g. `[]`?
2. The value isn't valid JSON, and is instead e.g. `{"key": "val`?
3. `timestamp` is actually a string, or null?
4. Any of those keys aren't even set?

That's right, we get... runtime errors! Surely there must be some way around
this.

A typical way to handle this is with [JSON schemas](https://json-schema.org/).
And indeed, they're great for this use case. But unless you're using something
like pydantic, they can be disconnected from the Python type system. For
example, if the schema gets updated to say

```yaml
type: object
properties:
  timestamp:
    type:
      - number
      - string
required:
  - timestamp
```

then we're back at runtime errors again. Surely there must be a better way.

Maybe [pydantic](https://docs.pydantic.dev/) could help? In many cases, yes!
It's a great project. However, it does require specifying your schemas via
pydantic -- and what if the messages are coming from a JavaScript frontend, or
an Android app in Java?

# Enter mypy

[mypy](https://mypy.readthedocs.io/en/stable/) is the *de facto* standard type
checker for Python.

There is a catch though -- mypy is intentionally a bit loosey-goosey around
things like JSON deserialization and dict access, to ease the gradual typing
transition. So for example:

```python
import json

reveal_type(json.loads(""))
```

```bash
$ mypy --strict example.py
example.py:3: note: Revealed type is "Any"
Success: no issues found in 1 source file
```

`Any` is kind of the Python typing cop-out type. Instead of starting from the
narrowest-possible type (i.e. `object`) we instead get the widest-possible type
(i.e. `Any`).

Let's take the example from above, and see if we can make things a bit safer.

```python
import datetime
import json


def handle_update_msg(timestamp: datetime.datetime, usk: str) -> None:
    pass

def handle_unknown_msg(timestamp: datetime.datetime, msg_type: str) -> None:
    pass


def handle_msg(msg_serialized: str) -> None:
    msg = safe_json_load(msg_serialized)

    msg_type = msg["type"]
    timestamp = msg["timestamp"]
    dt = datetime.datetime.fromtimestamp(timestamp)

    if msg_type == "update":
        update_key = msg["update_specific_key"]
        handle_update_msg(dt, update_key)
        return

    handle_unknown_msg(dt, msg_type)


def safe_json_load(msg_serialized: str) -> object:
    msg: object = json.loads(msg_serialized)
    return msg
```

```bash
$ mypy --strict example.py
example.py:15: error: Value of type "object" is not indexable  [index]
example.py:16: error: Value of type "object" is not indexable  [index]
example.py:20: error: Value of type "object" is not indexable  [index]
Found 3 errors in 1 file (checked 1 source file)
```

Now we're getting somewhere. Type safety is pain, darling.

```python
import datetime
import json
from typing import TypeGuard, TypedDict


def handle_update_msg(timestamp: datetime.datetime, usk: str) -> None:
    pass

def handle_unknown_msg(timestamp: datetime.datetime, msg_type: str) -> None:
    pass


def handle_msg(msg_serialized: str) -> None:
    msg = safe_json_load(msg_serialized)
    if not is_dict(msg):
        print("Not a dict!")
        return

    msg_type = msg["type"]
    timestamp = msg["timestamp"]
    dt = datetime.datetime.fromtimestamp(timestamp)

    if msg_type == "update":
        update_key = msg["update_specific_key"]
        handle_update_msg(dt, update_key)
        return

    handle_unknown_msg(dt, msg_type)


def safe_json_load(msg_serialized: str) -> object:
    msg: object = json.loads(msg_serialized)
    return msg


class NarrowDict(TypedDict):
    pass


def is_dict(msg: object) -> TypeGuard[NarrowDict]:
    return isinstance(msg, dict)
```

```bash
$ mypy --strict example.py
example.py:19: error: TypedDict "NarrowDict" has no key "type"  [typeddict-item]
example.py:20: error: TypedDict "NarrowDict" has no key "timestamp"  [typeddict-item]
example.py:24: error: TypedDict "NarrowDict" has no key "update_specific_key"  [typeddict-item]
Found 3 errors in 1 file (checked 1 source file)
```

Well, at least it knows it's a dict now. But it's complaining that our keys
aren't defined. That's good -- that's one of the runtime errors we wanted to
avoid.

## The final version

```python
import datetime
import json
from typing import TypeGuard, TypedDict


def handle_update_msg(timestamp: datetime.datetime, usk: str) -> None:
    pass

def handle_unknown_msg(timestamp: datetime.datetime, msg_type: str) -> None:
    pass


def handle_msg(msg_serialized: str) -> None:
    msg = safe_json_load(msg_serialized)
    if not is_dict(msg):
        print("Not a dict!")
        return

    msg_type = msg.get("type")
    if not isinstance(msg_type, str):
        print("Bad msg_type")
        return

    timestamp = msg.get("timestamp")
    if not isinstance(timestamp, float):
        print("Bad timestamp")
        return

    dt = datetime.datetime.fromtimestamp(timestamp)

    if msg_type == "update":
        update_key = msg.get("update_specific_key")
        if not isinstance(update_key, str):
            print("Bad USK")
            return

        handle_update_msg(dt, update_key)
        return

    handle_unknown_msg(dt, msg_type)


def safe_json_load(msg_serialized: str) -> object:
    msg: object = json.loads(msg_serialized)
    return msg


class NarrowDict(TypedDict):
    pass


def is_dict(msg: object) -> TypeGuard[NarrowDict]:
    return isinstance(msg, dict)
```

```bash
$ mypy --strict example.py
Success: no issues found in 1 source file
```
