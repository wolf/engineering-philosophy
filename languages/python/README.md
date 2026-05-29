# Python

The Python layer of the [engineering philosophy](../../README.md): language-specific standards, an audit checklist, and starter samples. The language-agnostic foundations — [What Matters Most](../../WHAT-MATTERS-MOST.md) and [Standards for Shared Projects](../../STANDARDS-FOR-SHARED-PROJECTS.md) — apply first; everything here makes them concrete for Python.

## The Documents

* **[Standards](STANDARDS.md)** — Python-specific standards. Covers tooling (ruff, ty, pytest, prek, loguru), project layout (uv, pixi, hatchling), style, imports, error handling, CLI and library conventions, and AI usage.

* **[Audit Checklist](AUDIT-CHECKLIST.md)** — Practical methodology for auditing existing Python code against these standards. Each section maps to one or more of the four principles.

## Samples

The [samples/](samples/) directory contains starter files and reference patterns for common project artifacts:

| File | What it's for |
|---|---|
| `pyproject.toml.sample` | Complete project config: build system, metadata, ruff, ty, pytest |
| `pre-commit-config.yaml.sample` | Standard prek hook sequence (detect-secrets, ruff-format, ruff, ty, pytest with doctests) |
| `__init__.py.sample` | Package init — docstring, relative imports, `__all__` as tuple |
| `_logging.py.sample` | Loguru library setup (disabled by default, tagged) with client-side usage |
| `LICENSE.md.sample` | MIT license template |
| `envrc.sample` | Project-level `.envrc` for direnv |
| `direnvrc.sample` | Global `layout_uv` function for `~/.config/direnv/direnvrc` |

## Tool Choices

The standards are opinionated about tools: **uv** for packaging (fast, lockfile-first, one tool for environments, dependencies, and builds), **ruff** for linting and formatting (one fast tool in place of a stack of plugins), **ty** for type checking (fast, from the same toolchain family), **prek** for hooks (a Rust pre-commit runner with no Python bootstrap), **loguru** for logging (good output with minimal ceremony, clean library/application separation), and **typer** for CLIs (type-annotated functions become interfaces). Where [Standards](STANDARDS.md) argues a choice, argue back at the reasoning; where it merely asserts one, treat it as a default that a defended bend can override (see [Bending the Rules](../../WHAT-MATTERS-MOST.md#bending-the-rules)).
