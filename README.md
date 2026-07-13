# Engineering Philosophy

My personal engineering standards (how I set up, write, test, and ship software), built around four principles: **Goals**, **No Surprises**, **Honesty**, and **Tests and Measurement**.

These standards apply to my own projects and to contributions others offer to them. They are shared here for anyone in the open-source world who finds value in them.

**For new projects**, these standards are a foundation to build on from day one. **For existing work**, they are a compass, not a mandate: small, steady steps in the right direction, taken as you touch the code (the practice is spelled out in [Standards for Shared Projects § Adoption](STANDARDS-FOR-SHARED-PROJECTS.md#adoption)). No one is expected to stop and rewrite a working codebase to comply. The standards themselves keep evolving as practice teaches more; they will never fully satisfy me, but they are already useful, and useful is [the bar for shipping](WHAT-MATTERS-MOST.md#know-and-state-your-goals).

In shape, this corpus is a set of Markdown documents, an informational artifact on [the deliverable spectrum](STANDARDS-FOR-SHARED-PROJECTS.md#the-deliverable-spectrum). It keeps exactly the machinery that position needs and no more: one `release` branch whose tip is always the deliverable; no versioned releases, no tags, no changelog. Read cover to cover, it is a bit under two hours; [What Matters Most](WHAT-MATTERS-MOST.md) alone (the foundation) is ten minutes.

## Goals

Every piece of work should know its Why and its What ([Why, What, How](WHY-WHAT-HOW.md)); here are this corpus's own.

The Why:

* I value sharing knowledge, and the harder-won the knowledge, the better. (I'm paying it forward!)
* Engineering is the thing I do, and this is how I do it. It took a long time to get good, so it is hard-won; it has proven effective, so it is worth sharing.
* The thing worth sharing: the four principles of [What Matters Most](WHAT-MATTERS-MOST.md), four decades of living them, and some knowledge to help you make them concrete in your own work.

The What, the goal of this corpus: **provide helpful, battle-tested, immediately usable engineering guidance that applies at every scale (a single engineer, a team, a company).**

Each of those words is a commitment:

* **Battle-tested**: nothing enters these documents that practice hasn't backed. Where I haven't seen enough, the corpus stays silent ([What This Is Not](#what-this-is-not)).
* **Immediately usable**: a reader can act the same day they read: start a project from the [samples](languages/python/samples/), audit existing code with the [checklist](languages/python/AUDIT-CHECKLIST.md), adopt any standard without translation.
* **At every scale**: the principles hold from a solo repository to a company's shared codebase; only how explicitly they are expressed changes ([What Matters Most § How These Scale](WHAT-MATTERS-MOST.md#how-these-scale)).

When these conflict, battle-tested outranks the rest: guidance without evidence is how bad practice spreads.

And the promise, if you leverage these ideas: you will produce more capable and more robust results; you will spend less effort doing it; and what you build will better fit the need, and adapt more easily as your clients' needs grow.

## The Documents

The documents form a hierarchy; each layer makes the one above it concrete for a narrower scope:

* **[What Matters Most](WHAT-MATTERS-MOST.md)**: The four principles that ground everything (Goals, No Surprises, Honesty, and Tests and Measurement) plus the framework around them: the standing What (a positive return, priced mostly in human effort), how the principles scale, and what it means to bend a rule in their service. Read this first.

* **[Standards for Shared Projects](STANDARDS-FOR-SHARED-PROJECTS.md)**: Language-agnostic baseline for any project other people depend on. Applies the four principles to project shape, documentation, API design, logging, CLI conventions, versioning, testing, and benchmarks.

* **[Language Standards](languages/)**: Per-language application of the layers above: concrete tooling, project layout, style, and conventions, plus a language-specific audit checklist and starter samples. Available: [Python](languages/python/README.md).

### Companion Documents

* **[Why, What, How](WHY-WHAT-HOW.md)**: A thinking framework for working at the right level of abstraction. Why the problem exists, what solution is needed, and only then how to implement it.
* **[You](YOU.md)**: The question the rest of the corpus isn't qualified to answer (who to be) and what I can honestly offer in its place: who I strive to be. My values, my Whys, and the daily Whats and Hows I derive from them.
* **[Using AI](USING-AI.md)**: AI as a development tool. Where it earns its keep, the understand-every-line bar for generated code, and the kinds of code never to accept from it. Where AI is applicable and acceptable (both to you, and to the upstream consumer) and you decide to use it, this document explains your responsibilities.
* **[Minimizing Divergence](MINIMIZING-DIVERGENCE.md)**: Why long-lived feature branches and version-pinned consumers are the same problem, and what to do about both.
* **[Respecting Users](RESPECTING-USERS.md)**: The user's side of the ledger; your artifact must win *their* ROI equation too. Their privacy (collect less, keep less, really encrypt, never your own), their mistakes (the artifact sets the exchange rate), and telemetry (overt and opt-in).
* **[Choosing Dependencies](CHOOSING-DEPENDENCIES.md)**: Whether a dependency should exist at all. Weighing the need, surveying the standard library and the alternatives (including your own code), judging fit and health, the transitive cost to your clients, vendoring, and keeping an exit.
* **[Using Current Versions](USING-CURRENT-VERSIONS.md)**: Trade-offs of keeping dependencies and language runtimes current. Why to upgrade, when to hold back, and how to find the right cadence.
* **[Handling Secrets](HANDLING-SECRETS.md)**: Non-negotiable rules, proven practices, and open scenarios for managing credentials and sensitive configuration.
* **[Cross-Platform Development](CROSS-PLATFORM.md)**: When and how to support multiple platforms. The "runs on" vs. "editable on" distinction, costs, and mechanics.

## Who This Is For

Me, first: these are the standards I hold my own work to, and the ones I ask of contributions to my projects. Beyond that, anyone who wants a coherent, well-reasoned standard for how to set up, write, test, and ship software. They were developed in practice and refined over time, not assembled from first principles in a vacuum.

They are opinionated. Every choice was made deliberately, and the reasoning is in the documents. If you disagree with a choice, the reasoning is there to argue with.

## What This Is Not

This entire collection of guidance is my personal conclusions and philosophy distilled from over four decades of practice. It is not:

* **A consensus standard**: it is valuable because it is coherent, not because it is universal.
* **A lint configuration**: the tools enforce the mechanics; these documents govern what tools can't judge.
* **A course**: it assumes a working engineer; even [You](YOU.md) shows who I strive to be, never who you must become.
* **A process framework**: it governs the artifacts of engineering, not the calendar of it.
* **Complete coverage**: it speaks only where I have seen enough to form opinions worth saying out loud, and stays silent elsewhere.
* **Tool-neutral**: it speaks in the vocabulary of my tools (to the least extent I can manage), Git above all, and doesn't translate for others. Where a tool is a stated default, you can bend it with a defense; the substrate is simply assumed.

This might not be what you need or want, or maybe you just don't agree with it. That doesn't make either of us wrong. It just means I wasn't able to help you this time.

## Participating

This corpus is personal: the opinions in it are the point, and I maintain them. That shapes what participation looks like here:

* **Small corrections are welcome directly as pull requests.** Typos, broken links, and factual drift: a tool renamed, behavior changed in a newer version, a reference gone stale. If it's wrong on its face, a PR is the fastest path.
* **Everything bigger begins as a conversation.** Every choice here was made deliberately, and the reasoning is written down precisely so you can argue with it. A proposed change is a How ([Why, What, How](WHY-WHAT-HOW.md)); open an [issue](https://github.com/wolf/engineering-philosophy/issues) that starts from its What (the gap it fills, the principle it serves, the goal it advances) and engages the written reasoning. When the conversation lands on a change, a PR referencing that issue is welcome. A PR beyond small corrections that didn't start as an issue will be declined automatically, not because the change is unwanted, but because the argument comes first.
* **Worn paths are the most valuable report.** If you've adopted these standards and find yourself bending the same rule, the same way, for the same defensible reason, in project after project: that is exactly the evidence [Bending the Rules](WHAT-MATTERS-MOST.md#bending-the-rules) exists to collect. Tell me where the pavement is wrong.
* **Forking is a fine outcome.** These standards are MIT-licensed. If your philosophy genuinely diverges from mine, the right move may be a corpus of your own: take whatever serves you.

To be explicit about governance: I decide what merges; a philosophy maintained by committee stops being one. But where I'm wrong, I **want** to learn better. The channels above are real; bring me your reasoning and I will engage with it, not just answer yes or no.
