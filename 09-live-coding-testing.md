# Live Coding: Car Class with TDD

## Goal
Build a `Car` class entirely via **Test-Driven Development**. Three iterations, **each with a full 🔴 RED → 🟢 GREEN → 🔵 REFACTOR cycle**.

## Requirements

Before writing any code, list what the `Car` needs to do. Kent Beck calls this a **test list** — tick each one off as you go.

1. ☐ A new car starts with **speed = 0**
2. ☐ The user **can set** the speed
3. ☐ Speed **cannot be negative** — raise `ValueError`

> **Key point:** Writing out the requirements up front is not the same as writing all the tests up front. The discipline is: **pick one, drive it to green, refactor, cross it off, repeat**. Never two at once.

## Setup

We'll use Python's built-in `unittest` framework — no installation needed, ships with Python.

Two files, side-by-side:
- `script.py` — production code (the `Car` class)
- `test_script.py` — tests

Run the tests with:

```bash
python -m unittest test_script.py
```

---

## Iteration 1: Speed starts at zero

> Tackling requirement **#1** from the list.

## Step 1: 🔴 RED — write the test first

```python
import unittest
from script import Car

class TestCar(unittest.TestCase):
    def test_speed_at_start(self):
        car = Car()
        self.assertEqual(car.speed, 0)
```

Run it:
```bash
python -m unittest test_script.py
```

### Output:
```
E
======================================================================
ERROR: test_script (unittest.loader._FailedTest.test_script)
----------------------------------------------------------------------
ImportError: cannot import name 'Car' from 'script'
----------------------------------------------------------------------
Ran 1 test in 0.000s

FAILED (errors=1)
```

> **Discussion:** The test fails because `Car` doesn't exist yet. **That's the point** — a failing test defines what "done" looks like. We now have a clear target.

## Step 2: 🟢 GREEN — minimum code to pass

```python
class Car:
    def __init__(self):
        self.speed = 0
```

### Output:
```
.
----------------------------------------------------------------------
Ran 1 test in 0.000s

OK
```

> **Key point:** Three lines. No validation, no extras. Just enough to pass.

## Step 3: 🔵 REFACTOR — encapsulate the state

Exposing `speed` as a plain attribute means anyone can do `car.speed = "banana"`. Let's hide the internal state behind a property so we control how it's read and (later) how it's written.

```python
class Car:
    def __init__(self):
        self._speed = 0

    @property
    def speed(self):
        return self._speed
```

### Output:
```
.
----------------------------------------------------------------------
Ran 1 test in 0.000s

OK
```

> **Key point:** The test didn't change. From the outside, `car.speed` looks the same — it's still readable. But we've hidden `_speed` behind a property getter. **Encapsulation added, behavior preserved** — that's refactoring.
>
> Notice something: we broke the ability to *write* `car.speed`. A plain attribute was readable AND writable; a property with only a getter is **read-only**. We'll discover that consequence in the next iteration.

---

## Iteration 2: User can set speed

> Requirement #1 ✅ done. Tackling requirement **#2**.

## Step 4: 🔴 RED — write the test

Add a new test method to the `TestCar` class:

```python
class TestCar(unittest.TestCase):
    def test_speed_at_start(self):
        car = Car()
        self.assertEqual(car.speed, 0)

    def test_user_is_able_to_set_speed(self):
        car = Car()
        car.speed = 100
        self.assertEqual(car.speed, 100)
```

Run it:
```bash
python -m unittest test_script.py
```

### Output:
```
.E
======================================================================
ERROR: test_user_is_able_to_set_speed (test_script.TestCar)
----------------------------------------------------------------------
AttributeError: property 'speed' of 'Car' object has no setter
----------------------------------------------------------------------
Ran 2 tests in 0.001s

FAILED (errors=1)
```

> **Discussion:** The previous refactor **had consequences**. Making `speed` a property without a setter made it read-only. The new test surfaces that regression instantly.
>
> This is TDD's safety net in action: a change that would have gone unnoticed in a scriptless codebase shows up immediately with a clear failure message.

## Step 5: 🟢 GREEN — add a setter

```python
class Car:
    def __init__(self):
        self._speed = 0

    @property
    def speed(self):
        return self._speed

    @speed.setter
    def speed(self, value):
        self._speed = value
```

### Output:
```
..
----------------------------------------------------------------------
Ran 2 tests in 0.000s

OK
```

> **Key point:** The setter just stores the value — no validation yet, because no test requires it. TDD's "pay as you go" discipline.

## Step 6: 🔵 REFACTOR — extract setUp

Both tests start with `car = Car()`. That's duplication. `unittest` provides a `setUp` method that runs before every test — use it:

```python
import unittest
from script import Car

class TestCar(unittest.TestCase):
    def setUp(self):
        self.car = Car()

    def test_speed_at_start(self):
        self.assertEqual(self.car.speed, 0)

    def test_user_is_able_to_set_speed(self):
        self.car.speed = 100
        self.assertEqual(self.car.speed, 100)
```

### Output:
```
..
----------------------------------------------------------------------
Ran 2 tests in 0.000s

OK
```

> **Key point:** `setUp` runs once **before each test**, giving every test method a **fresh, isolated** `Car` instance (`self.car`). Setup duplication gone, tests still green.

---

## Iteration 3: Speed cannot be negative

> Requirements #1 and #2 ✅ done. Tackling the last one: **#3**.

## Step 7: 🔴 RED — test for invalid input

```python
def test_speed_cannot_be_negative(self):
    with self.assertRaises(ValueError):
        self.car.speed = -100
```

Run it:
```bash
python -m unittest test_script.py
```

### Output:
```
..F
======================================================================
FAIL: test_speed_cannot_be_negative (test_script.TestCar)
----------------------------------------------------------------------
AssertionError: ValueError not raised
----------------------------------------------------------------------
Ran 3 tests in 0.001s

FAILED (failures=1)
```

> **Discussion:** The test fails — our setter silently accepts `-100`. We need validation.

## Step 8: 🟢 GREEN — add validation

```python
@speed.setter
def speed(self, value):
    if value < 0:
        raise ValueError("Speed cannot be negative")
    self._speed = value
```

### Output:
```
...
----------------------------------------------------------------------
Ran 3 tests in 0.000s

OK
```

> **Key point:** Validation lives where it belongs — in the setter. No other code in the class had to change. That's the payoff of the encapsulation refactor from Step 3.

## Step 9: 🔵 REFACTOR — stronger privacy + type hints

```python
class Car:
    def __init__(self) -> None:
        self.__speed: int = 0

    @property
    def speed(self) -> int:
        return self.__speed

    @speed.setter
    def speed(self, value: int) -> None:
        if value < 0:
            raise ValueError("Speed cannot be negative")
        self.__speed = value
```

### Output:
```
...
----------------------------------------------------------------------
Ran 3 tests in 0.000s

OK
```

> **Key point:** Double-underscore name mangling (`__speed`) makes the internal state harder to bypass. Type hints document intent. All three tests still pass — proof the refactor was safe.

---

## The TDD Rhythm

Every iteration looked the same:

| Phase | What you do | How you know you're done |
|---|---|---|
| 🔴 **RED** | Write **one failing test** | You see a real failure message |
| 🟢 **GREEN** | Write the **minimum code** to pass | Test runner shows all green |
| 🔵 **REFACTOR** | Clean up **without changing behavior** | Tests are still green AND the code reads well |

Nine steps. Three complete cycles. One clean `Car` class — every line of it backed by a test.

## Discussion Points

- **Why refactor in Iteration 1 already?** Because every RED→GREEN leaves the code minimal and a bit rough. Refactoring immediately keeps the codebase clean and teaches the full discipline from the first cycle.
- **Why write the test BEFORE the code?** It defines "done" up front, proves the test actually catches the absence of the feature, and forces you to think about the interface before the implementation.
- **What did Step 3's refactor cost us?** The ability to *write* `car.speed`. Step 4 proved it. This is a gentle lesson: refactors have consequences; tests make them visible.
- **Why didn't we need to change the tests in Step 8?** Because we used a `property`. From the outside, `car.speed = -100` looks the same whether validation is there or not — it just now raises. That's encapsulation paying off.
- **When should you skip TDD?** Spike/prototype code, UI exploration, throwaway scripts — anywhere you don't yet know the interface. TDD assumes you can express the desired behavior up front.
- **The REFACTOR step is the most-skipped — why?** It feels unproductive ("the tests already pass"). But skipping refactors is how codebases rot. Tests exist precisely to make refactoring safe.
