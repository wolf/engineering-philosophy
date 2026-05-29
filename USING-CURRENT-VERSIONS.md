# Using Current Versions

Keeping dependencies and language runtimes current is a trade-off, not a rule. There are real reasons to stay current and real reasons to hold back. The goal is to make this decision deliberately — informed by your project's goals, your dependencies' track records, and the tools available to you — rather than by inertia or habit.

This document is language-agnostic in principle. Where Python-specific tooling or conventions apply, they are noted as such.

## Reasons to Stay Current

**No balloon payments.** Every version you skip accumulates migration work. If you upgrade incrementally, each step is small and well-documented. If you wait until an upgrade is forced — by a security advisory, a dropped support window, or a dependency that requires the newer version — you pay for every skipped version at once, under pressure, with less room to test carefully.

**Security patches arrive naturally.** When you track current releases, security fixes land as part of your normal update cycle. You already have the patch before the CVE makes the news.

**Performance improves over time.** Language runtimes, libraries, and tools generally get faster. Staying current means these gains arrive without dedicated effort.

**New features become available.** A newer version might offer capabilities that meaningfully simplify your code or enable functionality you need. Staying current means you can adopt them when they help, rather than discovering they exist but are out of reach.

**[No Surprises](WHAT-MATTERS-MOST.md#no-surprises) for new contributors.** A developer joining the project encounters the APIs and idioms they already know. Old versions mean old patterns — workarounds for bugs that were fixed, polyfills for features that are now built in, documentation that no longer matches the version in use.

**Dependency compatibility.** Current packages are tested against other current packages. When you fall behind on multiple dependencies at once, you risk version resolution conflicts — where A requires B>=2.0 but C pins B<2.0. This is distinct from the balloon payment: it's not just accumulated work, it's a puzzle that may not have a clean solution short of upgrading everything simultaneously.

**Community support.** Bug reports on old versions are often met with "please upgrade first." Answers on forums and in documentation assume recent APIs. Staying current keeps you in the mainstream where help is available.

## Reasons to Hold Back

**Proven stability in your environment.** A version that has been running in your stack for weeks or months is a known quantity. "Works on the latest release" and "works in our deployment" are different claims. There is value in letting others discover the rough edges first.

**Upgrading too quickly can adopt versions with undiscovered problems.** New releases occasionally introduce security vulnerabilities, regressions, or subtle behavior changes that take time to surface. A brief waiting period lets the community shake these out before you're exposed.

**Migration cost is real, even when smooth.** Even a clean, non-breaking upgrade takes time to test, validate, and deploy across your projects. That time competes with feature work and other goals. This is not laziness — it is a legitimate trade-off, and it should be weighed honestly.

**Coordinating multiple projects.** When several projects cooperate — sharing libraries, running in the same environment, or deploying together — holding them at the same version avoids compatibility work. Upgrading one project may force upgrades across the group, multiplying the cost and risk.

**Immovable external constraints.** Sometimes you cannot upgrade. You are a plugin for an application that runs your code in-process and bundles a specific library version. You deploy to an environment that provides a fixed runtime. A downstream consumer has pinned your package at a version that limits what you can depend on. These constraints are not choices — they are facts that must be respected.

## Cadence

The question is not *whether* to upgrade, but *when*.

Upgrading the day a release drops puts you on the bleeding edge — first to encounter regressions, first to hit undocumented behavior changes, first to file bugs that will be fixed in the next patch. This is rarely the right posture for production code.

Never upgrading is worse. The costs accumulate silently — in security exposure, in growing distance from the ecosystem, in the eventual forced migration that arrives at the worst possible time.

The sweet spot is a deliberate lag: current enough to stay in the mainstream, delayed enough to let early adopters find the problems. For most dependencies, a window of 60 to 90 days after a release gives the community time to surface issues while keeping you close enough that each upgrade is incremental. For security-sensitive dependencies, the window may need to be shorter — or eliminated entirely when an advisory demands it.

This is a default, not a mandate. Some dependencies have excellent track records and can be upgraded sooner. Some have a history of regressions and deserve more caution. Your project's goals and risk tolerance should inform the decision — but having a default cadence means upgrades happen as part of your rhythm rather than being perpetually deferred.

Language runtimes deserve their own cadence. They move on annual release trains with published support windows, and the forcing function is usually the end-of-life calendar rather than features: adopt a new runtime once its first patch releases have landed, and schedule the drop of an old one before its EOL date, not after. The freeze-a-branch escape hatch for consumers who can't follow is discussed in the language layers (e.g., [Python Standards § Version Policy](languages/python/STANDARDS.md#version-policy)).

## Tools

Don't rely on memory or good intentions to track what's out of date. Use tooling to make the state of your dependencies visible and actionable.

**Dependency auditing.** *(Python-specific)* Recent versions of `uv` include `uv audit`, which checks your dependency tree against known security advisories. Run it regularly — as part of CI if possible, manually if not. An audit that runs automatically catches problems that a quarterly review would miss.

**Lockfile diffing.** When you update dependencies, review what changed. Your lockfile (`uv.lock`, `pixi.lock`, or equivalent) is a precise record of every resolved version. Diff it before and after an update. A dependency that jumped three major versions deserves more scrutiny than one that bumped a patch.

**Dependabot, Renovate, or equivalent.** Automated PR tools that propose dependency updates on a schedule. They don't make the decision for you, but they make the current state visible and keep upgrades from falling off the radar. Evaluate whether these are available and appropriate for your hosting platform.

**Release notes and changelogs.** Before upgrading, read them. A changelog that documents breaking changes, deprecations, and migration steps tells you what to expect. A release with no changelog is a warning sign — proceed with more caution, not less.

## See Also

* [Choosing Dependencies](CHOOSING-DEPENDENCIES.md) — the decision that precedes this document's: whether a dependency should exist at all.
* [Minimizing Divergence](MINIMIZING-DIVERGENCE.md) — the general principle behind upgrade cadence: the tip of use should not drift far from the tip of development, because the cost of closing the gap grows non-linearly.
