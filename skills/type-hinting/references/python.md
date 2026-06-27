# Python Type Hints

## Modern Syntax (Python 3.10+)

Python's typing has evolved rapidly. Prefer modern syntax when the project supports it (3.9+ for built-in generics, 3.10+ for union syntax).

| Old | New (prefer) | Since |
|---|---|---|
| `from typing import List, Dict` | `list[str]`, `dict[str, int]` | 3.9 |
| `Union[str, int]` | `str \| int` | 3.10 |
| `Optional[str]` | `str \| None` | 3.10 |
| `from typing import Type` | `type[MyClass]` | 3.9 |
| `from typing import Tuple` | `tuple[int, str]` | 3.9 |
| `TypeVar('T', bound=X)` | Can still use TypeVar, but `TypeVar` with `bound=` is the same | — |

**`Self` type (3.11+):** Use `Self` for return types that return `self` or `cls`:

```python
from typing import Self

class Builder:
    def with_name(self, name: str) -> Self:
        self.name = name
        return self
```

**`TypeAlias` (3.10+):** Name complex types for readability:

```python
from typing import TypeAlias

JSON: TypeAlias = dict[str, "JSON | str | int | float | bool | list[JSON] | None"]
```

---

## Common Patterns

### TypedDict — Structured Dictionaries

For dictionaries with a known shape, use `TypedDict` instead of `dict[str, Any]`:

```python
from typing import TypedDict, NotRequired

class UserDict(TypedDict):
    id: int
    name: str
    email: NotRequired[str | None]  # 3.11+
```

This gives editor autocomplete, type checking on dict literals, and validation of field access. Use it for JSON responses, config dicts, and any dict where the keys are known at write time.

### Protocol — Duck Typing with Safety

`Protocol` lets you define structural types — "anything with these methods/attributes" — without requiring inheritance:

```python
from typing import Protocol

class Drawable(Protocol):
    def draw(self) -> None: ...

def render(obj: Drawable) -> None:
    obj.draw()
```

Any object with a `draw` method satisfies `Drawable`, no inheritance needed. This is ideal for:
- Plugin/strategy patterns
- Testing (pass mock objects)
- Decoupling callers from concrete classes

### dataclass — Typed Data Containers

```python
from dataclasses import dataclass, field

@dataclass
class Config:
    host: str
    port: int = 8080
    retries: int = field(default=3, metadata={"desc": "Number of retries"})
```

Use `InitVar` for fields available at init but not stored, and `field(default_factory=...)` for mutable defaults (never use `mutable=[]` as a default).

### Generics and TypeVar

```python
from typing import TypeVar, Generic, Sequence

T = TypeVar("T")

class Stack(Generic[T]):
    def push(self, item: T) -> None: ...
    def pop(self) -> T: ...

def first(items: Sequence[T]) -> T:
    return items[0]
```

Use `TypeVar` constraints when the type must satisfy a protocol:

```python
from typing import TypeVar
from collections.abc import Hashable

H = TypeVar("H", bound=Hashable)

def deduplicate(items: list[H]) -> list[H]:
    return list(dict.fromkeys(items))
```

### Overload — Complex Signatures

When a function's return type depends on its arguments in ways unions can't express:

```python
from typing import overload

@overload
def load(path: str) -> str: ...
@overload
def load(path: bytes) -> bytes: ...

def load(path: str | bytes) -> str | bytes:
    if isinstance(path, str):
        return path
    return path.decode()
```

### Literal — Constrained Values

```python
from typing import Literal

def set_mode(mode: Literal["read", "write", "append"]) -> None: ...
```

Use this for flags, options dicts, and configuration values. The type checker will flag invalid values.

### Final and ClassVar

```python
from typing import Final, ClassVar

MAX_RETRIES: Final = 3  # Cannot be reassigned

class Database:
    connection_pool: ClassVar[list[Connection]] = []  # Shared across instances
```

---

## Gradual Typing Strategies

### Starting from Untyped Code

1. **Enable mypy/pyright in lenient mode** on the whole project to see what you're dealing with
2. **Run the checker on one file at a time** to avoid overwhelming error counts
3. **Start with data structures:** add TypedDicts, dataclass types, and class field annotations. These are usually self-contained and high-value.
4. **Function signatures next:** add parameter and return types to public functions. Start with the most-called functions — the ripple effect catches more callers.
5. **Add `--strict` flags incrementally:** `--disallow-untyped-defs` first, then `--warn-return-any`, then `--strict-equality`, etc.

### Using Stub Files

For third-party libraries without type stubs, you have options:
- **typeshed:** Install `types-<package>` from PyPI for common libs (e.g., `types-requests`, `types-Pillow`)
- **Inline stubs:** Write a `.pyi` file alongside the library for the types you need
- **`# type: ignore`** on imports as a last resort

### Handling Dynamic Features

Python's dynamic features (metaclasses, `__getattr__`, `eval`, `exec`) are hard to type. Strategies:
- **Protocols with `@runtime_checkable`** when you need both type checking and runtime validation
- **`Any` on the boundary:** use `Any` at the interface between typed and untyped code, with clear documentation
- **`cast()` for known invariants:** `cast(User, obj)` when you know something the type checker doesn't

---

## Tooling

### mypy

```ini
# mypy.ini
[mypy]
strict = True
python_version = 3.11
exclude = (build/|dist/|tests/)
```

Key flags:
- `--strict` — enables all strict options (don't start here on a brownfield project)
- `--disallow-untyped-defs` — requires annotations on all functions
- `--check-untyped-defs` — checks bodies of functions without annotations (less strict than above)
- `--warn-return-any` — warns when a function returns `Any` (catches missing annotations)
- `--strict-equality` — flags comparisons between incompatible types
- `--ignore-missing-imports` — suppress errors for packages without stubs

### pyright (used by VS Code's Pylance)

```json
{
  "typeCheckingMode": "standard",
  "reportMissingTypeStubs": "warning",
  "reportUnknownMemberType": "none"
}
```

pyright is generally faster than mypy and integrates natively with VS Code. It catches some things mypy misses (and vice versa). Key differences:
- pyright infers more aggressively
- pyright handles conditional types (isinstance narrowing) better in some edge cases
- mypy has better documentation and more config flags

### Pre-commit Hooks

```yaml
# .pre-commit-config.yaml
repos:
  - repo: https://github.com/pre-commit/mirrors-mypy
    rev: v1.8.0
    hooks:
      - id: mypy
        args: [--strict]
```

---

## Anti-Patterns

### Over-Parameterized Generics

```python
# Hard to read and maintain
def transform[A, B, C, D](data: dict[A, list[tuple[B, C]]]) -> Generator[D, None, None]: ...
```

If you need 4+ type parameters, the data structure is probably doing too much.

### Recursive Types Without Forward References

```python
from typing import TypeAlias
JSON: TypeAlias = dict[str, "JSON | str | int | float | bool | list[JSON] | None"]
```

The quotes around `"JSON"` are a forward reference. Without them, the name isn't defined yet at evaluation time. Always quote forward references to recursive type aliases.

### `Any` Everywhere

`Any` is contagious — once a value is `Any`, the type checker stops checking everything that touches it. A single `Any` return from a library function can silence type checking across your entire call chain.
