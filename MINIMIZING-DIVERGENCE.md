# Minimizing Divergence

The tip of **use** should not drift far from the tip of **development**. Every day they diverge, the gap grows; the cost to close it grows faster.

This is one principle with two common manifestations: long-lived feature branches, and consumers pinned to old versions. They look different on the surface, but they are the same dynamic: work is happening at the head while something else stays behind, and the longer that lasts, the more painful the eventual reconciliation.

## Why the Cost Is Non-Linear

A branch one day old can be merged in minutes. A branch three months old may take longer to merge than the original work took to write. A consumer pinned one minor version behind can usually upgrade with a number bump. A consumer pinned three years back may need a multi-week migration, or may never upgrade at all.

The cost is non-linear because the work that accumulates while you wait is not additive; it interacts. Two refactors that each took a day to do can take a week to merge together. Two breaking API changes with clean individual migration paths can compose into a state with no obvious path at all. Behavior that quietly changed in version A can interact with behavior that quietly changed in version B in a way no one anticipated. Each piece of divergence multiplies the others.

The right time to close the gap is always "now, while it's small."

## Long-Lived Feature Branches

A feature branch is a private divergence from the shared line of development. While it lives, every change that lands on the integration and release branches is a change yours must eventually absorb. Your branch ages even when you don't touch it.

Defenses:

* **Finish before you start.** All other things equal, closing open work (review it, then merge it or decline it) outranks writing new code. An open branch diverges a little more every day it waits; code not yet written diverges not at all. The meter runs only on work in flight.
* **Incremental delivery.** Break work into steps where each completed step is a valid merge point: the program runs, tests pass, documentation matches behavior. Merge each step before starting the next.
* **Frequent rebases or merges from upstream.** If you cannot merge out yet, at least keep up with what's underneath you. Reconciling weekly is much cheaper than reconciling quarterly.
* **Push back on workflows that produce long-lived branches.** A "big feature branch" is rarely the only option. It is usually possible to land scaffolding, infrastructure, and refactors on the integration branch first, leaving only the core feature for the branch.

## Consumers Pinned to Old Versions

A consumer pinned to an old version of one of your libraries is a divergence between what you ship and what they use. While the pin holds, every release you make is one more version they must eventually traverse.

This is a relationship: your decisions and theirs both contribute.

As a library author, the things in your control:

* **Predictable cadence.** Releases that follow a known rhythm, rather than long droughts followed by floods, make it easy for consumers to plan their upgrades.
* **Honest changelogs.** Breaking changes documented with migration guides make it possible to skip versions safely. Breaking changes hidden in patch notes make every upgrade a research project.
* **Deprecation discipline.** Old names that keep working with warnings give consumers a runway to migrate. Silent removals force them to discover the break in production.

As a consumer, the things in your control:

* **A deliberate upgrade cadence.** See [Using Current Versions](USING-CURRENT-VERSIONS.md) for the full discussion. The short form: don't pin and forget; treat upgrading as ongoing work, not an occasional project.
* **Treat pin upgrades as load-bearing.** When a pin is far behind the tip, closing the gap is not a chore to defer; it is the same as merging an old branch, and it gets harder every day.

## When Divergence is the Right Choice

Sometimes divergence is the right call. A long-running release branch holds a stable version for clients while development continues elsewhere. A feature branch may genuinely lack mergeable intermediate states. A pin may be held deliberately because the cost of upgrading exceeds the benefit and the pinned version is internally supportable.

These cases are real, and the principle does not forbid them. What it does is make the cost visible. If you choose divergence, choose it knowingly, with a plan for how it ends. Divergence without an exit strategy is technical debt that compounds silently.

## See Also

* [Using Current Versions](USING-CURRENT-VERSIONS.md): applies this principle to dependency upgrades (the trade-offs of staying current, and how to find the right cadence).
* [Standards for Shared Projects § Branch Policy](STANDARDS-FOR-SHARED-PROJECTS.md#branch-policy): applies this principle to branch structure (which branch consumers depend on and where features integrate).
