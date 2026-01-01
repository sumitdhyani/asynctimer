# asynctimer

Deterministic, asyncio-native utility for **repeated, fixed-interval execution** , designed for **predictable behavior and correctness**.

## Overview

This library provides **Timer** with explicit drift and overrun semantics
It is intentionally conservative and explicit, making it suitable for protecting external systems (APIs, databases, hardware) where **bursts and ambiguity are unacceptable**.

## Why this library exists

Many timer utilities:
- implicitly drift over time
- allow bursts by default
- rely on best-effort timing
- do not clearly specify stop or overrun behavior

**asynctimer** takes a different approach:

> Behavior is explicit, testable, and deterministic — even if that costs some throughput, though in case of timers that's virtually never a concern.

---

## Installation

```bash
pip install asynctimer
```

---

### What it does

Implements a repeating, async-friendly timer that calls a user-provided callback at a fixed interval until stopped. Callbacks may be synchronous functions or async coroutines.


### Timer Guarantees

- Callbacks never execute concurrently
- Stop semantics are deterministic, i.e once the timer is stopped, the callbacks will not be executher any further
- Drift behavior is explicitly configurable
- Overrun behavior is well-defined and tested


### API

- `Timer(timeout: datetime.timedelta | float, callback: Callable, schedule_policy: SchedulePolicy = SchedulePolicy.FIXED)` — create a timer with a repeat interval, callback, and drift policy.
- `start()` — start the timer loop (non-blocking; schedules repeated executions).
- `stop()` — stop the timer loop.

### Drift / Schedule Policy

Timers can be scheduled in two fundamentally different ways.

#### Fixed Schedule (drift-free)

```
next_tick = initial_start + N × interval
```

- Anchored to the original schedule
- Prevents long-term drift
- Suitable for:
  - heartbeats
  - simulations
  - clock-aligned tasks

```python
from asyncio_utils import Timer, SchedulePolicy
from datetime import timedelta
# callback can be async too
def callback():
    print('tick')

timer = Timer(
    timedelta(seconds=2),
    callback,
    schedule_policy=SchedulePolicy.FIXED
)
timer.start()  # starts the repeating timer
# ... later: timer.stop()
```

#### Fixed Delay (completion-relative)

```
next_tick = callback_completion + interval
```

- Guarantees spacing between executions
- Suitable for:
  - retries
  - polling
  - cooldowns
  - background maintenance

```python
from asyncio_utils import Timer, SchedulePolicy
from datetime import timedelta

timer = Timer(
    timedelta(seconds=2),
    callback,
    schedule_policy=SchedulePolicy.DELAY
)
timer.start()
```

### Overrun Behavior

If a callback takes longer than the interval:
- Missed ticks are not executed retroactively
- The timer resumes according to its original schedule policy

This avoids burst execution and keeps behavior predictable.

### Stop Semantics

Calling `stop()` guarantees:
- No further callbacks will be executed

### Example

```python
import asyncio
from datetime import timedelta
from asyncio_utils import Timer, SchedulePolicy

# callback can be async too
def callback():
    print('tick')

async def main():
    timer = Timer(timedelta(seconds=2), my_async_callback)
    timer.start()  # starts the repeating timer
    await asyncio.sleep(10)
    timer.stop()

asyncio.run(main())
```


## Working Examples

For concrete examples showing actual working code using these classes, see the example code in the `examples/` directory:

- [examples/BasicTimerExample.py](examples/BasicTimerExample.py) — basic `Timer` usage
- [examples/MultipleTimersExample.py](examples/MultipleTimersExample.py) — using multiple `Timer` instances
- [examples/SchedulePolicyTimerExample.py](examples/SchedulePolicyTimerExample.py) — using `SchedulePolicy` with `Timer`

---

## Build Package from Source

The project uses `pyproject.toml`. To build distribution archives install `build` and run:

```bash
pip install --upgrade build
python -m build
```

The above produces a `dist/` folder with `.whl` and `.tar.gz` files. You can also install locally with:

```bash
pip install .        # install from source
pip install -e .     # editable install for development
```

Alternatively, this repository includes `build_package.py`; you can run it if you prefer (it wraps standard build steps):

```bash
python build_package.py
```

---

## Run Tests

The tests are in the `tests/` directory and use `unittest`'s async test support. You can run them with `unittest` or with `pytest`.

Run with unittest (cross-platform):
```bash
python -m unittest discover -v
```

Run with pytest (if installed):
```bash
pip install pytest
pytest -q
```

Platform-specific helper scripts are provided:
- Windows: `run_tests.bat`
- Unix/macOS: `run_tests.sh`

---

## Notes & Troubleshooting

- The utilities depend only on Python's standard library (`asyncio`, `datetime`, etc.). Tests use `unittest.IsolatedAsyncioTestCase` which requires Python 3.8+.
---

## License

MIT
