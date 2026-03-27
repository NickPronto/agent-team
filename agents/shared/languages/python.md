# Python Standards

## Version & Setup
- Python 3.10+
- Virtual environment required — never install to system Python
- `pyproject.toml` or `requirements.txt` for deps

## Type Hints
- **Required on all function signatures** — parameters and return type
- Use `from __future__ import annotations` for forward references
- `Optional[T]` → prefer `T | None` (Python 3.10+ union syntax)
- Use `TypedDict` for dict shapes passed between functions
- Use `dataclasses.dataclass` for structured data, not plain dicts

```python
# Good
@dataclass
class DeviceConfig:
    device_id: str
    location: str
    park_hours: int
    persistent: bool = False

# Bad
def process(config: dict) -> dict:  # no type info
    ...
```

## Style
- f-strings for all string formatting — no `.format()` or `%`
- `pathlib.Path` over `os.path` for file operations
- `logging` module over `print()` — configure at module level
- No bare `except:` — always catch specific exceptions (`except ValueError as e:`)
- Context managers (`with`) for file handles, locks, serial ports

## Async (Device Daemon)
- `asyncio` for concurrent I/O in the daemon
- `async def` / `await` consistently — no mixing sync blocking calls in async context
- Use `asyncio.create_task()` for fire-and-forget background work
- Serial comms: non-blocking reads with timeout, never `while True: read()` without sleep

## Module Layout
```
module/
  __init__.py
  config.py       # config loading + dataclass
  main.py         # entry point
  <feature>.py    # one concern per file
```

## Device Daemon Specifics
- Config loaded from `config.json` at startup — use `load_config()` pattern
- Sensitive values (API keys) read from a protected file path, never from `config.json`
- GPIO and serial access: always release in `finally` blocks
- Log level: `INFO` default, `DEBUG` only behind a flag

## Error Handling
- Daemon processes must not crash on transient errors — catch, log, continue
- Fatal errors (missing config, hardware not found): log clearly and `sys.exit(1)`
- Never silence `Exception` without at minimum logging it
