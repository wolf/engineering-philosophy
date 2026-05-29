# Code Audit Checklist

A systematic checklist for auditing a Python library or codebase.

This checklist is the verification layer for the standards described in:

* [What Matters Most](../../WHAT-MATTERS-MOST.md) — the four principles
* [Standards for Shared Projects](../../STANDARDS-FOR-SHARED-PROJECTS.md) — language-agnostic standards
* [Python Standards](STANDARDS.md) — Python-specific standards

Each section below maps to one or more of the four principles — **Goals**, **No Surprises**, **Honesty**, and **Tests and Measurement** — or to [the standing What](../../WHAT-MATTERS-MOST.md#the-standing-what) beneath them.

Not every standard needs a checklist line: anything the quality gates verify mechanically — formatting, lint rules, type checks, hook presence — is checked by running them. This checklist covers what tools can't judge.

## Inputs

Before auditing code, gather all available context:

* **Captured thoughts** — from whatever capture system you use: notes, braindumps, transcribed ideas, complaints, aspirations
* **Open tickets** — what's already known to be wrong or wanted
* **Inbox items** — accumulated ideas not yet ticketed
* **Client/consumer code** — how the library is actually used in production

## API and Design

*Principles: Goals, No Surprises, Honesty*

* [ ] **Goals.** Why does this code exist? Every decision you make must support these. (See [What Matters Most § Know and State Your Goals](../../WHAT-MATTERS-MOST.md#know-and-state-your-goals).)
* [ ] **No surprises.** Does everything behave as a reasonable caller would expect?
* [ ] **Honesty.** Does documentation acknowledge limitations, weaknesses, and cases where alternatives are better?
* [ ] **Every API justified.** Can you explain why each public function, class, and parameter exists in terms of stated goals?
* [ ] **Right extension points.** Can users extend the library where they need to? Are the seams in the right places?
* [ ] **Correct privacy.** Are private things that should be public? Public things that should be private?
* [ ] **Good names.** Everywhere, for everything. Names that give the caller correct expectations.
* [ ] **Exactly the right tools.** Nothing missing that clients keep reimplementing. Nothing too broad that becomes a promise you don't want to support.
* [ ] **Symmetry.** Load/save, add/delete, import/export, per-row/bulk — when two things should mirror each other, do they? In naming, capability, and discoverability?
* [ ] **Export/import symmetry.** Can you get data out as easily as you put it in?
* [ ] **Reducing ceremony.** Where is the API unnecessarily verbose or repetitive?
* [ ] **`__version__` exposed.** Shared libraries expose `__version__` at the top-level namespace and include it in `__all__`.

## Types and Patterns

*Principle: No Surprises*

* [ ] **Using the right types.** Parameters, return values, class variables — are they typed correctly? (Always use type annotations.)
* [ ] **Type machinery.** Does the code use Generics, Protocols, abstract classes where they would help?
* [ ] **Well-known patterns fully supported.** When the code provides something *like* a Mapping, Set, Collection, or Sequence, does it satisfy the protocol completely? Almost-but-not-quite is a surprise.
* [ ] **Protocol conformance.** Check against `collections.abc` — Mapping, MutableSet, Collection, Sequence, Iterator. Register or inherit where appropriate.
* [ ] **Representation-space awareness.** When the same value exists in two representations (e.g., as the database stores it vs. as Python code uses it), is the boundary explicit in names and parameters?

## Code Quality

*Principles: Goals, Tests and Measurement*

* [ ] **Code density.** Comprehensions over collection-building loops, walrus operator for compute-check-use, compound conditionals over nesting, `next()` for single-item search, yield instead of collect-then-return, data-driven over copy-paste, early return/continue to flatten nesting. (See [Python Standards § Density](STANDARDS.md#density).)
* [ ] **Serial work that could be vectorized.** Row-by-row Python where pandas/numpy operations would be both faster and more idiomatic.
* [ ] **Eager/lazy choice is deliberate.** Returning a list when you computed per-element — should be a generator. Returning a generator that's always fully consumed — that's just overhead. Wrapping a cached collection in a generator — that's hiding information. Each case should be a conscious decision. (See [Python Standards § Eager vs. Lazy](STANDARDS.md#eager-vs-lazy).)
* [ ] **Wrong level / wrong big-O complexity.** Scanning a registry on every mutation when a pre-computed reverse lookup would be O(1).
* [ ] **Unnecessary copies.** Especially DataFrame copies — count how many times the same data is copied through a pipeline. One allocation is the goal.
* [ ] **Benchmark before optimizing.** Capture before-numbers so improvements are measurable.

## Architecture

*Principle: No Surprises*

* [ ] **Layer separation.** Each layer does exactly and only the thing for which it was introduced. No upward or downward references that violate the abstraction.
* [ ] **Class hierarchy.** Do classes inherit from the right things? Are mixins identified as such? Is the exception taxonomy honest? Did you even **need** a class here, or would functions have been enough. Use `@dataclass` when appropriate.
* [ ] **Globals that should be members.** Functions that take `cls` as first arg are classmethods in disguise. Module-level state that belongs on a class.
* [ ] **Magic justified.** Metaclasses, `__init_subclass__`, descriptor injection, automatic name generation — does each piece earn its keep by eliminating real repetition? Is it discoverable when something goes wrong?

## Secrets and Credentials

*The Standing What: asset protection* (See [Handling Secrets](../../HANDLING-SECRETS.md).)

* [ ] **No secrets in source.** Grep for API keys, passwords, tokens, connection strings, private keys. Check string literals, comments, and default parameter values.
* [ ] **No secrets in configuration files.** Check any `.yaml`, `.json`, `.toml`, or `.ini` files committed to the repo. Sample files should have placeholder values, never real credentials.
* [ ] **No secrets in logs.** Trace logging calls — are connection strings, tokens, or credentials passed to any log statement, even at DEBUG level?
* [ ] **No secrets in error messages.** Check exception messages and user-facing output for interpolated credentials or connection details.
* [ ] **Environment-based injection.** Are secrets read from environment variables or a secret manager, not from files in the repo?
* [ ] **`.envrc.sample` has no real values.** The sample shows the shape of each variable without providing working credentials.
* [ ] **`detect-secrets` baseline exists and is current.** The `.secrets.baseline` file should exist and the pre-commit hook should reference it.

## Platform Assumptions

*Principle: No Surprises* (See [Cross-Platform Development](../../CROSS-PLATFORM.md).)

* [ ] **Platform intent is stated.** Does the README declare which platforms the software runs on and which platforms developers can work from?
* [ ] **No hardcoded paths.** Grep for `/tmp`, `/dev/null`, `C:\\`, `%APPDATA%`, `%USERPROFILE%`, `~/.config`, and other platform-specific locations. These should use `platformdirs`, `tempfile`, or `pathlib.Path` equivalents.
* [ ] **No hardcoded path separators.** Look for string concatenation with `/` or `\\` to build file paths. Use `pathlib.Path` instead.
* [ ] **No platform-specific APIs without isolation.** Calls to Win32, COM, `/proc`, macOS Keychain, or similar should be behind an interface with per-platform implementations, not scattered through business logic.
* [ ] **No `shell=True` in subprocess calls.** These tie you to a specific shell's syntax. Use argument lists instead.
* [ ] **Case sensitivity assumptions.** Are there file lookups or comparisons that assume case-insensitive (or case-sensitive) behavior? These will break on a different platform.
* [ ] **Line ending safety.** Does a `.gitattributes` file exist with `* text=auto`?
* [ ] **Entry points over scripts.** Are executables defined as `console_scripts` entry points in `pyproject.toml`, not as shell scripts or batch files?
* [ ] **Environment markers for platform dependencies.** Are platform-specific dependencies in `pyproject.toml` guarded with `sys_platform` or `platform_system` markers?
* [ ] **Tests skip gracefully.** Platform-specific tests use `pytest.mark.skipif` with a reason string, not bare `if` guards that silently pass.

## Error Handling

*Principles: No Surprises, Honesty*

* [ ] **Non-happy paths.** Assert in the right spots (programming errors only, never user input). Return an expected signal when failure is known to be possible. Raise the right exception for things that are genuinely unexpected.
* [ ] **No `assert` for user-facing validation.** Assertions are optimized away with `-O`.
* [ ] **Implicit contracts made explicit.** If code depends on `bool(handle)` working a certain way, that's a contract — test it and document it.

## Documentation

*Principle: Honesty* (See [Standards for Shared Projects § Documentation](../../STANDARDS-FOR-SHARED-PROJECTS.md#documentation).)

* [ ] **Docs tell the truth.** Are there claims that don't match the current codebase?
* [ ] **Code examples work.** Would they actually run against the current API?
* [ ] **Tutorial shows what clients do.** Not what the author thinks they should do — what they actually do, based on reading consumer code.
* [ ] **Missing features documented.** Features in the code that the docs don't mention at all?
* [ ] **Docs promise features that don't exist?**

## Testing

*Principle: Tests and Measurement* (See [Python Standards § Testing](STANDARDS.md#testing).)

* [ ] **Risky/critical code has tests.** The parts where design choices were made — are those choices encoded in tests?
* [ ] **Tests test behavior, not implementation.** Will they break if internals are refactored?
* [ ] **Critical paths annotated in source.** `# CRITICAL:` comments at code that is risky, has broad impact, or where correctness depends on subtle invariants.
* [ ] **Benchmarks verify goals.** If the project has stated performance goals, do benchmarks measure against them?
* [ ] **Coverage is measurable and meets the floor.** `pytest-cov` (or equivalent) is set up, and coverage meets the 80% floor set in [Standards for Shared Projects § Testing](../../STANDARDS-FOR-SHARED-PROJECTS.md#testing).

## Client Analysis

*Principles: Goals, No Surprises*

* [ ] **Read consumer code.** What patterns do clients repeat? What workarounds did they build?
* [ ] **Safe lookup patterns.** Do clients guard every lookup with a membership check? That's a missing `.get()`.
* [ ] **Direct internal access.** Are clients reaching into private attributes? That's a missing public API.
* [ ] **Compatibility bridges.** Are clients writing properties just to map old names to new names? That's migration friction.
* [ ] **Boilerplate in load/save methods.** The same 5 lines in every consumer's `load_default` — that's a missing building block.

## Process

*All four principles*

1. **Gather inputs** — notes, tickets, inbox, consumer code
2. **Audit code** module by module, noting observations
3. **Merge observations** into a consolidated list
4. **Trim** — remove items that belong elsewhere; route them to the projects they belong to
5. **Rate** — A (must do), B (should do), C (could do) + scope (trivial/small/medium/large)
6. **Order** — within each priority band, by dependency chains and theme
7. **Note dependencies** — "requires X, first"
8. **File tickets** — epic + individual tickets, clustered where reasonable
9. **Work the list** — in order, one commit per ticket, smallest items first
