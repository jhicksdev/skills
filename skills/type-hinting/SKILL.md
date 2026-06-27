---
name: type-hinting
description: >-
  Principles and best practices for type hinting in dynamically-typed languages
  (Python, JavaScript/TypeScript). Use this whenever the user asks about type
  annotations, type hints, type checking, mypy, pyright, JSDoc types, TypeScript
  types, gradual typing, adding type annotations to existing code, static
  analysis configuration for types, or fixing type errors. Also use it when
  the user is designing new functions/APIs and you want to annotate them well,
  when refactoring untyped code toward typed code, or when the user's type
  checker is producing confusing errors they need help understanding. Do NOT
  use this for explaining TypeScript's type system in a TypeScript project
  (that's just using TypeScript normally) — use it when the user is working in
  a traditionally dynamic context (Python, plain JS with JSDoc, JS-to-TS
  migration) or when they explicitly ask about typing strategy and conventions.
---

# Type Hinting

Type hints in dynamic languages serve two roles: **documentation that machines can verify** and **contracts between callers and implementations**. A well-annotated codebase catches entire classes of bugs at analysis time, makes interfaces self-documenting, and gives editors superpowers (autocomplete, go-to-definition, refactoring tools). A poorly-annotated one creates noise, false confidence, and maintenance burden.

This skill covers Python and JavaScript. The core principles are shared; the language-specific mechanics live in the reference files.

## Language Selector

- **Working in Python?** → Read `references/python.md`
- **Working in JavaScript or TypeScript?** → Read `references/javascript.md`

Read the appropriate reference now, then return here for the shared principles and anti-patterns.

---

## Core Principles

### Types Are Contracts, Not Grades

Every annotation is a promise. `def load_user(id: int) -> User` says: "give me an int, I'll give you a User." If the function actually accepts a string, or might return None, the annotation is a lie. Lies in type annotations are worse than no annotations — they mislead both humans and tools.

Ask yourself: "If someone reads this signature, will their assumption about how to call it match what the code actually does?"

### Annotate Public APIs First, Internals Second

The highest-value targets are:
1. **Public function/method signatures** — parameters and return types
2. **Data structures** — dataclass fields, TypedDicts, interfaces
3. **Module-level constants and configuration**

Internal helper functions, local variables, and loop counters benefit least from annotations. Let the type checker infer those; focus your annotation effort on the boundaries between components.

### Prefer Specificity, but Not at the Cost of Readability

A maximally precise type that takes 30 seconds to parse is worse than a 90%-correct type that communicates intent instantly. For example:

```python
# Overly precise — hard to read
def process(items: dict[str, list[tuple[int, str, float | None]]]) -> list[dict[str, Any]]: ...

# Better — still precise, but uses aliases
ItemData = list[tuple[int, str, float | None]]
def process(items: dict[str, ItemData]) -> list[dict[str, Any]]: ...
```

If a type feels too complex to write, it's probably too complex to maintain. Consider whether the data structure itself needs simplification.

### Let the Type Checker Do the Heavy Lifting

Don't annotate what the type checker can infer. Letting the checker infer local variables and intermediate values means:
- Less code to maintain
- Fewer opportunities for annotation drift
- The checker validates your actual implementation, not your stated intent

Annotations on function signatures are non-negotiable (those are the public contract). Annotations on local variables are usually noise.

### Prefer Structural Over Nominal Typing

When you need to define what a value looks like, prefer structural types (Protocol in Python, interface in TypeScript) over inheritance-based types. This makes your code more flexible and easier to test — callers don't need to know about your class hierarchy, only what shape of value you expect.

### Understand Your Tool's Strictness Level

Every type checker has a dial from "barely checking" to "draconian." Know where your project sits:
- **Python:** mypy has `--strict` which enables every flag. Most projects use something between `--check-untyped-defs` and full strict. pyright has `typeCheckingMode: strict|standard|off`.
- **JavaScript (JSDoc):** TypeScript's `checkJs: true` with `strict: true` is the max; `allowJs: true` with no `checkJs` does nothing.
- **TypeScript:** `strict: true` in tsconfig enables the full set of strict flags.

Match the strictness to your project's maturity and tolerance for type-error noise. A project starting from zero types should begin with lenient settings and tighten over time.

---

## Common Anti-Patterns (Both Languages)

### Lying to the Type Checker

The most common and damaging anti-pattern. Examples:
- A function annotated as `-> int` that raises an exception instead of returning (the annotation should be `-> int | None` or include a `raise` in the signature)
- A `@param {string}` JSDoc that can actually receive `null`
- Using `Any`/`any` as a blanket type for a parameter that has a known shape

**Fix:** Think of the annotation as a test that must pass for every code path. If a function sometimes returns None, say so. If a parameter can be multiple types, use a union.

### Over-Annotating Internals

Every unnecessary annotation is a liability when the code changes. The type checker will infer types for local variables, comprehensions, and intermediate values — let it.

### The `Any` Escape Hatch Habit

Using `Any` (Python) or `any` (TypeScript/JSDoc) to silence the type checker is like turning off a smoke alarm because it's annoying. You're not fixing the problem; you're blinding yourself to it.

When you're tempted to use `Any`, try:
- `unknown` instead — it forces a type assertion/check before use
- A more specific type, even if it's a broad union
- A `# type: ignore` / `@ts-expect-error` with a comment explaining why (this at least documents the decision)

### Copying Types from Elsewhere Without Understanding

Copying a TypedDict definition from Stack Overflow or a type annotation from a similar function without understanding the actual data shape creates silent bugs. The type checker trusts your annotations — it won't tell you they're wrong if they're internally consistent.

### Overly Broad Types on Public APIs

`dict[str, Any]` as a return type tells callers almost nothing. A TypedDict, dataclass, or interface communicates the actual structure. The extra verbosity pays for itself on the first use of the function.

### Ignoring Type Checker Errors

Adding `# type: ignore` without a comment, or passing `--no-strict-optional` to make the noise go away, hides real bugs. Each suppressed error is a bug that will surface at runtime, probably at the worst possible moment.

---

## What to Do When Confused

1. **Read the type checker's error message carefully.** Most modern type checkers (mypy, pyright, TypeScript) give detailed error messages that explain exactly what they expected and what they got.
2. **Isolate the problem.** Strip away everything non-essential. Can you reproduce with a simpler example?
3. **Check the obvious:** Did you import the type? Is it in scope? Are you using the right version of the typing module (Python 3.9 vs 3.10+)?
4. **Ask: "Am I fighting the type checker or is the type checker finding a real issue?"** Most of the time it's the latter.

---

## Reference Files

- `references/python.md` — Python type hints: syntax, patterns, tooling, gradual typing strategies
- `references/javascript.md` — JavaScript type hints: JSDoc annotations, TypeScript patterns, hybrid JS/TS workflows
