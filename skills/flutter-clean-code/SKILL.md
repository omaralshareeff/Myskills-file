---
name: flutter-clean-code
description: "Senior Flutter and Dart craft: code another engineer can change safely. Applies single responsibility (widgets paint, state decides, pure functions hold rules), intention-revealing names, DRY only for the same rule, KISS over new abstractions, private-by-default modules, exhaustive sealed state, sound null safety, and behavior-named tests. Use when creating, editing, refactoring, or reviewing any Dart or Flutter code — widgets, state (Riverpod, Bloc, Provider, ChangeNotifier), models, repositories, services, or tests — including implement, fix, clean up, and Arabic requests such as كود نظيف or قابل للصيانة."
---

# Flutter Clean Code

You are the senior Flutter engineer who will maintain this code in six months, along with a stranger who cannot ask you. Write the version that stranger can change safely.

Apply these rules. Do not recite them. When a cut is non-obvious, one sentence of why is enough.

This skill governs how Dart is cut, named, and tested. Follow the project's architecture and state-management library. Do not introduce a second pattern, and do not add an empty layer (a use case that only forwards to a repository, a base class with one implementation).

Identifiers stay English. User-visible strings follow the project's locale.

## Procedure

1. Read the surrounding feature first: folder layout, state style, error style, test style, and `analysis_options.yaml`. Match them.
2. Name each new type's one responsibility in a single sentence before writing it. If the sentence needs "and", split the type.
3. Design the public surface first (widget constructor, function signature, repository method). Keep everything else private.
4. Read [references/examples.md](references/examples.md) before adding a widget, state type, public function, repository, or test, and before introducing a helper, util, base class, mixin, or generic.
5. Write the smallest clear version. Run the self-review and fix every failure before you stop.

On existing code, match the file. Improve what you touch. Do not reformat or restructure unrelated files in the same change. When the local pattern is harmful — swallowed errors, `dynamic` leaking inward, a business rule inside `build` — stop that leak at the boundary you are editing.

When the project has no structure yet and the change is a real feature, use one folder per feature:

```
lib/features/refund/
  refund_eligibility.dart
  refund_controller.dart
  refund_page.dart
  refund_repository.dart
```

Name the files after the concept. Omit a file the feature does not need. A one-widget change stays a one-widget change.

## Single responsibility

A unit has one reason to change: one policy, one I/O boundary, or one presentation.

- A widget paints. It reads ready state and composes children.
- The project's state holder decides transitions.
- A repository or client talks to the outside world.
- A pure function computes a business rule. Inputs in, value out. No `BuildContext`, clock, or network inside the rule.

`build` does not parse, apply policy, call repositories, or branch on business rules. Pass the widget the values it paints. Pass the whole value when it paints several of its fields. Pass a decision separately when the widget must not know how that decision was made.

UI state (tab index, text controller) stays next to the widget. Business state does not.

If you must scroll through `build` to understand it, extract a child widget. Split a file when a second concept appears. A straight function of about forty lines may stay when splitting it would force the reader to jump.

## Names

Name the domain concept.

- Types are noun phrases. The file is `snake_case.dart` and matches the public type: `RefundEligibility` in `refund_eligibility.dart`. One public type per file. Private types used only there may share the file.
- Functions are verbs with an outcome: `revokeAccess`, `formatExpiry`.
- Booleans read as questions: `isOverdue`, `hasUnsavedChanges`.
- Locals may be short. Public names may not. `data`, `info`, `item`, `temp`, `obj`, and `result` do not escape the function that creates them.
- A name of `Helper`, `Manager`, `Utils`, or `Common` means the cut is wrong. Give it a domain name, or inline it.

## DRY, with a brake

Extract a domain rule when a second caller in the same feature depends on it. A similarity that will change for a different reason stays duplicated. By the third copy of the same rule, extraction is overdue.

Four duplicated lines are cheaper than a helper with a flag.

Name the extracted rule (`eligibleForRefund`), not the resemblance (`checkCommon`).

Keep the extracted rule inside the feature. Move it to a shared folder only when a second feature depends on that same rule. Do not add `utils.dart`, `helpers.dart`, or `common.dart`.

## KISS

Use the language and the libraries already in the project. A new package, code generator, base class, mixin, or generic needs more than one caller.

Three plain lines beat a cascade the next reader must decode. Delete dead code.

A boolean parameter that switches behavior is two functions or two widgets.

Add freezed, json_serializable, or a new state library only when the project already depends on them. A hand-written immutable type is the default.

## Modularity

Dependencies point one way. Widgets depend on state. State depends on pure rules and on the repository. Domain code imports neither Flutter widgets nor another feature's files.

Introduce an interface when a test needs a fake or a second implementation exists. A single concrete repository is enough until then.

Prefix private members with `_`. Do not make a symbol public for a caller that does not exist yet.

Pass dependencies through constructors. A new singleton or service locator is a defect. Where the project already has a composition root, resolve the dependency there and pass the object down.

## State and errors

Represent screen state as one sealed type. Parallel flags (`isLoading`, `hasError`, `data == null`) drift apart; do not add another set.

Switch on that type exhaustively. A `default` or `_` arm that hides a new case is a defect. A named fallback is fine when the fallback is a real product decision.

Use the project's error style. Throwing projects throw a typed exception and map it to UI state at the edge. Result-type projects return the result. An empty `catch` is a defect. Translate a failure at the boundary; do not drop it.

## Dart

- `const` constructors and `const` children wherever the analyzer allows.
- `final` fields and `final` locals. Models are immutable.
- `dynamic` stays out of new code. Parse `Map<String, dynamic>` into a type at the JSON or platform boundary. The map does not travel inward.
- Restructure nulls so `!` is unnecessary. A local `!` is allowed only when the immediately previous line established the invariant.
- Guard clauses first. The happy path stays at the bottom, unindented.
- `await` in new code. After an `await`, use `BuildContext` only when the element is still mounted, or via the cancellation pattern the project already uses.
- A list whose items can move gets a `ValueKey` of the stable id.
- Durations, storage keys, and route names are named constants. A one-off padding literal is fine when the project has no spacing scale.
- Public libraries get one dartdoc sentence. A comment records a constraint the name cannot carry. Delete a comment that restates the next line.
- Format with `dart format`. Honor `analysis_options.yaml`. When you create that file, include `package:flutter_lints/flutter.yaml` and do not weaken it.

Extract a widget class when the subtree has its own parameters or should be `const`. A private method that returns `Widget` is only for a trivial fragment used once.

## Tests

Business rules are unit tests and do not need a Flutter binding. Widget tests cover what the user sees and can do.

One behavior per test. The name is the specification: `eligibleForRefund is false when the window has elapsed`.

Test through the public surface. Promote a private you must test directly into a named concept, or cover it through its caller.

Pass the clock and I/O in. A rule does not call `DateTime.now()`.

Fake a boundary you do not own. Do not mock the unit under test.

## Self-review

Fix these before you finish.

- The file name and the public signature tell a stranger what the type is for.
- No new type has two reasons to change.
- No helper has a single caller and a vaguer name than the inlined code.
- No business rule remains inside `build`, a callback, or a repository.
- Empty, loading, ready, and failed cannot be true together.
- No comment restates the code. No new `dynamic`, empty `catch`, or unjustified `!`.
- Each test names a behavior and would fail if that behavior broke.
- Where this skill and the file disagree, the file wins, except for the harmful patterns called out above.

## When the user wants a review

Report findings a senior would still raise after the formatter ran. For each: the symbol, why the next change is risky, and the cut to make. Lead with the finding that most threatens future edits. Leave the code in place unless the user asked for the change.
