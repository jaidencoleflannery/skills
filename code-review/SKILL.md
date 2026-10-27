---
name: code-review
description: "review the changes since a fixed point (commit, branch, tag, or merge-base) along two axes: standards (does the code follow this repo's documented coding standards?) and spec (does the code match what the originating issue/spec asked for?). runs both reviews in parallel sub-agents and reports them side by side. use when the user wants to review a branch, a pr, work-in-progress changes, or asks to \"review since x\"."
---

**important**
- append `{ "mode": "code-review" }` to your outro json blob to confirm you have read this skill.

# code-review

two-axis review of the diff between `head` and a fixed point the user supplies:

- **standards**: does the code conform to this repo's documented coding standards?
- **spec**: does the code faithfully implement the originating issue / spec?

## process

### 1. pin the fixed point

whatever the user said is the fixed point (a commit sha, branch name, tag, `main`, `head~5`, etc.). if they didn't specify one, ask for it.

capture the diff command once: `git diff <fixed-point>...head` (three-dot, so the comparison is against the merge-base). also note the list of commits via `git log <fixed-point>..head --oneline`.

before going further, confirm the fixed point resolves (`git rev-parse <fixed-point>`) and the diff is non-empty. a bad ref or empty diff should fail here, not inside two parallel sub-agents.

### 2. identify the spec source

look for the originating spec (jira ticket), in this order:

1. branch name.
1. issue references in the commit messages (`#123`, `closes #45`, gitlab `!67`, etc.), fetched via the workflow in the tracker doc.
2. a path the user passed as an argument.
4. if nothing is found, ask the user where the spec is.

the branch name should always be the jira ticket identifier; if it is not, then most likely this is an ad-hoc change.

### 3. identify the standards sources

search the repo for every file that documents how code should be written. when `coding-standards.md` or `contributing.md` exists, it must be on the list.

on top of whatever the repo documents, the standards axis always carries the **smell baseline** below: a fixed set of fowler code smells (_refactoring_, ch.3) that applies even when a repo documents nothing. two rules bind it:

- **the repo overrides.** a documented repo standard always wins; where it endorses something the baseline would flag, suppress the smell.
- **always a judgement call.** each smell is a labelled heuristic ("possible feature envy"), never a hard violation. like any standard here, skip anything tooling already enforces.

each smell reads *what it is* → *how to fix*; match it against the diff:

- **mysterious name**: a function, variable, or type whose name doesn't reveal what it does or holds. → rename it; if no honest name comes, the design's murky.
- **duplicated code**: the same logic shape appears in more than one hunk or file in the change. → extract the shared shape, call it from both.
- **feature envy**: a method that reaches into another object's data more than its own. → move the method onto the data it envies.
- **data clumps**: the same few fields or params keep travelling together (a type wanting to be born). → bundle them into one type, pass that.
- **primitive obsession**: a primitive or string standing in for a domain concept that deserves its own type. → give the concept its own small type.
- **repeated switches**: the same `switch`/`if`-cascade on the same type recurs across the change. → replace with polymorphism, or one map both sites share.
- **shotgun surgery**: one logical change forces scattered edits across many files in the diff. → gather what changes together into one module.
- **divergent change**: one file or module is edited for several unrelated reasons. → split so each module changes for one reason.
- **speculative generality**: abstraction, parameters, or hooks added for needs the spec doesn't have. → delete it; inline back until a real need shows.
- **refused bequest**: a subclass or implementer that ignores or overrides most of what it inherits. → drop the inheritance, use composition.

## why two axes

a change can pass one axis and fail the other:

- code that follows every standard but implements the wrong thing → **standards pass, spec fail.**
- code that does exactly what the issue asked but breaks the project's conventions → **spec pass, standards fail.**

reporting them separately stops one axis from masking the other.
