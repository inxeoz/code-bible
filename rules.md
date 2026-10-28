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

---

# Additional Maintainability & Readability Rules

## Naming & Intent

### Name things by intent, not type

Use names that describe purpose. Avoid abbreviations unless universal.

**Valid:**

```python
user_count = len(users)
is_valid = validate(user)
```

**Invalid:**

```python
n = len(users)
flag = validate(user)
```

### No boolean parameters (flag arguments)

A boolean argument hides two behaviors. Split into separate functions or pass an enum/strategy.

**Valid:**

```python
def render(item):
    ...

def render_highlighted(item):
    ...
```

**Invalid:**

```python
def render(item, highlight=False):
    if highlight:
        ...
```

### Avoid "and" in function names

A name with "and" usually does two things. Split it.

**Valid:**

```python
def validate(user):
    ...

def save(user):
    ...
```

**Invalid:**

```python
def validate_and_save(user):
    ...
```

---

## Function Shape

### One level of abstraction per function

Do not mix high-level orchestration with low-level parsing in the same body.

**Valid:**

```python
def process_order(order):
    items = parse_items(order)
    total = compute_total(items)
    save(order, total)
```

**Invalid:**

```python
def process_order(order):
    items = [parse_line(line) for line in order.split("\n")]
    total = sum(i.price * i.qty for i in items)
    db.insert("orders", {"items": items, "total": total})
```

### Limit parameters

More than ~3-4 arguments suggests a missing object or config. Group related args.

**Valid:**

```python
def connect(config):
    ...
```

**Invalid:**

```python
def connect(host, port, timeout, retries, tls, proxy):
    ...
```

### Prefer guard clauses / early returns

Return early instead of nesting. Complements the nesting-depth rule.

**Valid:**

```python
def discount(user):
    if not user:
        return 0
    if not user.is_active:
        return 0
    return user.tier.discount
```

**Invalid:**

```python
def discount(user):
    if user:
        if user.is_active:
            return user.tier.discount
    return 0
```

### No mutable default arguments

Mutable defaults are shared across calls.

**Valid:**

```python
def append_item(item, items=None):
    if items is None:
        items = []
    items.append(item)
    return items
```

**Invalid:**

```python
def append_item(item, items=[]):
    items.append(item)
    return items
```

### Prefer pure functions where possible

Keep side effects (I/O, DB, network) at the edges. Core logic takes inputs and returns outputs.

**Valid:**

```python
def total_price(items):
    return sum(i.price * i.qty for i in items)
```

**Invalid:**

```python
def total_price(items):
    total = sum(i.price * i.qty for i in items)
    db.save("totals", total)
    return total
```

---

## Control Flow

### Avoid stringly-typed dispatch

Use enums or constants, not raw strings scattered through code.

**Valid:**

```python
class Action(Enum):
    CREATE = "create"
    UPDATE = "update"

def handle(action):
    ...
```

**Invalid:**

```python
def handle(action):
    if action == "create":
        ...
    elif action == "update":
        ...
```

### No silent failures

Do not swallow exceptions. Handle explicitly or let them propagate with context.

**Valid:**

```python
try:
    save(user)
except DBError as err:
    log(err)
    raise
```

**Invalid:**

```python
try:
    save(user)
except Exception:
    pass
```

### Fail fast

Validate inputs at boundaries. Do not let bad data travel deep before erroring.

**Valid:**

```python
def create_user(data):
    if not data.get("email"):
        raise ValueError("email required")
    ...
```

**Invalid:**

```python
def create_user(data):
    ...
    send_email(data["email"])  # fails deep inside
```

---

## Structure & Coupling

### Separation of concerns

Parsing, validation, business logic, and I/O belong in distinct layers.

**Valid:**

```python
def handle_request(raw):
    data = parse(raw)
    validate(data)
    result = process(data)
    return render(result)
```

**Invalid:**

```python
def handle_request(raw):
    data = json.loads(raw)
    if not data.get("email"):
        return "bad"
    result = db.query("SELECT ...")
    return f"<html>{result}</html>"
```

### Dependency direction

Inner/domain code must not depend on outer/infrastructure code.

**Valid:**

```python
# domain/user.py
class User:
    ...
```

**Invalid:**

```python
# domain/user.py
from infra.db import session
class User:
    def save(self):
        session.commit()
```

### Avoid global mutable state

Pass dependencies explicitly instead of reaching for module globals.

**Valid:**

```python
def process(order, db):
    db.save(order)
```

**Invalid:**

```python
DB = connect()

def process(order):
    DB.save(order)
```

### Prefer composition over inheritance

Deep hierarchies are hard to reason about. Compose small behaviors.

**Valid:**

```python
class Order:
    def __init__(self, pricing, shipping):
        self.pricing = pricing
        self.shipping = shipping
```

**Invalid:**

```python
class Order(BaseOrder, PricedOrder, ShippedOrder, LoggedOrder):
    ...
```

---

## Data & Defaults

### Sentinel values must be documented

If `None`/`-1`/`""` means "unset", name it or use an explicit optional/enum.

**Valid:**

```python
UNSET = object()

def find(id=UNSET):
    ...
```

**Invalid:**

```python
def find(id=None):
    # None means "not provided"
    ...
```

### Single source of truth

Do not duplicate a value in two places that can drift. Derive one from the other.

**Valid:**

```python
PRICE = 10
def total(qty):
    return PRICE * qty
```

**Invalid:**

```python
PRICE = 10
TOTAL_FOR_5 = 50  # must be kept in sync manually
```

---

## Readability Hygiene

### No commented-out code or dead branches

Delete them. Version control remembers.

**Valid:**

```python
def calc(x):
    return x * 2
```

**Invalid:**

```python
def calc(x):
    # return x + 1
    # if x > 0:
    #     return x
    return x * 2
```

### No magic strings or numbers

Extend the no-hard-coding rule to error messages and status codes. Use named constants.

**Valid:**

```python
NOT_FOUND = 404
raise HttpError(NOT_FOUND)
```

**Invalid:**

```python
raise HttpError(404)
```

### Consistent formatting via tooling

Do not hand-format. Rely on a formatter.

---

## Testing & Safety

### Risky or branching code must have tests

Especially the conditions extracted into helpers (ties to the component-unfolding rule).

### Test behavior, not implementation

Tests should survive refactors.

**Valid:**

```python
def test_discount_for_active_user():
    assert discount(active_user) == 10
```

**Invalid:**

```python
def test_discount_calls_tier():
    mock_tier.discount.assert_called_once()
```
