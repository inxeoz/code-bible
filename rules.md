# Coding Rules

## Nested Scope

Nesting is allowed up to 2 levels only.

**valid** 2 level only:

```python
def my_func():
    if cond1:
		#do something
```



**Invalid** (3 levels):

```python
def my_func():
    if cond1:
        if cond2:
            # do something
            pass
```

**Invalid** (extra blank lines do not reduce nesting):

```python
def my_func():

    if cond1:

        if cond2:
            # do something
            pass

        # do something
        pass
```

---

## if-else

Do not use `else` in any condition. It can bypass hidden conditions.

**Valid** — use separate `if` statements:

```python
def my_func():
    if cond1:
        pass
    if cond2:
        pass
```

**Invalid** — nested `if` with `else`:

```python
def my_func():
    if cond1:
        if cond2:
            pass
        else:
            pass
    else:
        pass
```

---

## try-catch

Nested `try`/`catch` blocks are not allowed.

**Valid** — single, flat `try`/`catch`:

```python
def my_func():
    try:
        do_work()
    except ValueError as err:
        handle(err)
```

**Invalid** — nested `try`/`catch`:

```python
def my_func():
    try:
        try:
            do_work()
        except ValueError as err:
            handle(err)
    except Exception as err:
        log(err)
```

---

## if-elif

`if`/`elif` chains are not allowed. Use a mapping, dictionary, or switch/case instead.

**Valid** — dispatch via a dictionary:

```python
def my_func(action):
    handlers = {
        "create": create_item,
        "update": update_item,
        "delete": delete_item,
    }
    handler = handlers.get(action, default_handler)
    handler()
```

**Invalid** — `if`/`elif` chain:

```python
def my_func(action):
    if action == "create":
        create_item()
    elif action == "update":
        update_item()
    elif action == "delete":
        delete_item()
    else:
        default_handler()
```

---

## Hard-Coded Values

Avoid hard-coding values. Every value must have a default and be changeable at runtime.

**Valid** — configurable with defaults:

```python
def connect(host="localhost", port=5432, timeout=30):
    return open_connection(host, port, timeout)
```

**Invalid** — magic numbers and fixed strings:

```python
def connect():
    return open_connection("localhost", 5432, 30)
```

---

## Component Unfolding

A function must not unfold multiple components. A function should only orchestrate components. Each component gets its own unfolding function (one function per component).

**Valid** — orchestration only, conditions extracted into helpers:

```python
def my_func():
    cond1 = CheckCodintion(values)
    cond2 = CheckConditionByDifferentStyle(values)
    if cond2 and cond1:
        cond3 = CheckConditionByDif2(values)

    if cond3:
        cond1 = cond2 = cond3

    if cond2 and not cond1:
        cond2 = False
```

**Invalid** — inlining condition checks and nested unfolding inside one function:

```python
def my_func():
    if values is not None:
        if 5 in values:
            cond1 = True
    if values is not None:
        if 7 in values and 8 in values:
            cond2 = True

    if cond2 and cond1:
        if values is not None:
            if 7 in values and 8 in values and values[0] != 0:
                cond3 = True

    if cond3:
        cond1 = cond2 = cond3

    if cond2 and not cond1:
        cond2 = False
```
