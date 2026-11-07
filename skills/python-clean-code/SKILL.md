---
name: python-clean-code
description: "Senior Python craft: code another engineer can change safely. Applies single responsibility, intention-revealing names, DRY only for the same rule, KISS over new frameworks, explicit module boundaries, typed public APIs, failures as exceptions, and behavior-named tests. Use when creating, editing, refactoring, or reviewing Python — functions, modules, packages, CLIs, scripts, services, FastAPI, Django, or pytest — including implement, fix, clean up, and Arabic requests such as كود نظيف or قابل للصيانة."
---

# Python Clean Code

You are the senior Python engineer who will maintain this code in six months, along with a stranger who cannot ask you. Write the version that stranger can change safely.

Apply these rules. Do not recite them. When a cut is non-obvious, one sentence of why is enough.

This skill governs how Python is cut, named, typed, and tested. Follow the project's framework, formatter, and type checker. Do not introduce a second one.

Identifiers stay English. User-visible strings follow the project's locale.

## Procedure

1. Read the surrounding package first: layout, error types, typing style, tests, and `pyproject.toml`. Match them.
2. Name each new function or type's one responsibility in a single sentence before writing it. If the sentence needs "and", split it.
3. Design the public signature first. Keep everything else `_private`.
4. Read [references/examples.md](references/examples.md) before adding a public function, module, type, CLI, service, or test, and before introducing a helper, util, base class, metaclass, or generic.
5. Write the smallest clear version. Run the self-review and fix every failure before you stop.

On existing code, match the file. Improve what you touch. Do not reformat or restructure unrelated modules in the same change. When the local pattern is harmful — a bare `except`, a `dict` leaking inward, I/O at import — stop that leak at the boundary you are editing.

A script stays a script: one wiring function and a few pure functions. A package appears when a second concept needs a home.

When you do create a package:

```
src/invoicing/
  __init__.py
  invoice.py
  approval.py
  store.py
tests/
  test_approval.py
```

`__init__.py` exports the names callers actually use. A one-file tool stays one file until a second concept appears.

## Single responsibility

A unit has one reason to change.

- A pure function computes one rule at one level of abstraction.
- A function that both decides and performs I/O is two functions, plus a thin composer when a caller needs both.
- A class exists to protect an invariant, or to be the project's configured boundary (client, repository, session). A bag of fields is a frozen dataclass, or the schema library the project already uses.
- A module groups one concept. Split it when a second concept appears.

A straight function of about forty lines may stay when the steps are one abstraction level. Split when the reader must hold a policy and an I/O protocol in the same function.

## Names

- Functions and modules are `snake_case`. Classes are `PascalCase`. Module constants are `UPPER_SNAKE`.
- Functions are verbs with an outcome: `revoke_access`, `parse_invoice`.
- Booleans read as questions: `is_overdue`, `has_unsaved_changes`.
- Locals may be short. Public names may not. `data`, `info`, `item`, `temp`, `obj`, and `result` do not escape the function that creates them.
- A name of `helper`, `manager`, `utils`, or `common` means the cut is wrong.

## DRY, with a brake

Extract a domain rule when a second caller in the same package depends on it. A similarity that will change for a different reason stays duplicated. By the third copy of the same rule, extraction is overdue.

Four duplicated lines are cheaper than a helper whose parameters are flags.

Name the extracted rule (`eligible_for_refund`), not the resemblance (`check_common`).

Keep the extracted function in the module that owns the rule. Move it to a shared module only when a second package depends on that same rule. Do not add `utils.py` or `helpers.py`.

## KISS

Use the standard library and the libraries already in the project. A new framework, metaclass, decorator stack, or generic needs more than one caller.

Three plain lines beat a nested comprehension the next reader must decode. Delete dead code.

A boolean parameter that switches behavior is two functions. More than four parameters means the function does too much, or the parameters are one typed object. `*args, **kwargs` is for forwarding to a wrapped callable.

Prefer functions. Inherit only where the project already models a real subtype. An ABC or Protocol arrives with the second implementation, or with the first test double that would otherwise be awkward.

## Modularity

A module does not touch the network, the environment, argv, or logging configuration at import time. `main` or the framework's composition point is where configuration is read and objects are wired.

Dependencies point inward. Domain functions do not import the CLI, the web framework, or a database driver. The edge adapts those and calls in.

On a package other code imports, `__all__` is the public surface. Everything else is `_prefixed`. Do not export a symbol for a caller that does not exist yet.

Pass clients, clocks, and paths in. Where the project already uses a global, read it at the edge and pass the value down.

## Data and errors

Parse untyped input into a real type at the edge. `dict[str, Any]` does not travel inward.

A closed set of strings is `enum.Enum` or `Literal`.

`None` means absence. A broken contract or a failed I/O call raises.

Give the package one base exception and the specific errors callers must handle. The message says what failed and which input. At a boundary, `raise SpecificError(...) from err`.

Catch `Exception` only at a process edge (CLI, request handler, worker loop), where you log and translate. `except:`, `except Exception: return None`, and `pass` inside an except are defects.

## Language

- Public functions have type hints, and so do locals whose type is not obvious. Write `list[str]` and `X | None`. Use `Optional` and `List` only in a file that already uses them. `Any` is allowed at the boundary you narrow immediately.
- Default arguments are immutable. Use `None` and create the list or dict inside the function.
- Files, clients, locks, and sessions are context managers.
- New filesystem code uses `pathlib.Path`.
- Instants that are stored or compared across machines are timezone-aware UTC: `datetime.now(timezone.utc)`.
- Library and service code logs through `logging.getLogger(__name__)`. A CLI prints its intended output. Library code does not `print`.
- A public function whose name is not enough gets one docstring line (PEP 257). A comment records a constraint. Delete a comment that restates the next line.
- Format and lint with the project's tool. When you create the project, use ruff to format and lint, with the default rules plus `B` and `UP`. Do not also add black. Turn on the project's type checker for the packages you add.

## Tests

Use the project's runner. A new project uses pytest.

One behavior per test. The name is the specification: `test_approval_required_when_total_exceeds_limit`.

Arrange, act, and assert are visually separate. Construct collaborators in the test. A fixture provides an expensive fake, not a hidden global.

Parametrize cases of one behavior. Separate behaviors are separate tests.

Pass the clock and I/O in. Unit tests do not open network connections. Fake a boundary you do not own. Do not mock the unit under test.

## Self-review

Fix these before you finish.

- The module name and the public signature tell a stranger what it is for.
- No new function both decides a rule and performs I/O.
- No class exists without an invariant, unless it is a dataclass or the project's schema type.
- No helper has a single caller and a vaguer name than the inlined code.
- No `dict[str, Any]` past the edge. No mutable default. No bare `except`. No I/O at import.
- Absence, empty, and failure are distinct.
- Each test names a behavior and would fail if that behavior broke.
- Where this skill and the file disagree, the file wins, except for the harmful patterns called out above.

## When the user wants a review

Report findings a senior would still raise after the formatter ran. For each: the symbol, why the next change is risky, and the cut to make. Lead with the finding that most threatens future edits. Leave the code in place unless the user asked for the change.
