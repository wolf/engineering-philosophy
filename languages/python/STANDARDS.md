# Python Development Standards

Standards for writing Python. This document applies to every developer working on Python projects that adopt these standards.

This is the language layer of a four-document hierarchy:

* [What Matters Most](../../WHAT-MATTERS-MOST.md) — the four principles that ground everything
* [Standards for Shared Projects](../../STANDARDS-FOR-SHARED-PROJECTS.md) — language-agnostic baseline for any shared project
* **This document** — Python-specific standards
* [Audit Checklist](AUDIT-CHECKLIST.md) — practical methodology for verifying all of the above in Python code

This document makes [Standards for Shared Projects](../../STANDARDS-FOR-SHARED-PROJECTS.md) concrete for the Python stack and adds conventions that aren't fully captured in the language-agnostic baseline. Where a standard is language-agnostic and fully stated in the parent document, this document references it rather than repeating it. Where Python demands specific choices, they are stated here. For companion documents on specific topics — secrets, cross-platform development, versioning trade-offs, divergence, and more — see the [top-level README](../../README.md).

Two framing sections of the parent apply here unchanged: [Adoption](../../STANDARDS-FOR-SHARED-PROJECTS.md#adoption) — apply the standards incrementally, every time you touch existing code — and [Your Tools, Your Choice](../../STANDARDS-FOR-SHARED-PROJECTS.md#your-tools-your-choice) — the standards govern shared artifacts, not your personal workflow.

## No Surprises

*(See [What Matters Most § No Surprises](../../WHAT-MATTERS-MOST.md#no-surprises))*

This is a guiding principle at every level — not just project layout, but code, APIs, naming, error handling, documentation, and commit messages.

If someone familiar with Python encounters your work, nothing should be unexpected. A module named `exceptions` should contain exceptions. A `pyproject.toml` should be configured the way the ecosystem expects. A commit message should describe what happened and why. (New programmers are surprised by everything they see. That is not the case these standards address.)

The No Surprises principle applies everywhere:

* **Project layout**: Follow the standard shape for your project type. Someone who has seen one uv project should recognize yours immediately.
* **Code**: Write Pythonic code. Use established patterns. If you must do something surprising, explain why in a comment.
* **APIs**: Names, signatures, and return types should tell the caller exactly what to expect. Exceptions should be specific and documented.

When you're unsure whether something is surprising, it probably is. Make it obvious.

---

## Projects

This section covers project setup — referenced when starting new work, not every day.

### Developer Prerequisites

Install these tools globally, using whatever scheme you favor (e.g., apt, Chocolatey, brew, or scoop):

* **`direnv`** — makes starting work on an existing project (or switching between projects) seamless. It activates the right virtual environment, sets environment variables, and gets you to a working state just by entering the directory. Optional, but you live a happier life if you install it.
* **`uv`** — Python package and project management.
* **`pixi`** — for projects with conda-ecosystem dependencies.
* **`prek`** — pre-commit hook runner (Rust-based, reads `.pre-commit-config.yaml`).

If you need conda-ecosystem packages, use `pixi` — it manages them without polluting your global environment. Avoid installing conda, mamba, or miniconda globally; they pollute the global environment and make development unreliable and unrepeatable.

### Project Shape

Python projects follow a standard shape. The specifics depend on whether the project has conda-ecosystem dependencies:

* **`uv` projects** — the default for most work. No conda requirements.
* **`pixi` projects** — when the project needs packages from conda-forge or similar channels.

Both share these conventions:

* **`src/` layout** — prevents accidental imports of the source tree instead of the installed package. Your package lives in `src/<package_name>/`.
* **`pyproject.toml`** as the single configuration file. Tool configuration (ruff, pytest, ty), project metadata, build system — all in one place. Avoid scattering config across dotfiles. See [samples/pyproject.toml.sample](samples/pyproject.toml.sample) for authorship, licensing, and dynamic versioning.
* **hatchling + hatch-vcs** for building and versioning. The version comes from git tags, not from a manually-edited string.
* **`.envrc.sample`** committed to the repo with useful setup (virtualenv activation, environment variables, path configuration). Developers copy or symlink this to their own `.envrc`, which is gitignored and may contain credentials. See [samples/envrc.sample](samples/envrc.sample) and [samples/direnvrc.sample](samples/direnvrc.sample).
* **Lockfiles are always committed.** `uv.lock` or `pixi.lock` goes into version control (an unfortunate current known problem in `pixi` projects is a stray `uv.lock` that does not apply — `pixi` projects must `.gitignore` that file). Teammates on different machines on different days must be able to build exactly the same thing. In `pyproject.toml`, declare dependencies with the ranges and constraints you actually need — specific pins, version ranges, git branches, optional features. For everything else, let the lockfile do the pinning. This keeps `pyproject.toml` focused on *intent* and the lockfile focused on *reproducibility*. When you want newer versions of dependencies, update the lockfile explicitly — don't let it drift.
* **`.gitignore`** covers the obvious — build artifacts, virtual environments, editor files, OS files, `.envrc`.
* **Generated files excluded from linting** (e.g., `_version.py`), per [Standards for Shared Projects § Project Hygiene](../../STANDARDS-FOR-SHARED-PROJECTS.md#project-hygiene).

### Required Files

How much documentation a project needs depends on the breadth of its audience — just as names with broad scope in code must be more descriptive, more widely shared work needs more thorough documentation. But every project, no matter how small, has at least a README and a LICENSE.

**Every project:**

* **README** — the starting point for everyone. Non-negotiable. It must include everything specified in [Standards for Shared Projects § The Non-Negotiable Files](../../STANDARDS-FOR-SHARED-PROJECTS.md#the-non-negotiable-files). For small projects where the README can say it all, it should. Contributing instructions, usage walkthrough, and design notes can live here when separate documents would be ceremony without value.

* **LICENSE** — what others are allowed to do with your code. No license means no permission. The LICENSE file contains the full text; `pyproject.toml` declares it for tooling and package indices. See [samples/LICENSE.md.sample](samples/LICENSE.md.sample) for an MIT template.

**Projects producing versioned releases** must also have:

* **CHANGELOG** — what changed, when, and why. Follow the format specified in [Standards for Shared Projects § Changelog Discipline](../../STANDARDS-FOR-SHARED-PROJECTS.md#changelog-discipline).

**Projects with a broader audience** — libraries others depend on, tools used across teams — might also need a TUTORIAL, an ARCHITECTURE document, and a CONTRIBUTING guide. What each is for, who it serves, and what it must contain are described in [Standards for Shared Projects § Documentation](../../STANDARDS-FOR-SHARED-PROJECTS.md#documentation) — none of it is Python-specific.

---

## Workflow

The shared rules of engagement — how everyone contributing to a project works together.

### Pre-commit and Quality Gates

Every Python project using these standards gates commits with Git's pre-commit hook. The hook itself is not optional; `prek` is the standard runner for it — a default that, like the other tool choices, a defended bend can override (see [Tool Choices](README.md#tool-choices)).

The hook runs, in order:

1. **detect-secrets** — secret scanning (fail fast on the worst possible mistake)
2. **ruff-format** — formatting
3. **ruff** — linting (with auto-fix where safe)
4. **ty** — type checking
5. **pytest** — as much of the test suite as can run helpfully without delaying the commit (the rest gates in [CI](#continuous-integration))

If any of these fail, the commit does not happen. Hooks are the fast, local gate, right next to the work; the gate no one can skip is CI ([Standards for Shared Projects § Continuous Integration](../../STANDARDS-FOR-SHARED-PROJECTS.md#continuous-integration)).

**One-time setup: create the detect-secrets baseline.** Before the hook can run, you must create `.secrets.baseline`:

```
detect-secrets scan > .secrets.baseline
```

Commit this file. The hook uses it to track known non-secrets and flag anything new. Without it, the hook fails on the first commit.

ruff, ty, and pytest are configured in `pyproject.toml` with deliberate settings — configured, not just installed, per [Standards for Shared Projects § Quality Tooling](../../STANDARDS-FOR-SHARED-PROJECTS.md#quality-tooling).

**ruff** is configured with explicit rule selection. The standard rule sets:

* **`D`** — docstring conventions (pydocstyle)
* **`E`** — PEP 8 errors (pycodestyle)
* **`F`** — pyflakes (unused imports, undefined names, etc.)
* **`I`** — import sorting (isort)
* **`W`** — PEP 8 warnings (pycodestyle)
* **`UP`** — pyupgrade (modernize syntax for your target Python)
* **`B`** — flake8-bugbear (common bugs and design problems)

Standard ignores: `D203` (conflicts with D211), `D212` (conflicts with D213). Both are docstring formatting rules that contradict each other — these standards pick D211 and D213.

**ty** needs minimal configuration. Exclude generated files (`_version.py`), build machinery (`hatch_build.py`), and any directories that aren't ready for type checking. For pixi projects, point ty at the pixi Python so it finds the right environment.

See [samples/pyproject.toml.sample](samples/pyproject.toml.sample) for the complete ruff and ty configuration blocks, and [samples/pre-commit-config.yaml.sample](samples/pre-commit-config.yaml.sample) for the hook sequence.

**Zero-warning policy.** Every commit passes with zero errors and zero warnings, including pre-existing issues in files you didn't otherwise touch — fix them. [Standards for Shared Projects § Quality Tooling](../../STANDARDS-FOR-SHARED-PROJECTS.md#quality-tooling) explains why zero is the only sustainable number.

**Format before staging.** Run `ruff format` on changed files before staging so the pre-commit hooks are a safety net, not a reformatter.

**Optional but recommended: `codespell`.** Adding codespell to your pre-commit hooks catches typos in code, comments, and documentation before they ship. It's not mandatory, but it's cheap to run and catches real mistakes.

### Continuous Integration

The expectations — the same gates as the hooks, on every shared-branch change, plus the matrices and audits a single machine can't provide — are stated in [Standards for Shared Projects § Continuous Integration](../../STANDARDS-FOR-SHARED-PROJECTS.md#continuous-integration). In Python:

* **Run the identical hook sequence** with `prek run --all-files` — same hooks, same configuration, zero drift between local and CI.
* **Mark slow tests** (e.g., `@pytest.mark.slow`, registered in `pyproject.toml`) and deselect them in the pre-commit hook (`-m "not slow"`); CI runs the full suite, markers included.
* **Matrix over Python versions**: the minimum supported version and the latest stable (see [Version Policy](#version-policy)), on every platform the project supports (see [Cross-Platform Development](../../CROSS-PLATFORM.md)).
* **Audit dependencies** with `uv audit` as a pipeline step, per [Using Current Versions § Tools](../../USING-CURRENT-VERSIONS.md#tools).

### Git Conventions

Commit messages, branch naming, pruning merged branches, and the ban on commented-out code in pull requests are language-agnostic — see [Standards for Shared Projects § Git Conventions](../../STANDARDS-FOR-SHARED-PROJECTS.md#git-conventions).

**Annotated tags only. Version tags are permanent.** (See [Standards for Shared Projects § Versioning and Releases](../../STANDARDS-FOR-SHARED-PROJECTS.md#versioning-and-releases).)

**Branch model:** Follow the three-branch model defined in [Standards for Shared Projects § Branch Policy](../../STANDARDS-FOR-SHARED-PROJECTS.md#branch-policy) — release (always production-ready, never rewritten), integration (where feature branches merge for testing, usually named `main`), and feature branches (disposable). See [Minimizing Divergence](../../MINIMIZING-DIVERGENCE.md) for why keeping branches short-lived and consumers current matters.

### Versioning and Releases

**Semantic versioning** as defined in [Standards for Shared Projects § Versioning and Releases](../../STANDARDS-FOR-SHARED-PROJECTS.md#versioning-and-releases). Major means breaking (with migration guide), minor means new features, patch means bug fixes.

**Tag-driven versioning** via hatch-vcs. The version comes from git tags — one source of truth, zero chance of mismatch. Before your first release, create an initial version tag — without one, hatch-vcs produces `0.0.0.dev0+g{hash}`. The right tag is a judgment call based on how complete the implementation is: `v0.1.0` is conventional for something functional but early; `v1.0.0` for something you'd consider complete. Use an annotated tag: `git tag -a v0.1.0 -m "Initial release"`.

**Ignore working-tree dirtiness.** Configure hatch-vcs to compute the version purely from committed git history. Without this, any uncommitted edit at install time bumps the reported version to `<next>.dev0+g{hash}.d{date}`, causing tagged releases to occasionally report wrong runtime versions and lockfiles to churn on every edit.

```toml
[tool.hatch.version]
source = "vcs"
raw-options = { local_scheme = "no-local-version", git_describe_command = ["git", "describe", "--tags", "--long", "--match", "*[0-9]*"] }
```

`git_describe_command` without `--dirty` hides working-tree state from setuptools-scm. `local_scheme = "no-local-version"` drops the `+g{hash}` segment, yielding clean version strings (`1.1.0`, `1.1.1.dev3`).

---

## Python

This section covers the code itself — the everyday work.

### Version Policy

Always develop against the best Python available. Use the latest stable release for new work.

At the same time, decide the oldest version of Python a client could reasonably have and still use your library. Declare that as your minimum supported version. Test against it.

There may come a time when an older Python version no longer deserves active support — when maintaining compatibility costs more than it's worth. When that happens, freeze a branch at that Python version and move on. The frozen branch exists for anyone who needs it; the main line moves forward.

### Style

Write Pythonic code. Prefer clarity and idiom over cleverness. Use the best names you can find for variables, functions, classes, and modules. Good names make comments unnecessary; bad names make comments insufficient.

**Quotes.** Prefer double quotes (`"`) over single quotes (`'`).

**`from __future__ import annotations`.** Use it in code that must run on Python 3.13 or older. It enables postponed evaluation of annotations, which avoids circular import issues and allows forward references. Python 3.14 makes deferred evaluation of annotations the default (PEP 649), which covers the same needs — once your minimum supported version is 3.14 or newer, the import is redundant; don't add it.

**`__all__`** should be a tuple (not a list), sorted alphabetically, with one name per line:

```python
__all__ = (
    "Connection",
    "ConnectionError",
    "connect",
)
```

Not every package needs `__all__` at every level. Some packages are designed so that callers dive into named sub-packages rather than importing everything from the top level. In that case, define `__all__` in the sub-packages that serve as entry points, but don't force a top-level `__all__` that flattens a namespace you deliberately kept structured. The rule is: wherever a caller is expected to `from X import name`, `X` should define `__all__`.

### Density

Write the densest code that remains readable. Dense code has fewer places for bugs to hide, less surface area to maintain, and communicates intent more directly. Each pattern below replaces a verbose multi-line idiom with a shorter, equally clear alternative.

* **Comprehensions over loops.** If a loop exists only to build a collection, it should be a comprehension or generator expression.
* **Walrus operator (`:=`).** Use it to avoid compute-then-check-then-use across separate lines.
* **Compound conditionals.** `if a and b:` instead of nesting. `if not x: continue` at the top of a loop instead of wrapping the body.
* **`next()` for single-item search.** `next((x for x in items if pred(x)), None)` beats a loop-with-break.
* **Yield, don't collect.** If a function builds a list only to return it, yield instead.
* **Data-driven over copy-paste.** N near-identical blocks → one loop over a table of differences.
* **Early return / continue.** Flatten nesting by exiting early.
* **One expressive line > three obvious lines.** Chaining, unpacking, and conditional expressions are fine when they clarify.

Density is not cleverness. If a one-liner requires a comment to explain what it does, it isn't dense — it's obscure. The test: can another developer read it at review speed without pausing? If yes, prefer the shorter form.

### Docstrings

Use docstrings at every level: module, class, method, and function. Use Markdown formatting.

**Single-line docstrings:**

```python
"""Create the thing."""
```

Imperative mood, ends with a period, same line as the surrounding quotes.

**Multi-line docstrings:**

```python
"""
Create the thing with the given parameters.

Additional details here. Like a git commit message: succinct imperative
summary, blank line, then paragraphs as needed.

"""
```

Opening and closing `"""` on their own lines. Summary immediately after the opening quotes, left-aligned. Blank line after the summary, then body. Blank line before the closing `"""` (NumPy-style convention).

**Doctests.** Docstrings are a natural home for testable examples. When a function's behavior is best explained by showing it in action, write that example as a doctest. Doctests serve double duty: they document usage *and* they verify it. A doctest that drifts out of sync with the code will fail — which is exactly the point.

Not every docstring needs a doctest. Use them where the example adds clarity — typically for functions with non-obvious behavior, string formatting, edge cases, or transformations where seeing input → output is worth more than prose. Don't add doctests to trivial one-liners or to functions whose behavior is fully conveyed by the summary and type annotations.

Doctests run as part of the test suite via pytest's `--doctest-modules` flag. They are subject to the same zero-failure policy as all other tests.

**Content guidelines:**

* Don't mention types — that's what annotations are for
* No arguments section — good names + annotations suffice; discuss relationships between arguments when the interaction is important
* Don't describe the return value unless details are critical beyond what the summary conveys
* Do mention exceptions raised
* Do mention invariants maintained when important
* Be succinct

**When to omit:** Obvious functions — one-liners, well-known interfaces like `__repr__` — may skip docstrings. Add a comment to suppress the linter warning.

### Comments

Like docstrings: succinct, Markdown, only where needed. Comments explain the *why*; code explains the *how*. Use comments when the technique, code, or result is surprising or non-Pythonic. Obvious code is better than non-obvious code with good comments.

### Type Annotations

Annotate all function signatures — parameters and return type. Functions without a return value use `-> None`. Use `typing_extensions` for backports where applicable.

**Type philosophy:**

* **Parameters**: Accept the broadest reasonable type. If a function needs to iterate over items, accept `Iterable`, not `list`. If it needs to look things up by key, accept `Mapping`, not `dict`. This gives callers maximum flexibility.
* **Return types**: Return the narrowest useful type, adjusted by what is most useful to the caller, what the Pythonic convention is, and what promise the type makes. For example, `list` implies order is meaningful — consider whether `Sequence`, `Iterable`, or `set` better expresses your intent.

**Enforce strictly on library code.** Relax on examples, experiments, and scripts — those serve different purposes.

**`TYPE_CHECKING` blocks** prevent circular imports without losing type information. Use them when needed — see [Imports](#imports) for details.

### Imports

**Prefer qualified names when reasonable.** Names fundamental to your problem space, or names from within your own package, are typically better unqualified. For anything less familiar or external, `import foo` then `foo.bar()` keeps the origin visible at every call site. Prefer this over `from foo import bar` or renaming with `as`. Renaming is acceptable when the qualified name is genuinely unwieldy, when convention demands it (e.g., `import numpy as np`), when resolving a name conflict, or when one layer of indirection turns a generic name into a clearly understandable one (`np.nan` is clearer than bare `nan`).

**Absolute vs. relative imports:**

* **Inside `__init__.py`**: use relative imports (`from .module import Name`). This is the one place relative imports belong.
* **Everywhere else**: use absolute imports starting from the top-level package. Absolute imports are unambiguous — you can tell exactly where the code lives by reading the import line.
* Never use multi-level relative imports (`from ...something import x`). If you're reaching up multiple levels, you're importing from a layer you shouldn't depend on.

**Layering prevents circular imports.** Structure code in layers. Lower layers provide services; upper layers consume them. A lower layer never imports from an upper layer. If your packages respect layers, circular imports cannot arise. If a package straddles multiple layers, it's too big — split it. When you can't avoid calling upward, use callbacks, signals/slots, or dependency injection — not imports.

**Import from the package, not its internals.** Callers use `from your_package import Thing` — they go through the front door (`__init__.py`), never reaching into internal modules. The only code that imports internal modules is the package's own `__init__.py`. If many callers bypass `__init__.py` to import a specific internal, that internal probably deserves to be its own package. A package may deliberately expose named sub-packages as part of its public API to reduce namespace pollution. In that case, the sub-package names are public — callers import from them directly, and each sub-package follows the same `__init__.py` rules.

**`__init__.py` rules:**

* It is the public API of the package. It imports exactly and only the names the package exports, using relative imports.
* It defines `__all__`.
* It starts with a module-level docstring describing the package.
* It contains no class, function, or variable definitions — only imports, `__all__`, and the docstring. Definitions in `__init__.py` are surprising. The one exception is the `__version__` fallback assignment below.
* It does not re-export names from outside the package. Re-exporting external names creates two import paths to the same object, which breaks patching and confuses readers.
* **Shared libraries also expose `__version__`**, included in `__all__`. Import it from `._version` (generated by hatch-vcs at install time) with a fallback for environments where the package isn't fully installed.

See [samples/\_\_init\_\_.py.sample](samples/__init__.py.sample) for a complete example.

**`TYPE_CHECKING` for type-only imports.** When you import a name solely for type annotations, guard it with `if TYPE_CHECKING:`. This prevents the import at runtime, avoiding circular import issues and unnecessary load-time cost. Annotations that mention the guarded names must not be evaluated at runtime: on Python 3.14 and newer, deferred evaluation (PEP 649) takes care of this by default; on older versions, combine the guard with `from __future__ import annotations` so the annotated types become strings that don't need to be resolvable at runtime.

**No star imports in production code.** `from package import *` is for tests and interactive use (REPL, Jupyter) only. Production code imports specific names.

**Delayed imports as a last resort.** Moving an import into a function body can break a circular import cycle. This is acceptable when the name is used in only one or very few functions, is not re-exported, and is not performance-critical. Always comment why the import is delayed. Python 3.15 introduces explicit lazy imports (PEP 810), which will largely eliminate both this workaround and the need for `TYPE_CHECKING` guards. When these standards move to 3.15, expect this guidance to simplify.

**Don't manipulate `sys.path` in production code.** If your imports require `sys.path` surgery at runtime, the real problem is your package structure or your `PYTHONPATH` setup. Fix the root cause.

**No namespace packages.** Every package directory must have an `__init__.py`. Don't rely on implicit namespace packages — they are surprising, fragile, and rarely needed.

**Import ordering.** ruff (via isort rules) handles import ordering automatically. Don't fight it. When order genuinely matters (rare), use `# noqa: I001` and a comment explaining why.

### API Design

The language-agnostic principles are stated in [Standards for Shared Projects § API Design](../../STANDARDS-FOR-SHARED-PROJECTS.md#api-design). In Python specifically:

**Define `__all__` in every module that has a public API.** Every name in it is a commitment you're making to your users. Every name not in it is an implementation detail you're free to change.

**Use `Protocol` to define what callers must implement.** This tells them exactly what's needed without coupling them to your class hierarchy.

**Deprecate with `warnings.warn()` and `DeprecationWarning`.** Point to the replacement. Remove in the next major version.

### Eager vs. Lazy

When you give access to a collection, you get to choose: return a completely resolved result (e.g., a `list`), or return items one at a time.

**Eager if it is already fully resolved.** If you have a cached list of children, return the list. This is independent of your return *type* which might still be `Iterable`.

**Eager if that is the point, and a promise you are making.** This comes into play when computation is required, but you can do it more efficiently than the caller, and the caller genuinely wants a fully-resolved result.

**Lazy if you must compute one item at a time.** Even if the per-item work is insignificant, `yield`ing an answer is more Pythonic, and almost always sufficiently low-cost. A caller who needs eager just wraps the call in `list` or `set` or whatever.

**Lazy if you get it from a lazy (or unknown) source.** No need to force eager when it is unneeded and not already in place. `yield from` sometimes comes into play here.

### File-System Locations

*(See [What Matters Most § No Surprises](../../WHAT-MATTERS-MOST.md#no-surprises) and [Cross-Platform Development](../../CROSS-PLATFORM.md).)*

Any application that stores configuration, caches data, or writes logs must put those files in the platform-appropriate location — not in the working directory, not in a hardcoded path, not next to the executable.

**Always use `platformdirs`** (or an equivalent library) to find the right directory for each kind of data: user config, user data, cache, and logs. `platformdirs` respects XDG conventions on Linux, standard locations on macOS and Windows, and handles the differences so you don't have to. Hardcoding `~/.myapp` or `~/.config/myapp` is wrong on at least one platform — let the library decide.

**Separate concerns by directory type.** Configuration, cache, data, and logs serve different purposes and have different lifecycles. A user should be able to delete the cache without losing configuration. Use `platformdirs.user_config_dir()`, `user_cache_dir()`, `user_data_dir()`, and `user_log_dir()` — not a single directory for everything.

**Create directories on first use, not at import time.** Directory creation is a side effect. Do it when you first need to write, not when the library is loaded.

### Logging

The library/application responsibilities — libraries emit, applications configure; library logging is opt-in; lines are identified; the log level is exposed to end users — are stated in [Standards for Shared Projects § Logging](../../STANDARDS-FOR-SHARED-PROJECTS.md#logging). In Python:

**Always use loguru** — not the stdlib `logging` module. Loguru is simpler, more powerful, and produces better output with less configuration.

**"Emit without configuring" means a library never calls `logger.add()` or sets a global level.** Libraries start with their own logging disabled; users opt in by enabling logging for that library.

**Tag with `logger = logger.bind(library="your-project")`** at module level — tag with the *project* name; the `logger.enable()`/`disable()` key is the *package* name. This attaches context to every message without making decisions about sinks or levels. See [samples/\_logging.py.sample](samples/_logging.py.sample) for the full pattern, including the tag/key distinction and client-side usage.

### Error Signaling

**`assert` is for programmer errors and invariants only.** Never for input validation or control flow. Write assertions as though the end user will always be running with `python -O` — because they should be. Assertions are stripped by `-O`, so they must never have side effects or guard against user-provided data.

**`raise` is the primary error signaling mechanism.** Use it for expected failure modes that callers should handle. Use specific exception types (see API Design). Every library defines a base exception class for all errors it can emit; callers can catch the base when they want a blanket handler and catch specifics when they need precision.

**Return values for expected, normal-path failures.** When failure is a routine outcome — not an exceptional condition — returning `None` or an explicit result type is appropriate (e.g., `find_user` returning `None` when no user matches). Use `Optional` or a result type, not magic sentinel values.

---

## When Your Project...

Not every project is the same shape. The standards above apply universally. This section adds requirements that apply only when your project has certain characteristics.

### ...Has a Command-Line Interface

The universal conventions — results to stdout and diagnostics to stderr, meaningful exit codes, only the application exits the process, the standard argument grammar, standard global options — are stated in [Standards for Shared Projects § Command-Line Interfaces](../../STANDARDS-FOR-SHARED-PROJECTS.md#command-line-interfaces). In Python:

**Always use typer** — not `argparse`, not `click` directly. Typer gives you a CLI from type-annotated functions, which aligns with the type annotation standards above.

**Global options:** `--help` (typer provides it), `--version` / `-V` (get the version from `importlib.metadata`), `--log-level` (accept a level name; default to INFO or WARNING depending on the tool's purpose), and `--log-file` (optional; when provided, also write logs to the given path).

**Exiting:** inside a Typer command, use `raise typer.Exit(code)` — Typer catches this and exits cleanly, and handles usage errors (`2`) itself. In application code outside Typer (a `__main__` block, an entry-point wrapper), use `raise SystemExit(code)`. Never use `sys.exit()` — and never exit the process from library code at all (see [Error Signaling](#error-signaling)).

**Rich is already available.** Typer depends on `rich`, so it's in your environment at no extra cost. Use it for colored output, formatted tables, progress bars, and status spinners. Don't reach for hand-rolled ANSI codes or third-party formatting libraries when `rich` already does it better.

### ...Is a Library

**Do not configure logging.** See [Logging](#logging): libraries emit, applications configure sinks and levels; start with your logging disabled; tag every line with `logger = logger.bind(library="your-project")`.

**Define `__all__` (as a tuple) in every module that callers import from.** This is especially critical for libraries, where the public surface is a contract with external callers. For libraries with deliberately structured sub-packages, define `__all__` at each entry point rather than forcing everything through a single top-level export.

**Export a clean public API from `__init__.py`.** Users should be able to `from your_lib import TheThing` without knowing the internal module structure. This often means internal modules use underscore-prefixed filenames (e.g., `_utilities.py`) — the definitions they contain are public and exported, but the module itself is not part of the API.

**`__init__.py` is for re-exports, not definitions.** It is surprising when `__init__.py` contains actual class or function definitions. Define things in their own modules; re-export from `__init__.py`.

**Type your public surface strictly.** Internal code can be looser, but anything in `__all__` must have complete, accurate type annotations.

**Deprecate before removing.** As stated in [API Design](#api-design): `warnings.warn()` with `DeprecationWarning`, a pointer to the replacement, removal in the next major version.

**Avoid side effects at import time.** Importing a library should not print, log at WARNING+, open network connections, or modify global state.

---

## Verification

*(See [What Matters Most § Tests and Measurement](../../WHAT-MATTERS-MOST.md#tests-and-measurement))*

Every change needs tests. Every change needs documentation. This is daily work, not a phase. The philosophy and expectations are stated in [Standards for Shared Projects § Testing](../../STANDARDS-FOR-SHARED-PROJECTS.md#testing). This section adds Python-specific requirements.

### Testing

The full expectations — tests as part of the work, contract over implementation, test independence, the 80% coverage floor, tracking known-but-unfixed problems, annotating critical paths — are stated in [Standards for Shared Projects § Testing](../../STANDARDS-FOR-SHARED-PROJECTS.md#testing). In Python:

**Coverage** is measured with `pytest-cov`. Keep it in the dev group and measure with `uv run pytest --cov`.

**Known-but-unfixed critical problems** are marked with pytest's `xfail`.

**`tests/` is not a package.** The test directory does not have an `__init__.py` file. Tests import the code under test starting from the top-level package name, just like any other consumer.

### Documentation

A missing doc update is the same as a missing test (see [Standards for Shared Projects § Documentation](../../STANDARDS-FOR-SHARED-PROJECTS.md#documentation)). If you change behavior, the docs must reflect it in the same branch. Not in a follow-up PR, not in a "docs sprint" — in the same unit of work that changed the code.

This applies to all documentation the project maintains — docstrings, README, CHANGELOG, and any additional documents (TUTORIAL, ARCHITECTURE, CONTRIBUTING). If a change affects what users see, what contributors need to know, or how the system works, the corresponding documentation must be updated.

### Benchmarks

Tests prove correctness. Benchmarks measure progress toward the goal. The full philosophy is in [Standards for Shared Projects § Benchmarks](../../STANDARDS-FOR-SHARED-PROJECTS.md#benchmarks).

Not every project needs benchmarks. But when your work is a critical user of a resource — typically time, sometimes storage — benchmarks earn their keep. A CRUD adapter around someone else's API has little to benchmark. A data pipeline that processes millions of records does.

When benchmarks apply, follow the standards: know the goal, measure against it, compare honestly (see [What Matters Most § Honesty](../../WHAT-MATTERS-MOST.md#honesty)), and make results reproducible.

---

## AI

The full philosophy is in [Using AI](../../USING-AI.md): AI is a legitimate tool and a choice, and when you let it generate code, you must understand every line — the same standard you would hold yourself to writing it by hand.

Several items on that document's do-not-accept list take specific Python forms:

* **Non-Pythonic** — fighting the language instead of using it, reimplementing what the standard library already provides
* **Using the wrong data structures** — a list where you need a set, a dict where you need a dataclass, a class where you need a function
* **Carrying the wrong complexity** — O(n²) where O(n) is straightforward, nested loops where a comprehension or `itertools` call would do
* **Testing implementations instead of promises** — tests coupled to private methods, internal state, or call counts rather than observable behavior (see [Testing](#testing))

The standards in this document apply equally to AI-generated code. If it wouldn't pass your review written by a colleague, it doesn't pass written by a machine.

---

For a practical checklist to verify these standards in existing code, see the [Code Audit Checklist](AUDIT-CHECKLIST.md).
