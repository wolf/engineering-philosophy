# What Matters Most

Four things matter more than anything else in software work. They are not rules to follow — they are the foundations that make the rules make sense. Every guideline, every standard, every convention in these documents is an application of one or more of these principles at a specific scope.

They apply at every level: a shared library, a module, a function, a line of code. At the largest scopes they are stated explicitly — written down, measured, tested, documented. At the smallest scopes they are inherited and implicit — but still present, still load-bearing. Code that meets every style rule but violates these principles is bad code. Code that bends a style rule in service of these principles is probably right — but bending carries obligations; see [Bending the Rules](#bending-the-rules).

The four principles come first. Everything after them — [The Standing What](#the-standing-what), [How These Scale](#how-these-scale), [Bending the Rules](#bending-the-rules) — is framework, not additional principle.

For a framework on how to reason from these principles down to day-to-day decisions, see [Why, What, How](WHY-WHAT-HOW.md).

## Know and State Your Goals

Every piece of work exists to accomplish something. Know what that something is. Say it out loud.

Goals are not wishes. Know the difference between what you actually need and what you merely want. "It would be nice if..." is not a goal — it's a distraction wearing a goal's clothing. The things you actually need are fewer and harder than the things you want, and they deserve all of your attention.

A goal must describe something you can validate. If you can't define what "met" looks like — even in principle — it isn't a goal. It might be a value or a philosophy, and those matter, but they can't tell you when you're done or whether you've succeeded. Goals can.

Goals come in different shapes. Some are durable: a minimum performance bar, a correctness invariant, a compatibility guarantee. Some are ephemeral: a delivery constraint tied to a specific client or date, a migration that must be complete before another team can ship. Both are real goals, but ephemeral goals expire. Recognize them as such, and remove them when their conditions no longer apply.

Goals drive everything downstream. They determine what to build, how to design it, what to measure, and what to test. They tell you when you're done — not just "it works," but "it works, and now I can move on to the next problem." They also tell you when to stop: if the goal is met, stop building. If the goal can't be met, stop and say so. And they tell you when to ship: you will never be **done**, but there will come a point where you are useful — and useful, not done, is the bar. Don't wait — release when you are useful.

Goals must be prioritized. When goals conflict — and they will — priority is the tiebreaker. Without explicit priority, conflicts are resolved by whoever is loudest or most recent, which is not a strategy.

Goals require maintenance. Review them. A goal that has been met is no longer a goal. A goal whose conditions have changed may no longer be the right goal. Stale goals are deadweight — they distort priorities and justify work that no longer matters.

Every goal has both urgency and importance. New goals will be proposed — by clients, by managers, by yourself. Some will feel urgent. Urgency is compelling, but it is not sufficient. A goal that is urgent but unimportant is a trap. A goal that is both urgent and important demands immediate attention. Do not be tricked by urgency alone.

High-level goals can and will evolve. That's fine — the world changes, you learn things, priorities shift. But at any given moment, you should be able to point to the goal that justifies the work in front of you. If you can't, you might be solving the wrong problem.

At large scope, goals are written down: in a README, in a design document, in a ticket. At small scope, they're inherited — a function doesn't need its own mission statement, but it must serve the goal of the module that contains it. When you can't trace a piece of work back to a goal, something has gone wrong.

## No Surprises

If the thing you're building has an existing model — a shape that consumers already carry in their heads — satisfy that model. A Python package should look like a Python package. A REST API should behave like a REST API. A function called `get_user` should get a user.

If you're building something genuinely new, your job is to build a predictable model in the consumer's mind — and then satisfy it. Documentation, naming, structure, and behavior all work together to create expectations. Once those expectations exist, they are promises.

The test: when a consumer tries to do something new with your work — something you didn't explicitly document — their guess about how to do it should probably be right. If it isn't, you need to understand why. Either your model is inconsistent (fix it), or the consumer's mental model is wrong (fix your documentation), or the task genuinely doesn't fit (say so clearly).

No Surprises is not about simplicity. Complex systems can be unsurprising. It's about consistency, predictability, and honoring the expectations you create. A surprise is a broken promise — even if you never said the words.

## Honesty

Be honest. In everything.

Honest about your goals — what you're actually trying to accomplish and why. Honest in your documentation — what the code does, what it doesn't do, where it falls short. Honest in your measurements — real numbers from real scenarios, not cherry-picked results that tell the story you wish were true. Honest in your promises — don't claim stability you haven't earned, performance you haven't measured, or compatibility you haven't tested.

Honest in comparisons. If an alternative is better at something, say so. If your approach has weaknesses, document them. Every project has limitations; the ones that acknowledge them earn trust, and the ones that hide them lose it.

Honest about time. If you don't know how long something will take, say that. A wrong estimate believed is worse than uncertainty acknowledged.

Honest about your own motivations. Sometimes the interesting problem and the important problem aren't the same one. Sometimes the thing you want to build and the thing that needs building aren't the same thing. Knowing the difference — and choosing the important one — is integrity.

Honesty compounds. A team that is honest with itself about what's working and what isn't will improve. A team that isn't, won't — no matter how talented the individuals are.

## Tests and Measurement

You have goals. Tests show you are meeting them. Measurements show you are making progress toward them.

Tests are not bureaucracy. They are the mechanism by which you know your work is correct. A feature without tests is a claim without evidence. Changed behavior without updated tests is a promise you've stopped keeping.

Measurement is not vanity metrics. The goals tell you *what* to measure. If the goal is performance, measure latency and throughput — not lines of code. If the goal is reliability, measure failure rates and recovery time — not test count. Measure the thing that matters, not the thing that's easy to count.

Measurement includes measuring yourself against alternatives. If you know your goals, and you honestly measure your work against other approaches to the same goals, you might discover that the best path forward is someone else's code. That's not failure — that's the measurement doing its job. Work that shouldn't be done at all is the most expensive kind of waste.

At large scope, tests are explicit: a test suite, a benchmark suite, a measurement dashboard. At small scope, tests still cover the code — you don't test every line in isolation, but every line is reachable by tests that verify the behavior it contributes to. If code can't be reached by any test, ask whether it should exist.

---

That is the whole list — four. Nothing below this line is a fifth principle. What follows is the framework around them: the standing What every project inherits, how the principles express at every scope, and what happens when a derived rule works against the principle it serves.

## The Standing What

The four principles are Whys — values, held before any particular project exists ([Why, What, How](WHY-WHAT-HOW.md)). Whats are concrete, and one What stands above every project ever shipped, stated or not: **provide a solution with a positive return — it must cost less to create and sustain than the value it produces.** Every project-specific goal serves this one, and it is the most validatable statement in these documents: an inequality you can check. Your users are running the same equation on you ([Respecting Users](RESPECTING-USERS.md)). The standing What sits below the principles deliberately — the values gate how you may pursue it. You may not lie, surprise, or fudge your way to a return.

Count the cost side honestly. Its components:

* **Human time, attention, and effort** — the scarcest resources in software, and the largest line item. Machines are cheap and getting cheaper; people are not. Count both sides of the artifact: the effort to build and sustain it, and the effort every user spends on it — reading its docs, learning its API, recovering from its errors (value your users' time, this is critical).
* **Your assets** — data, code, credentials, your users' trust. The data users lend you belongs on this list with a twist: it is a liability you accepted, not an asset you own ([Respecting Users](RESPECTING-USERS.md)). Most costs are bounded; a leak's is not: stolen data, stolen code, stolen time, rotation and regeneration, opportunity handed to a competitor. That unboundedness is why the secrets rules are among the rules that [don't bend](#bending-the-rules) ([Handling Secrets](HANDLING-SECRETS.md)).
* **Divergence** — the gap between the tip of use and the tip of development. Its cost grows non-linearly; close gaps while they're small ([Minimizing Divergence](MINIMIZING-DIVERGENCE.md)).
* **Work that shouldn't exist** — the most expensive waste of all. If measurement shows the goal is already met, or met better by someone else's code, stop ([Tests and Measurement](#tests-and-measurement)).

Two consequences reach into every document here. First, **automate**: tests, pre-commit hooks, automated formatting, type annotations, CI — each trades a little up-front work for a recurring reduction in human effort, and each catches problems early, while the fixes are small and the humans involved are few — or one. Second, **prefer the simplest reasonable answer**: simplicity is not a fifth principle — it is what this section and No Surprises jointly demand. Every moving part beyond what the need requires costs understanding to acquire, maintenance to keep, and surprise to encounter — and while a complex system can be unsurprising, a simple one gets there at lower cost.

When a rule in these documents privileges the human — hooks that stay fast, review that arrives promptly, tests too slow for the commit loop moved to CI, code that reads at review speed — this is why.

## How These Scale

These four principles don't switch on at some threshold of project size. They are always present. What changes is how explicitly they are expressed.

A shared library states its goals in the README, publishes honest benchmarks, defines its public surface to avoid surprises, and runs a comprehensive test suite. Every principle is visible and formal.

A module within that library inherits the library's goals, maintains the library's conventions (no surprises), doesn't misrepresent what it does (honesty), and is covered by tests that verify its contract.

A function inherits the module's goals, has a name and signature that tell the caller what to expect (no surprises), does what its docstring says (honesty), and is exercised by the tests.

A single line of code serves the function's purpose, does the obvious thing, doesn't lie, and is reachable by a test.

The principles don't change. The explicitness does. At the top, you write them down. At the bottom, you live them.

## Bending the Rules

Everything downstream of these four principles — every standard, convention, lint rule, and checklist item in these documents — is derived. Rules are tools: formalized shortcuts that usually lead to the goals, which is exactly why they're worth having. *Usually.* When following a rule in a specific situation works against the principle it exists to serve, it is the rule that bends, not the principle. That isn't a loophole in these standards; it's their design. You'll find it stated throughout — "a compass, not a mandate," "a default, not a mandate," and, in the Python layer, "use `# noqa` and a comment explaining why." Those are all the same idea. This section is that idea, stated once, with its conditions.

A bend is legitimate only when it is **visible** and **defended**.

**Visible** means the deviation is marked where the next reader will encounter it. A suppressed lint rule carries a comment. A surprising structure is explained at the point of surprise. A departure from a shared-project standard is called out in the PR, the README, or wherever a consumer would otherwise trip on the broken expectation. A silent bend is indistinguishable from a mistake — the next person to touch the code will either "fix" it back or, worse, learn that the rules are quietly optional. Both outcomes destroy the thing the rules exist to build. An invisible bend violates No Surprises even when the bend itself was correct.

**Defended** means the mark says which goal or principle the bend serves, specifically enough to be challenged. "This ordering matters because module B registers handlers at import time" is a defense. "It was faster this way," "the deadline was tight," and "I prefer it" are not — those aren't bends in service of a principle, they're just breaks. If you can't name what the rule would have cost here, follow the rule. And a defense must survive challenge: a bend that can't be defended when someone questions it gets un-bent.

Why insist on visibility? Because visible bends are how the standards improve. When the University of Michigan laid out the Diag, the paths weren't guessed at from a drawing board — the grass was left open, students walked where walking made sense, and the worn dirt trails showed exactly where the pavement belonged. Every visible, defended bend is a footprint. One bend is a judgment call. The same bend, defended the same way, appearing in project after project is a worn path: the standard is wrong there, and the fix is to pave the path — change the rule to match where people demonstrably need to walk. That feedback loop only works if the footprints can be seen. Hidden bends deprive the standards of the evidence they need to evolve, which is how a living standard hardens into the edict it was never meant to be.

**Some rules don't bend.** A bend is defensible only because a bad one is cheap: it is visible, it can be challenged, and it can be undone. A few rules guard against harm that is irreversible — a secret that reaches version control cannot be unleaked (see [Handling Secrets](HANDLING-SECRETS.md)); a version tag that consumers may have pinned cannot be moved (see [Standards for Shared Projects § Version tags are permanent](STANDARDS-FOR-SHARED-PROJECTS.md#version-tags-are-permanent)). Where the downside of a wrong bend is permanent and unbounded, no defense can survive challenge, so the calculation is already finished: those rules are stated as absolute. That is not an exception to this framework; it is its conclusion.

So bend when the principles demand it. Mark the bend. Defend it. And when you notice the same worn path for the third time, don't keep bending — bring it back here and change the rule.
