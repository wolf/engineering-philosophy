# Using AI

AI is a legitimate and powerful development tool. It is also a choice. Some developers use it extensively, some selectively, and some not at all — for reasons ranging from the practical to the principled. All of these are compatible with these standards: nothing here requires you to use AI, and nothing here excuses the code you produce with it. The choice is not only yours, either — a project you contribute to may restrict or forbid AI-generated contributions, and its policy governs. What follows applies when you choose to use AI and the project accepts it.

## Where It Earns Its Keep

Where AI tends to earn its keep is the work that surrounds code: managing tickets, creating pull requests, wording commit messages, writing docstrings, drafting documentation, and writing tests. What these tasks share: verifying the result is cheap relative to producing it — you can judge a commit message or a docstring completely in the time it takes to read it. AI saves the most, at the least risk, where your verification costs least ([What Matters Most § The Standing What](WHAT-MATTERS-MOST.md#the-standing-what)). These are also tasks it often does better than a human doing them in a hurry between other work.

Generated code sits at the other end of that scale: the work is larger, and so is the cost of truly verifying it. That is why it gets its own bar.

## Generated Code

AI can also generate code. When you let it, there is a bar you must meet: **you must understand every line.** What it does, how it does it, why it exists. This is not a rubber-stamp review — it is the same standard you would hold yourself to if you wrote it by hand. The same standard you hold others to when you participate in [code review](STANDARDS-FOR-SHARED-PROJECTS.md#code-review).

Specifically, do not let AI give you code that is:

* **Unneeded** — solving problems you don't have, adding abstractions that don't earn their keep, including defensive code for cases that can't arise
* **Non-idiomatic** — fighting the language instead of using it, reimplementing what its standard library already provides
* **Using the wrong data structures** — a list where you need a set, a loose mapping where you need a structured type, a class where you need a function
* **Solving the wrong problem** — confidently implementing something adjacent to what you actually asked for
* **Carrying the wrong complexity** — O(n²) where O(n) is straightforward, nested loops where the language offers a direct construct
* **Testing implementations instead of promises** — tests coupled to private methods, internal state, or call counts rather than observable behavior (see [Standards for Shared Projects § Testing](STANDARDS-FOR-SHARED-PROJECTS.md#testing))

The [language standards](languages/) make this list concrete for each language.

## Responsibility

No matter how you came to provide the content of a pull request — writing it yourself, copy/pasting from Stack Overflow, auto-complete in your IDE, or a full-blown AI — the buck stops with **you**. You own the code. You are vouching for it. You must understand every line and the total flow. You are the first line of defense: is the code you are about to push good enough? Does it meet the standards? Does it do the (right) job, in the right way? It is **you** who must defend it, debug it, and champion it. AI is fast and tireless, but it does not know what matters — you do. In this particular way, AI is absolutely not a shortcut.

These standards — from [What Matters Most](WHAT-MATTERS-MOST.md) down through the language layers — apply equally to AI-generated code. If it wouldn't pass your review written by a colleague, it doesn't pass written by a machine.

---

This document is about you using AI to build your projects. For the reverse — supporting the AI tools *your users* bring to your project — see [Standards for Shared Projects § LLM Context](STANDARDS-FOR-SHARED-PROJECTS.md#llm-context).
