# Cross-Platform Development

Most software can run on more than one platform. Whether it *should* is a decision, but it should be a deliberate one, not an accident of assumptions.

This document addresses three distinct questions that are often conflated:

* **Can the software run cross-platform?** Whether the application itself works on multiple operating systems.
* **Can developers work on the code cross-platform?** Whether people can edit, lint, type-check, and test on whatever machine is best for them, even if the application only deploys to one platform.
* **How deeply does it embrace each platform it runs on?** Whether the artifact merely works there, or is truly at home there.

These are separate goals with different costs. Editable-anywhere is almost always achievable and almost always worth it. Runnable-anywhere depends on your dependencies and your architecture. At-home-everywhere is the most expensive of the three, and the one most often promised by accident.

This document is language-agnostic in principle. Where Python-specific tooling or conventions apply, they are noted as such; every ecosystem has equivalents.

## State Your Platform Intent

Every project whose deliverable runs should declare its platform posture explicitly: in the README, as part of its goals. There are three common positions:

* **Fully cross-platform.** The software runs on any supported platform. This is the default for libraries, CLI tools, and anything without inherent platform ties.
* **Runs on one platform, editable on any.** The software deploys to a specific target (e.g., Linux servers), but developers can work on it from any platform. Linting, type checking, and most tests pass everywhere. Platform-specific functionality is isolated and skippable in tests.
* **Platform-specific by nature.** A Windows service wrapper, a macOS menu bar app, a tool that directly interfaces with platform-specific hardware. Even here, consider whether the core logic could be portable and only the platform integration layer is specific.

If you don't state your intent, the project will drift toward whatever platform the original developer used, and that drift will be invisible until someone on a different platform tries to contribute.

Running on a platform and being at home on it are different commitments. Every platform has an ecosystem model (its conventions for menus and shortcuts, its integration points, its distribution channels, its idea of how an application behaves), and a **truly** Mac-like Mac app costs far more than a cross-platform solution that merely runs on macOS. If you want your cross-platform artifact to be exemplary on one platform, let alone every platform you support, you are signing up for that much more work, per platform. So the declaration has a third axis: beyond where the software runs and where developers work, state how deeply each target is supported.

* **Functional**: the tool works, and works identically everywhere. Paths, locations, and line endings are correct ([Mechanics](#mechanics)); nothing more is promised.
* **Respectful**: the tool follows each platform's conventions where the toolkit provides them cheaply (native dialogs, standard shortcuts, platform-appropriate locations; [GUI Frameworks](#gui-frameworks)).
* **Exemplary**: the artifact is indistinguishable from one built by and for that platform (conventions satisfied completely, integrations used, distribution channel honored). This is [having a recognizable shape — and satisfying it](STANDARDS-FOR-SHARED-PROJECTS.md#have-a-recognizable-shape--and-satisfy-it), applied to a platform's whole ecosystem: a large, per-platform commitment that a cross-platform core enables but does not buy.

Mixed declarations are legitimate and common: exemplary on the platform your users live on, functional everywhere else. What isn't legitimate is implying exemplary and shipping functional.

One architecture changes these economics more than any other: factor the project's core (the domain logic, the data, everything that makes it *your* tool) into a component that shares cleanly across platforms, and let the UI layer vary as much as each platform demands, up to and including its implementation language. A portable core wearing a native Swift shell on macOS and a native shell elsewhere costs one core plus thin shells; exemplary-everywhere without that split costs the whole application, per platform. The isolation discipline in [GUI Frameworks](#gui-frameworks) is this idea in miniature; the core/shell split is it at full scale. The split pays again beyond platforms: a portable core is a scriptable one ([Standards for Shared Projects § Scriptability](STANDARDS-FOR-SHARED-PROJECTS.md#scriptability)). And it keeps the editable-anywhere promise honest: the core remains editable on every developer platform, while each native shell is editable only at home; declare that boundary along with the rest of your intent.

## The Cost of Cross-Platform Is Usually Low

This is the cost of *functional* portability; being at home on a platform is a different bill (see [State Your Platform Intent](#state-your-platform-intent)). Most code written in a modern high-level language is inherently portable. Python, for example: the language, the standard library, and the major frameworks (Qt, Flask, FastAPI, pandas, numpy) all work across platforms. The things that break portability are almost always specific, avoidable choices:

* Hardcoded path separators (`/` or `\\`) instead of `pathlib.Path`
* Hardcoded locations (`/tmp`, `~/.config`, `%APPDATA%`) instead of `platformdirs` or `tempfile`
* Assumptions about case sensitivity (the default filesystems on macOS and Windows are case-insensitive; Linux's are not)
* Shell-specific subprocess calls (`shell=True` with bash syntax)
* Direct use of platform APIs (Win32, COM, `/proc`) without an abstraction layer

When these are avoided from the start, cross-platform support comes nearly for free. When they accumulate unchecked, porting becomes a project unto itself.

## Even When You Can't Run Cross-Platform

Even if the application only runs on one OS, supporting multiple platforms in your project configuration has real value.

**Developers can edit on their preferred machine.** A correct project configuration (in Python, `pyproject.toml`) with proper dependency declarations lets developers install, lint, type-check, and run most tests on whatever platform they prefer. This matters: people are most productive on the machine they know best.

**CI can validate on multiple platforms.** Matrix builds catch portability issues before they become entrenched (see [Standards for Shared Projects § Continuous Integration](STANDARDS-FOR-SHARED-PROJECTS.md#continuous-integration)). Even if you only *deploy* to Linux, testing on macOS and Windows catches assumptions you didn't know you were making.

**You preserve the option to port later.** Requirements change. A tool that was "Linux only" may need to run on a developer's laptop. A Qt application that was "Windows only" may need a Linux deployment. If the codebase was written with portability in mind, this is a small project. If it wasn't, it's a rewrite.

## Mechanics

The mechanics below are stated in Python terms; every ecosystem has equivalents for each.

### Paths

Use `pathlib.Path` for all file system operations. Never construct paths with string concatenation or hardcoded separators.

Use `platformdirs` for config, cache, data, and log directories (see [Python Standards § File-System Locations](languages/python/STANDARDS.md#file-system-locations)). Use `tempfile` for temporary files and directories. Use `Path.home()` for the user's home directory.

### Dependencies

Use environment markers in `pyproject.toml` for platform-conditional dependencies:

```toml
dependencies = [
    "pywin32 ; sys_platform == 'win32'",
    "some-linux-lib ; sys_platform == 'linux'",
]
```

This lets the project install cleanly on any platform, pulling in platform-specific packages only where they apply.

### Entry Points

Use `console_scripts` (or `gui_scripts`) entry points in `pyproject.toml` instead of shell scripts or batch files. The packaging tool generates the correct wrapper for each platform automatically.

### Subprocess Calls

Avoid `shell=True`. It ties you to a specific shell's syntax and introduces security risks. Use a list of arguments instead. Use `shutil.which()` to locate executables portably.

### Line Endings

Commit a `.gitattributes` file that ensures consistent line endings:

```
* text=auto
```

This lets Git handle conversion so files are checked out with the platform's native endings but stored consistently in the repository.

### GUI Frameworks

A cross-platform GUI framework keeps you portable only as long as you stay inside it. Qt (in Python, via PySide6 or PyQt6) is the running example here; the same discipline applies to any cross-platform toolkit. A Qt application that only runs on one platform is almost always that way by choice, not by necessity.

To keep it portable:

* Use Qt's own APIs for file dialogs, system tray, notifications, and other OS-integration features. Don't call platform-native APIs directly for things Qt already provides.
* Let Qt's style engine handle native look-and-feel. Don't override it with platform-specific styling.
* Isolate any genuinely platform-specific code (COM interop, Windows registry access, macOS Keychain) behind an interface. Provide the platform-specific implementation for each target, and a stub or skip for platforms where it doesn't apply.

### Testing Across Platforms

* Run CI builds on every platform the project supports, or aspires to support.
* Use `pytest.mark.skipif(sys.platform != 'win32', reason="Windows-only feature")` for tests that exercise genuinely platform-specific functionality. Always include a reason.
* The "editable on any platform" standard means: linting, type checking, and the non-platform-specific portion of the test suite must pass on every developer platform. If they don't, the project configuration needs fixing.
* Native shells at the exemplary tier need their own test story, in their own toolchains; the portable core's suite doesn't reach them, and a feature without tests is not finished, shells included.

## The Trap

Platform assumptions don't announce themselves. A `/dev/null` here, an `os.sep` assumption there, a test that relies on case-insensitive file lookup. They work perfectly on the original developer's machine and fail silently (or noisily) everywhere else. The cost of fixing one is trivial. The cost of fixing dozens, discovered all at once when someone tries a new platform, is not.
