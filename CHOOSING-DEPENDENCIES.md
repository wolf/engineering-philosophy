# Choosing Dependencies

Every dependency is code you ship but did not write and do not control. Taking one is a real design decision, and both reflexes get it wrong: reflexively pulling in a package for every task, and reflexively writing everything yourself. The discipline is [Why, What, How](WHY-WHAT-HOW.md): the functionality you need is the What; any particular package is one How among several (the standard library, a different package, or code of your own).

The currency in which every option is priced is the one that governs all of these standards: human time, attention, and effort, the most constrained resources in software work ([What Matters Most § The Standing What](WHAT-MATTERS-MOST.md#the-standing-what)). A dependency might be applicable when it reduces the total human effort your project will spend (writing, understanding, upgrading, debugging), not merely the effort of writing the first version. Reducing effort makes a candidate; it doesn't make a decision. The candidate must **also** deliver the needs of the What it serves, and satisfy the rest of the constraints this document walks through: fit, health, license, and the cost to your clients.

This document is language-agnostic in principle; where Python examples appear, they are noted as such.

## Weigh the Need

Two questions size the decision before any candidate is examined:

* **At how many call sites will you use this functionality?** One call site argues for the lightest answer available. Functionality you'll call from everywhere justifies real machinery, and deserves the most scrutiny, because it will be the hardest thing to replace.
* **How fundamental is it to your process?** The periphery tolerates imperfect fits and cheap answers. The core of what your project *does* is where a dependency's qualities (and its defects) compound.

## Survey the Field

* **The standard library first.** It is already installed, already versioned with your runtime, already trusted, and it will still be maintained in ten years. If it covers the need (even at 90%), the bar for reaching past it is high.
* **Third-party candidates, plural.** If one library provides it, others probably do too. Compare before committing; the first search result is a candidate, not a decision.
* **Your own code.** Is this in your wheelhouse? Be honest in both directions ([What Matters Most § Honesty](WHAT-MATTERS-MOST.md#honesty)): date arithmetic, cryptography, and parsers are famous traps for the confident. But a forty-line function squarely in your domain is often a better answer than a forty-module framework, and it can be exactly the shape you need.

Standard-library-first sets the bar; it doesn't forbid clearing it. A worked example, from these standards' own Python layer: Python has `logging` built in, yet the [Python layer](languages/python/STANDARDS.md#logging) mandates `loguru`; it has `argparse`, yet the layer mandates `typer`. Those recommendations clear the bar precisely because they answer this document's questions well: the functionality is fundamental and the call sites are many (logging and the CLI surface run through everything); the shape fit is far better (`typer` turns the type-annotated functions these standards already require into a CLI; `loguru` removes `logging`'s configuration ceremony); and each package is essentially *only* the thing needed, widely adopted, and actively maintained.

`rich` illustrates one more lesson: it rides along as `typer`'s own dependency, so using it costs nothing you weren't already paying: the marginal cost of a package already in your tree is nearly zero.

## Judge the Fit

* **Shape.** How close is each candidate to what you actually need? A dependency whose model matches yours disappears into the code; a near-miss shape means adapters, wrappers, and translation at every call site: friction you will pay forever.
* **Fitness.** Does it meet your needs where they're measurable (performance, memory, startup cost)? Your goals say what matters; measure the candidate against them, not against its own benchmarks ([What Matters Most § Tests and Measurement](WHAT-MATTERS-MOST.md#tests-and-measurement)).
* **The slice you'd use.** Is your need the whole thing the package does, or a small corner of something huge? Depending on a giant for one function buys the giant's full cost (install weight, upgrade churn, security surface, transitive dependencies) for a sliver of its value.

## Judge the Package

* **How fast is it changing?** A fast-moving package makes your upgrade cadence inherit its churn ([Using Current Versions](USING-CURRENT-VERSIONS.md)). An abandoned one makes you inherit its bugs.
* **And is stillness abandonment, or completion?** Some packages are *finished*: small, stable scope; nothing left to fix. Stillness is only a warning sign when there is evidence of neglect: issues accumulating without answers, breakage on current runtimes left unaddressed. Judge the silence by what it ignores.
* **License.** It must be compatible with your project's license, and with your clients' use of your project. A license problem discovered late is a rewrite.

## Count the Cost to Your Clients

If you're building a library, your dependencies are not only yours. **Every package you require is a requirement you transitively force on your clients**: its version ranges must intersect theirs, its platform limits become your platform limits, its install weight lands in their environments, and its conflicts become support tickets with your name on them. An application answers only to itself; a library answers to everyone who imports it. Be correspondingly more conservative.

## Vendoring

Between depending and writing it yourself lies vendoring: copying a dependency's source into your own tree. It is the right tool in narrow circumstances: the slice you need is small and stable (one function from a giant), the package is finished but you can't justify its weight, or a library can't afford to force the requirement on its clients. Vendored code answers the transitive-cost question decisively: your clients never see it.

The obligations are real. The license must permit vendoring, and its notice requirements must be honored: vendored code keeps its copyright. Upstream fixes stop arriving; you have adopted maintenance. So record where the code came from and which version, both to honor the license and so a future maintainer can check upstream for fixes. And vendored code is invisible to dependency tooling (audits and update scanners don't see it), so that record is the only trail.

## Leave Yourself an Exit

The decision is rarely permanent in principle, but it can become permanent in practice. The exit cost of a dependency is roughly the number of places that touch it directly. When the shape is imperfect, or the package is young, or your trust is provisional, confine it behind a boundary of your own so that most of your code doesn't know it exists. This loops back to the first question: many call sites make a dependency load-bearing, and load-bearing dependencies are chosen, not accumulated.

---

None of this is new principle; it is the standing ones meeting at a package boundary. [No Surprises](WHAT-MATTERS-MOST.md#no-surprises) favors the candidate a new contributor already knows and the shape that disappears into your code. [Honesty](WHAT-MATTERS-MOST.md#honesty) governs the wheelhouse question and whose benchmarks you believe. Your [goals](WHAT-MATTERS-MOST.md#know-and-state-your-goals) decide what fitness even means. And running through every question is the preference [The Standing What](WHAT-MATTERS-MOST.md#the-standing-what) states: the simplest reasonable answer (the fewest moving parts that genuinely meet the need), because every part beyond those costs understanding, maintenance, and surprise.

## See Also

* [Using Current Versions](USING-CURRENT-VERSIONS.md): the ongoing side of this decision (once a dependency is taken, keeping it current).
* [Minimizing Divergence](MINIMIZING-DIVERGENCE.md): why the gap between what you use and what upstream ships grows more expensive the longer it stands.
