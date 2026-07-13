# Standards for Shared Projects

Minimum standards for a project other people will depend on. Not aspirational: baseline. If a project doesn't meet these, it's not ready to share.

This document applies the four principles from [What Matters Most](WHAT-MATTERS-MOST.md) (Goals, No Surprises, Honesty, and Tests and Measurement) to the specific context of shared projects. Everything here is one of those principles made concrete for a project other people will depend on. For language-specific application of these standards, see the language layers under [languages/](languages/) (e.g., [Python Standards](languages/python/STANDARDS.md) and its practical [Audit Checklist](languages/python/AUDIT-CHECKLIST.md)).

## Adoption

New projects, new work, new code: start here.

Existing code can't change overnight. That's fine; don't let it stop you. Every time you touch existing code, apply these standards. When reasonable, add a test. When you find something broken (a non-reproducible release, a missing pre-commit configuration, a linter that's installed but never runs), fix it.

The goal is steady, incremental improvement. Not a big-bang rewrite. Not "we'll get to it later." Every commit is an opportunity to leave the codebase a little better than you found it.

## Your Tools, Your Choice

These standards govern a project's shared artifacts: the code that gets pushed, the branches everyone sees, the tests that gate commits, the APIs that get published. They do not govern how you get there.

Your editor, your Git client, your local branch names, whether you use worktrees, how you organize your terminal. All of that is yours. Use what works best for you. The standards apply to the shared artifacts, not to your personal workflow: your Git client is yours; the repository and its history are the project's ([Git Conventions](#git-conventions)).

## Know and State Your Goals

*(See [What Matters Most § Know and State Your Goals](WHAT-MATTERS-MOST.md#know-and-state-your-goals))*

A shared project must have a stated goal: in the README, right up front, as specified in [The Non-Negotiable Files](#the-non-negotiable-files).

The goal is not decoration. It drives what you build, what you measure, what you test, and when you stop. A project without a stated goal cannot be evaluated: you can't tell whether it's succeeding, failing, or solving the wrong problem entirely. And a goal that never evolves is a sign that nobody is paying attention.

Goals also discipline what arrives from outside. Users and stakeholders almost always ask in the form of a How: add this parameter, expose that internal, cache this call. The first job is to push the request back to its What ([Why, What, How](WHY-WHAT-HOW.md)): what outcome does the requester actually need? Requests granted at the How level accumulate surface area; requests understood at the What level accumulate capability.

One goal that every project whose deliverable *runs* should state explicitly is its platform intent: which platforms it runs on, and which platforms developers can work from. These are often different, and both matter. (An informational artifact has no platform to declare; see [The Deliverable Spectrum](#the-deliverable-spectrum).) See [Cross-Platform Development](CROSS-PLATFORM.md) for the full discussion.

## No Surprises

*(See [What Matters Most § No Surprises](WHAT-MATTERS-MOST.md#no-surprises))*

The single most important property of a shared project is that it behaves the way people expect.

### Have a recognizable shape — and satisfy it

Every ecosystem has standard project shapes: a uv project, a Cargo crate, an npm package, a Go module. Pick one. Then *satisfy* it completely. Your project should build the same way, run the same way, and have the same files and folders as every other project of that shape. Someone familiar with the shape should be able to clone your repo and already know where things are, how to build, how to test, and how to install, without reading a word of your documentation.

Don't invent a custom layout. Don't skip files the shape expects. Don't put things in surprising places. The shape is a contract with everyone who encounters your project.

### The Non-Negotiable Files

Every shared project has at minimum:

* **README** (the starting point for everyone). It must include:
  * **What this is.** Right up front, before anything else: what the project does, who it's for, and why they'd use it. Lead with intent, not implementation. A reader should know within the first sentence whether this project is relevant to them. For example: *"An ORM for clients who need Pandas DataFrames and NumPy arrays as a first-class interface."* Then follow with the technical shape (the primary language, version requirements, and project type) so the reader knows what they're working with. For example: *"This is a uv project, implemented almost entirely in Python with a small Rust core. We develop against Python 3.13; it's designed for use by Python 3.10+."*
  * **What this is not.** Who would *not* use this project? What problems does it deliberately not solve? How is it different from the alternatives? This is not modesty; it's orientation. A reader who doesn't need your project should find that out here, not after they've tried to use it.
  * **Goals.** The project's goals, stated explicitly: what it must achieve, what properties it must maintain, what constraints it operates under. These are not the same as the opening "what this is" paragraph; they are the validatable, prioritized commitments described in [What Matters Most § Know and State Your Goals](WHAT-MATTERS-MOST.md#know-and-state-your-goals). They drive what you build, what you test, and when you stop. They must be maintained as the project evolves: reviewed, updated, and pruned.
  * **How to install and use it.** A user who reads only the README should be able to use the library correctly.
  * **How to participate.** Where to file bugs, ask questions, start discussions, and provide feedback. Most project shapes support naming an author and an owner, but that's not enough; tell people explicitly how and where to engage: issue tracker, discussion forum, support channels. If the project accepts contributions, say what kinds, and which begin with a conversation rather than a pull request (see CONTRIBUTING under [Documentation](#documentation)). Note that the owner of a project (the organization or entity responsible for it) can be different from the principal author (the person who wrote it). Both should be clear.
  * **Links to the other key documents**: TUTORIAL, ARCHITECTURE, CONTRIBUTING, and CHANGELOG if the project produces versioned releases. The README is the hub; everything else is reachable from it.
* **LICENSE**: what others are allowed to do with your code. No license means no permission. Pick one and include it. For open source projects, choose an established license (MIT, Apache 2.0, etc.); for internal projects, choose a license appropriate for your organization (e.g., a proprietary internal-use license).

These exist before anything else. They are not optional, they are not "we'll add them later." This is not a rule that bends ([What Matters Most § Bending the Rules](WHAT-MATTERS-MOST.md#bending-the-rules)) because it is not a practice at all; it is the definition of *ready to share*. You can always choose not to share; you cannot share without a README and a LICENSE and call it meeting this standard.

### API Design

The public surface is a conscious decision. Define `__all__` (or your language's equivalent). Every name in it is a commitment. Every name not in it is an implementation detail you're free to change.

**Expose the version at runtime.** Shared libraries should expose their version as a string in the top-level namespace, following the language's convention (in Python, `__version__`, included in `__all__`). Consumers use it for runtime version checks and diagnostics; it is a widely-relied-upon convention, and omitting it forces consumers to dig the version out of package metadata themselves.

**Exceptions are part of the API.** Design your exception hierarchy for callers, not for your implementation. Callers need to catch specific things: give them specific types organized by what went wrong from their perspective.

**Deprecate with warnings, not silence.** When you rename or remove something, the old name should keep working with a deprecation warning and a pointer to the new name. Remove it in the next major version. Users who pin versions get time to migrate; users who don't get a clear message.

**Protocols over concrete inheritance at boundaries.** Use Protocol (or your language's equivalent) to define what callers must implement. This tells them exactly what's needed without coupling them to your class hierarchy.

### Type Safety

* **Annotate all public API.** Types are documentation that the toolchain can verify.
* **Use protocols or interfaces for extension points.** They tell implementors exactly what contract to satisfy.
* **Enforce strictly on library code.** Relax on examples and experiments; those serve different purposes.
* **Type-only imports should cost nothing at runtime.** When a name is needed only for annotations, don't let it create runtime coupling or import cycles (in Python, `TYPE_CHECKING` blocks).

### Logging

Logging crosses the library/application boundary, and the responsibilities on each side are different.

* **Libraries emit, applications configure.** Library code logs events at appropriate levels but never decides where logs go or what level is shown: no sinks, no global levels, no formatting decisions. The application (CLI, service, GUI) owns all of that. A library that configures logging is overstepping.
* **Library logging is opt-in.** A library starts silent; the consuming application enables it deliberately.
* **Libraries identify their log lines.** Every line a library emits carries the library's name (named loggers, bound tags, whatever the ecosystem provides), so the application can filter, route, and silence per library.
* **Applications expose the log level to end users.** How depends on the project type: a CLI takes a flag, a service reads an environment variable, a GUI offers a setting. The log destination should also be configurable where applicable: stderr, file, or both.
* **Format is project discretion.** Structured or human-readable: choose whatever is most useful to the people reading the logs, which is typically you, the developer, trying to understand errors.

### Command-Line Interfaces

A CLI is an API whose caller is a shell. The conventions are decades old; meet them.

* **Results go to stdout; diagnostics and logs go to stderr.** This is what makes CLI tools composable with pipes.
* **Exit codes mean something.** `0` for success, `1` for general errors, `2` for usage errors.
* **Only the application exits the process.** Library code never terminates the process; it raises errors and lets the application decide what they mean for the exit code.
* **Follow the standard argument grammar.** Single-character options start with a single dash and can be combined (`-ab` means `-a -b`). Long option names start with a double dash (`--verbose`). Required parameters are positional; they don't require option flags. Options with values accept the value with or without an `=` between the flag and the value (`--log-level DEBUG` or `--log-level=DEBUG`). Your CLI framework supports both; don't break either form.
* **Provide the standard global options.** `--help`, `--version`, and a way to control the log level (and, where useful, the log destination).

### Project Hygiene

* **A single canonical configuration file** (in Python, `pyproject.toml`). Avoid scattering config across multiple dotfiles when the toolchain supports centralized config. This includes declaring your license: the LICENSE file is the full text; the project config is where tooling and package indices read it from.
* **Separate the source tree from the installed artifact** (in Python, the `src/` layout): code under test runs against what will actually ship, and accidental imports of the source tree can't mask packaging mistakes.
* **Generated files excluded from linting** (in Python, e.g., `_version.py`). Don't fight tools over files you don't control.
* **`.gitignore` covers the obvious**: build artifacts, virtual environments, editor files, OS files.
* **Secrets never enter the repository.** Not in code, not in config, not in history. For the full treatment (rules, practices, and open scenarios), see [Handling Secrets](HANDLING-SECRETS.md).

## Honesty

*(See [What Matters Most § Honesty](WHAT-MATTERS-MOST.md#honesty))*

A shared project must be honest about what it is.

### Real Evaluation

If you compare your project against alternatives, the comparisons must be real. Each approach written idiomatically: no strawmen. If the alternative is simpler for a given task, show that. If your library is worse at something, say so.

### Real Benchmarks

If you publish performance numbers, they must be reproducible. Include the setup, the methodology, and the raw results. Document the performance capabilities of the benchmark hardware. Cherry-picked benchmarks are worse than no benchmarks: they set expectations you can't meet.

### Real Admissions

Every project has weaknesses, limitations, and cases where it's the wrong tool. Saying so explicitly builds more trust than any feature list. A user who discovers a weakness you documented will trust you. A user who discovers one you hid will not.

## Tests and Measurement

*(See [What Matters Most § Tests and Measurement](WHAT-MATTERS-MOST.md#tests-and-measurement))*

A shared project can't be taken on faith: tests are the evidence that its goals are met, and measurements the evidence that it's still making progress.

### Testing

Testing is fundamental. Not a phase, not a nice-to-have, not something you add after the code works. Tests are part of writing each new piece of code: they validate that it works now, and they verify it still works before and after every future change.

* **Write tests as part of the work, not after it.** New code means new tests. Changed behavior means updated tests. A feature without tests is not finished.
* **TDD is encouraged but not mandated.** What *is* mandated: tests exist, they pass, and they cover the contract. Whether you write them first or alongside the code is your call, but they must be there before the commit.
* **Tests must pass before every commit.** Not before release, not before merge: before *commit*. Pre-commit hooks enforce this. If the tests don't pass, the code doesn't enter history. (The hook runs as much of the suite as can helpfully gate a commit; slow-marked tests gate in [CI](#continuous-integration).)
* **Tests encode the contract**, not the implementation. They should break when behavior changes, not when internals are refactored.
* **Tests must be independent.** No test should depend on another test having run first. No test should leave state that another test relies on. Each test sets up what it needs and cleans up after itself.
* **Cover features, edge cases, and integration points.** A test suite that only covers the happy path is a test suite that only catches trivial bugs.
* **Maintain a coverage floor, and the ability to measure it.** At least 80% test coverage, with coverage measurement always ready to run. This is a floor, not a ceiling.
* **Test critical problems found but not yet fixed.** Mark them as expected failures (in Python, `xfail`) so the suite tracks them until they're fixed.
* **Annotate critical paths in source.** Code that is risky, has broad impact, or depends on subtle invariants gets a `CRITICAL:` comment explaining the stakes, and tests that cover it.
* **Testing is part of your standard workflow.** Running tests should be as automatic as saving a file. If it requires thought or effort to run the tests, the friction will win and people will skip them.

### Benchmarks

Tests prove correctness. Benchmarks measure how far along the path to success you are.

This has a prerequisite: **you must know what you intend to provide.** What is the actual goal of this project? What problem does it solve, and what does a successful solution look like? Without a clear goal, you have nothing to measure against.

Once you know the goal, benchmarks answer the questions that matter:

* **Are you moving toward or away from success?** A change that makes the tests pass but moves the benchmarks in the wrong direction is a regression, even if nothing is "broken."
* **Have you reached success?** At some point the benchmarks should show that the project delivers what it set out to deliver.
* **Are there still better alternatives?** Benchmarking against other approaches (held to [Real Evaluation](#real-evaluation)'s bar, idiomatic code on both sides) tells you whether your project earns its existence. If an alternative is consistently better, that's important to know and important to say.

A benchmark is a measurement instrument, and its honesty requirements (reproducibility, full methodology, raw results, no cherry-picking) are stated once, in [Real Benchmarks](#real-benchmarks). What a benchmark must never be is a demonstration that mints its own numbers: examples quote benchmarks, never the reverse (see [Examples](#examples)).

### Scriptability

Almost every tool, at every size, benefits from a non-GUI way in: an interface (a CLI, a REPL, an embedded interpreter, an API) with access to every underlying capability, whether or not end users ever see it. Tests drive the real thing instead of a mock of it; debugging reaches inside a live system; measurement gets its hooks in without a screen in the way. The need grows with the scale of what you're building: a small tool can be exercised by hand, but hand-driving a large one is combinatorial, while the scriptable path stays linear.

Some stacks make this nearly free. A Python program already ships inside a language its authors can drive: an embedded IPython session with the application's objects in scope adds a whole control language without adding a language at all.

Two things come later. The interface you built for tests and debugging sometimes graduates into a user-level scripting language, a major feature at a minor price, though not a free one: the moment it faces users, it is public API, with everything that entails ([API Design](#api-design), [Versioning and Releases](#versioning-and-releases)). And the factoring that makes it possible is one you may already have: a [core/shell split](CROSS-PLATFORM.md#state-your-platform-intent) built for platform reach pays a second time here.

One caution: an interface with access to every capability is an attack surface with access to every capability. Hidden is not secured: least privilege applies ([Handling Secrets](HANDLING-SECRETS.md#practices)), and in production the interface ships locked down or not at all. The obligation extends to what you ship: the tools you give users owe them the same respect for privacy, authority, and secrets that your own process enforces at home ([Respecting Users](RESPECTING-USERS.md)). A tool that can reach sensitive data needs guardrails (authentication at minimum), and a scripting surface that amplifies the tool's power needs the same guardrails or stronger.

### Quality Tooling

Quality tooling is the **automate** consequence of [the standing What](WHAT-MATTERS-MOST.md#the-standing-what) put into practice: machines catching problems before they spend human time. Tools must be configured, not just installed. Installing a linter and never running it is worse than not having it: it gives false confidence. Every tool in the project must be:

* **Configured** in the project's canonical config file with deliberate settings
* **Enforced** via pre-commit hooks and [CI](#continuous-integration): not optional, not manual

Together, the tools must cover the basics: secret scanning, linting, formatting, type checking, and tests.

**Zero tolerance for warnings.** Every commit must pass with zero errors and zero warnings. This includes pre-existing issues in files you didn't touch. Fix them. When warnings accumulate and go unfixed, people learn to ignore them. Once people ignore warnings, the tooling is dead weight. The only sustainable policy is zero.

**Pre-commit hooks that actually gate quality.** The pre-commit hooks should run: secret scanner first (fail fast on the worst possible mistake; see [Handling Secrets](HANDLING-SECRETS.md)), then formatter, linter (with auto-fix where safe), type checker, and test suite. If any of these fail, the commit doesn't happen. Hooks are the fast, local gate, right next to the work; [Continuous Integration](#continuous-integration) is the gate no one can skip.

### Continuous Integration

Pre-commit hooks are the developer's gate: they give feedback in seconds and keep obvious mistakes out of history. But they run on the developer's machine, at the developer's discretion: a hook can be skipped, misconfigured, or simply not installed on a fresh clone. CI is the gate that closes behind everyone: it runs where individual discretion can't reach, and its verdict attaches to the shared history everyone depends on.

* **CI re-runs the same gates as the pre-commit hooks** (secret scan, formatting, linting, type checking, tests) with the same tools and the same configuration, read from the project's canonical config file. If CI and the hooks can disagree, developers learn to distrust one of them. One gate is asymmetric: a secret that CI catches is already in history: detection there means rotation, not prevention ([Handling Secrets](HANDLING-SECRETS.md#practices)).
* **CI runs the tests too slow for the commit loop.** The pre-commit suite must stay fast, or friction wins and people route around it. Tests that would slow the commit flow too much (long integration scenarios, large datasets, exhaustive property sweeps) are marked as such and run in CI, where minutes are cheap. They are still gates: slow tests that fail block the merge just as surely as fast ones block the commit.
* **CI runs on every change to a shared branch**: every pull request, every push to integration or release. Not nightly, not on demand.
* **CI checks what a single machine can't.** The platform matrix from [Cross-Platform Development](CROSS-PLATFORM.md): every platform the project supports or aspires to support. The runtime matrix: the minimum supported version and the current one. The dependency audit from [Using Current Versions](USING-CURRENT-VERSIONS.md#tools).
* **Branch protection makes the policy mechanical.** The release branch (and ideally the integration branch) accepts changes only through a green pipeline. This is what turns "the release branch is never broken" ([Branch Policy](#branch-policy)) from a hope into a mechanism.
* **A red pipeline on a shared branch outranks feature work.** The same reasoning as the zero-warning policy: a pipeline that is allowed to stay red stops meaning anything.

That last norm is older than the tooling that now makes it easy. Mozilla's Tinderbox (one of the earliest continuous-build displays, retired in May 2014) was a scrolling page with a column for every machine building the current source and time running down the axis. Each checkout opened a box; build and tests filled it; completion closed its top. Green was the happy path. Yellow meant still building, or warnings. Red meant a broken build or failing tests, and red closed the tree: all hands on deck, no commits accepted except ones addressing the red. Decades of tooling later, the principle hasn't been improved upon: a red pipeline is the whole project's problem, and it outranks everything else.

CI needs credentials of its own: registry tokens, deploy keys. Those are injected by the CI platform's secret management, never stored in pipeline configuration; see [Handling Secrets § CI/CD Pipelines](HANDLING-SECRETS.md#cicd-pipelines).

---

The four principles above shape what a shared project *is*. The sections that follow (documentation, the deliverable spectrum, versioning, branch policy, git conventions, code review, changelog) are the recurring mechanics where all four principles meet.

## Documentation

Beyond the README and LICENSE, a shared project can need additional documents because it has multiple audiences.

* **TUTORIAL** (walkthrough for someone using it the first time). Different from the README: the README is a reference you scan; the tutorial is a path you follow.
* **ARCHITECTURE**: design rationale and internals. Why it works the way it does, where the extension points are, what trade-offs were made. Include things you decided **against** and why; don't make future contributors re-prove that a rejected path won't earn its keep. The audience is future contributors (including future you).
* **CONTRIBUTING** (how to fork, how to submit a PR, how to set up a development environment, what tools to run, what the branch policy is, what the commit conventions are). A new contributor should be able to go from clone to passing tests by following this file. This includes listing the **developer prerequisites**, the tools a contributor must have installed to work on the project (package manager, hook runner, environment manager, etc.). These are different from what a *user* needs to install the library, and the distinction must be explicit. Don't assume people have your tools installed; tell them what to install and how. And beyond the mechanics: **what contributions the project accepts** (which kinds of changes are welcome as direct pull requests, and which must begin as an issue and a discussion). Substantial new work always begins with that discussion, and the discussion is not free-form: it engages the project's stated goals ([Know and State Your Goals](#know-and-state-your-goals)). A proposal arrives as a How; the conversation's first job is to find its What and ask whether that What belongs among the project's goals at all. A contribution that meets every technical standard but serves no stated goal adds surface area, not capability ([Know and State Your Goals](#know-and-state-your-goals)).

The size of your project, the scope at which it will be shared, and your own judgment will help you decide which of these documents are needed. A missing doc update is the same as a missing test. If you change behavior, the docs must reflect it in the same branch.

### Examples

Good examples are the most underrated form of documentation.

* **State the intent up front.** Each example should open with what it's demonstrating, why that scenario is worth examining, and how it relates to the project's goals. Is this a strength? A known weakness? A case where an alternative shines because it optimizes for something different? The reader should know the thesis before they see the code.
* **Compare against alternatives.** Show the same task done with your library and without it (or with a competing approach). Let the reader judge. Compare against the alternative a user would actually reach for first, held to [Real Evaluation](#real-evaluation)'s bar: idiomatic code on both sides, no strawmen. The best examples start from what users actually want to do; only then show how your library does it **better**.
* **Each example earns its own verdict.** Your library isn't the best tool for every job. Showing where it isn't is [Real Admissions](#real-admissions) at work; it builds more trust than pretending it always is.
* **Include complexity metrics where they serve the comparison** (line counts, benchmark results). But an example never mints its own numbers: figures quoted in an example come from the project's reproducible [benchmarks](#benchmarks), so they can be regenerated when the code changes. A quoted number that no benchmark can reproduce is a claim, not a measurement.

An example is documentation; a benchmark is measurement. The two meet only as quotation.

### Documentation Systems

Markdown files in a repository are the right starting point. But when your public API spans many modules, when users need to search across reference material, or when you want API reference generated directly from docstrings, invest in a documentation site.

The essential property: the site generates content from source. Docstrings become API reference; Markdown guides become navigable pages. Documentation generated from code can't silently diverge from it.

Every ecosystem has options. In Python, [Sphinx](https://www.sphinx-doc.org/) is the established standard (deep ecosystem support, large plugin library, output in HTML and PDF), and [Great Docs](https://posit-dev.github.io/great-docs/) is a newer option with a lighter setup.

### LLM Context

Developers increasingly write code against libraries using an LLM as a coding assistant; whether or not you use these tools yourself, some of your users do. The LLM draws on training data that may be outdated or incomplete for your specific project: it may hallucinate function signatures, miss recent changes, or invent behavior entirely. Providing an `llms.txt` file (and the fuller `llms-full.txt` companion) gives those tools accurate, current context: a machine-readable summary of your project's API, design patterns, and conventions, maintained alongside your docs.

This is worth doing earlier than you might expect. If developers are depending on your library and writing code against it, some are already asking an LLM to help, and working from stale or invented information.

[Great Docs](https://posit-dev.github.io/great-docs/) generates both files automatically. For other toolchains, these are plain-text files you maintain alongside your other documentation.

## The Deliverable Spectrum

How much of the machinery that follows (versions, tags, changelogs, branch structure) a project needs depends on what kind of deliverable it is. Projects sit on a spectrum:

* **A book** is a project, but it is its own deliverable: a single artifact. Books aren't the subject of these standards; they anchor one end of the scale.
* **An informational artifact** (documentation, standards, a knowledge base) sits close to the book end. The content is the deliverable, and only the tip is meaningful: versioned releases, tags, and a CHANGELOG add noise, not value. It belongs under source control, but one branch carries the whole promise (its tip is always the deliverable), and only the most unusual cases need more.
* **An unshared artifact** (a personal script) might not even be under source control. It should be: source control is cheap, and "unshared" has a way of becoming "shared."
* **A tool for you and a few others** is in Git, and branches begin to help. This is the first point on the spectrum where a specific branch known to be "the good version" earns its keep.
* **Production code** (a library or a runnable application) is where reproducibility becomes the point. The artifact reaches a wider group, and that group ends up holding different incarnations of it. They need to know *which one* they have; you need to be able to build exactly *that one*, locally, on demand. And the next delivery may need to be "what they had, plus a minimal fix," or something brand-new from the tip. Either way, there must be a scheme for perfectly reproducible source versions and, wherever possible, perfectly reproducible binary artifacts built from them. Here, the good path is a well-known branch that promises durable reproducibility, and everything below applies in full.
* **Code that accepts changes from others** adds the last piece: feature branches as everyday workflow, and the full branch policy.

Find your place on the spectrum deliberately, shed the machinery your position doesn't need, and say in the README what you kept. An informational repo carrying release ceremony is noise; a shared library without reproducibility is a hazard.

## Versioning and Releases

Everything in this section presumes a project on the production end of the [deliverable spectrum](#the-deliverable-spectrum): an artifact whose consumers must know which incarnation they hold. Toward the book end, none of it applies.

### Single Source of Truth for Version Numbers

Use tag-driven versioning (in Python, hatch-vcs or setuptools-scm; every ecosystem has an equivalent). The version comes from git tags, not from a manually-edited string somewhere. One source of truth means zero chance of mismatch.

### Semver with Actual Meaning

* **Major**: breaking changes. Must include a migration guide in the changelog.
* **Minor**: new features, backward-compatible.
* **Patch**: bug fixes only.

If your major bumps don't come with migration guides, your versioning is decoration.

### Annotated Tags Only

Annotated tags are durable git objects with metadata (who tagged, when, why). Lightweight tags are just refs that can drift. Use `git tag -a` always.

### Releases are reproducible

Given the same tag, anyone should be able to build the same artifact. This means pinned dependencies (a committed lockfile), deterministic builds, and no manual steps that aren't documented.

### Versions reflect committed state, not the working tree

The version your build reports must depend only on committed git history (commits and tags), not on whether the working tree happens to be clean or dirty when the build runs. Tag-driven versioning tools (in Python, setuptools-scm and hatch-vcs) default to including a "dirty" suffix and bumping to a `.dev0` pre-release when the working tree has uncommitted changes: useful in some workflows, harmful for releases.

Concretely, tree-dirtiness leaking into the version causes:

* Tagged releases that report wrong runtime versions, when something dirties the tree at install time and the version file gets baked with the dev string.
* Lockfiles that churn nondeterministically: the embedded version field shifts on every uncommitted edit.
* Surprise version drift between developer machines.

Configure your versioning tool to ignore working-tree dirtiness. The version a user sees at runtime should match the tag they checked out, regardless of working-tree state.

### Version tags are permanent

Once pushed, never delete or move a version tag. If a tag landed on the wrong commit, create a new version. Consumers may have already pinned to it. This is one of the rules that [does not bend](WHAT-MATTERS-MOST.md#bending-the-rules): a moved tag cannot be un-pinned by consumers you don't know about.

### Keeping Dependencies Current

Staying current with dependencies and language runtimes is a trade-off with real costs on both sides. For a full discussion of the reasons to upgrade, the reasons to hold back, recommended cadence, and tooling, see [Using Current Versions](USING-CURRENT-VERSIONS.md).

## Branch Policy

A shared project needs a branch that consumers can always depend on. How much structure surrounds it is a [deliverable-spectrum](#the-deliverable-spectrum) question: an informational artifact collapses this model to a single branch carrying the release promise; production code that accepts changes from others needs all three.

* **Release branch**: always production-ready. Releases are tagged here. History is durable, never rewritten. Typically named `release`.
* **Integration branch**: where feature branches merge for testing. Not inviolate. History could be rewritten to recover from a catastrophic commit, though thorough testing of feature branches before they reach here makes this an essentially unreachable state. Consumers should not depend on this branch. Typically named `main`.
* **Feature branches**: disposable. Branch off release, merge to integration for testing, squash-merge to release when complete.

Integration and release converge in content rather than history: each feature reaches both branches (as a merge on integration, as a squash on release), so integration's history is noisier while its tree tracks release's. If integration is ever wrecked beyond repair, rebuild it from release; that is the catastrophe-only rewrite noted above. Feature-branch commit messages (the letters to the future maintainer of [Git Conventions](#git-conventions)) live on in integration's history; the squash message is their summary on the release line.

The key property: the release branch is never broken. If a consumer pins to it, they get working code.

## Git Conventions

Whatever the language, the repository itself is a shared artifact, and the same expectations apply to it. These standards assume that artifact is a Git repository: a declared choice, not an oversight. [Your Tools, Your Choice](#your-tools-your-choice) puts the version-control *client* on the personal side of its boundary; the repository and its history sit on the shared side, and for that shared substrate these standards pick the tool the world has already picked: Git won version control so completely that its vocabulary (commits, branches, tags, history) is the lingua franca of every ecosystem's shape. A project that chooses differently spends every collaborator's familiarity, and that bend needs a defense like any other. The principles beneath these conventions (durable history, immutable release identifiers, a line consumers can always trust) survive translation to any serious version-control system; the translating, though, is yours to do.

**Commit messages** use imperative mood: "Add feature" not "Added feature." The first line is a concise summary. The body explains the *why*: what motivated the change, what alternatives were considered, what the reader should know. A commit message is a letter to the future maintainer.

**Changes trace to stated needs.** The commit body (or the PR it belongs to) names the need the change serves: a ticket, an issue, a stated goal. Work that can't be traced to a stated need is the small-scale version of a project without a goal ([What Matters Most § Know and State Your Goals](WHAT-MATTERS-MOST.md#know-and-state-your-goals)): it can't be evaluated, only admired.

**Branch naming** should be consistent within your project. A common convention: `<category>/<summary>` where categories are `feature`, `bugfix`, or `hotfix`. Local branch names are whatever you want; only the remote name matters (see [Your Tools, Your Choice](#your-tools-your-choice)).

**After merging a PR, delete the source branch.** Keep the canonical repository pruned.

**No commented-out code in PRs.** Comment out whatever you need locally while debugging, but commented-out blocks of code must not appear in a pull request. Dead code is noise. If you might need it later, that's what version control is for.

## Code Review

The quality gates are machines; review is the human gate. By the time a person looks at a change, formatting, lint, types, and tests have already passed, so human attention goes where tools can't: whether the change solves the right problem, whether it creates the right expectations, whether its claims are honest, and whether its tests encode the promise. Those are the four principles, applied one change at a time.

Review spends the largest line item of [What Matters Most § The Standing What](WHAT-MATTERS-MOST.md#the-standing-what) (human time, attention, and effort) more directly than anything else in these standards, so everything here serves one aim: tools catch what tools can catch, problems surface while they're small, and the human receives only the work that needs a human.

**The submitter owes the reviewer:**

* **A reviewable unit.** One intent per pull request, as small as the intent allows. A PR that mixes a refactor with a behavior change hides the behavior change; a PR too large to hold in your head gets a shallower review, not a longer one. (See [Minimizing Divergence](MINIMIZING-DIVERGENCE.md); small units are also the cheapest to merge.)
* **The first review.** Review your own diff before requesting anyone else's time. You own every line you submit, no matter how it was produced ([Using AI § Responsibility](USING-AI.md#responsibility)); the reviewer is your second line of defense, never your first.
* **The why, stated.** The diff shows what changed; the description says what the change is for and links the need it serves: the ticket or issue where its What was agreed ([Git Conventions](#git-conventions)). A reviewer who must reverse-engineer the goal will review the wrong thing.
* **Green gates before review.** Never spend human attention on what the machines already catch. Where the project uses AI review, that is a gate too: run it and resolve its findings before requesting a human (the submitter's responsibility, like every other gate). The escalation is deliberate: deterministic tools first, AI second, humans last. Each rung costs more than the one below it, and the goal is to hand the human the smallest, highest-value piece of the work.
* **Visible bends.** Any deviation from these standards is marked and defended in the PR ([Bending the Rules](WHAT-MATTERS-MOST.md#bending-the-rules)).
* **A response to every comment.** A change, or a reasoned reply. Silence is neither.

**The reviewer owes the submitter:**

* **Understanding.** Approval means you understood the change and vouch for it entering the shared history. If you didn't understand it, you haven't reviewed it. Ask.
* **The What before the How.** First: should this change exist, and does it do the right job? Line comments on a change that solves the wrong problem are wasted precision. (See [Why, What, How](WHY-WHAT-HOW.md).)
* **Promptness.** An unreviewed PR is a diverging branch, aging like any other. And a submitter waiting on review is the most constrained resource, idling. Review is work; give it the priority of work, not the leftovers of it. All other things equal, closing open work outranks starting new ([Minimizing Divergence](MINIMIZING-DIVERGENCE.md)).
* **Blocking and non-blocking, distinguished.** Block on correctness, broken contracts, and principle violations. Offer preferences as preferences, and say which is which; a submitter who can't tell must treat everything as blocking.
* **Critique of the code, not the coder.** The review is about what the change does, not who produced it.

**How many reviewers?** In a single-maintainer project, a single review (your own) is all that's available, all that matters, and all that's needed: the submitter's obligations stand, the self-review *is* the review, and the language layer's audit checklist is its checklist. When more reviewers are applicable, the requirement is project-level policy, stated explicitly: these specific three reviewers; any two of these three; any three of all possible reviewers. Which policy matters less than that it is stated: a submitter should never wonder whose approval they are waiting for.

One feedback loop deserves pavement: a review comment you find yourself writing for the third time is not a comment; it's a rule. Encode it: a lint rule, a type constraint, a checklist line, a sentence in these standards. Review attention is the most expensive quality tool you have; spend it only on what nothing cheaper can catch.

## Changelog Discipline

Any project that produces versioned releases must have a CHANGELOG. The changelog is not a git log summary. It's a curated document for people who depend on your code.

Format: heading per release (`## v1.2.0 (YYYY-MM-DD)`). The first text under the heading is a sentence or short paragraph describing the main intent of the release: no bold label, just prose. Then bold section headers, as applicable:

* **Breaking:** what broke and why the break was necessary.
* **Features:** new capabilities.
* **Fixes:** what was wrong and what's fixed. Include ticket numbers where applicable.
* **Migration:** step-by-step instructions to update from the previous version.
* **Documentation:** doc-only changes worth noting.
* **Internal:** refactors, tooling, CI changes (nothing user-facing).

Write for the person upgrading, not for the person who wrote the code.
