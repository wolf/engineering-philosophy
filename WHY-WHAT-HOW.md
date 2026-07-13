# Why, What, How

Every software decision lives at one of three levels.

At the top: *Why*, the engineering principles and values your work is built on. These are the commitments that don't change with project or technology: that code should do what it promises, that behavior should be unsurprising, that systems should be maintainable and testable. These are established in [What Matters Most](WHAT-MATTERS-MOST.md). They are not goals you achieve; they are truths you act from.

In the middle: *What*, the specific outcome a piece of work needs to achieve. A What is concrete and testable: a behavior the system must exhibit, a property it must have, a contract it must fulfill. A What is a *goal* in the sense of [What Matters Most § Know and State Your Goals](WHAT-MATTERS-MOST.md#know-and-state-your-goals): validatable, prioritized, and maintained. It must serve the Why. And it does something crucial: it *constrains* the How. A clearly stated What becomes a measuring stick for every implementation you might consider.

At the bottom: *How*, the implementation. The specific algorithms, data structures, libraries, patterns, and code you use to reach the What.

**The direction matters.** Good engineering reasoning runs downward: Why → What → How. Starting at Why gives you the full solution space. One Why can support many possible Whats; one What can be reached by many possible Hows. Specifying the What closes the space down, not arbitrarily, but by the thing that actually needs to happen. From there, evaluating Hows is straightforward: does this approach actually deliver the required behavior? Not "is this the conventional pattern?" Not "is this what we've done before?" Does it produce the What?

**The common error is starting at the bottom.** In software, implementation details get treated as requirements. "We need an index on this column" is a How. "The system must respond to users in under a second" is a What: concrete, testable, project-specific. "The system respects its users' time" is a Why: a commitment that outlives any one requirement. When "add an index" gets written into the spec, you've closed off a cache, a query rewrite, denormalization, and a dozen other options, before you've understood what property the system actually needs.

Starting at How collapses your options before you've understood the problem. Worse, it lets you build a correctly-implemented How that serves the wrong What, in service of the wrong Why. The code works. It just shouldn't have been written.

**The professional implication.** When a stakeholder brings you a problem, they almost always specify the How. *Add a button. Use a message queue. Cache this endpoint.* That How is usually wrong, not because stakeholders are foolish, but because the reason they need you is that you're better at finding the right How than they are. Your first job is to push back to the What: does the specified How actually achieve the required behavior? Your second job is to push to the Why: does that behavior actually serve the engineering principles you're committed to?

You rarely get to Why from outside. Stakeholders don't volunteer it, and often haven't articulated it even to themselves. But when you find it, you can deliver something they didn't know to ask for, and couldn't have asked for, because they were thinking at the wrong level.
