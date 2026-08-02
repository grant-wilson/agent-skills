---
name: general-coding-standards
description: Language-agnostic coding standards — self-documenting names, comments that explain why and never narrate the change, small single-responsibility units, fail-fast validation, DRY applied with judgment, SOLID defaults, no dead or speculative code, tooling as the style authority, reviewed and tested changes, and security hygiene. Use when writing, refactoring, or reviewing code in any language, and when deciding how to name, structure, or comment it.
---

**Self-documenting code.** Names of files, types, functions, and variables say
what a thing is or does — precisely enough that most comments become
unnecessary. Comments exist only to explain _why_: a non-obvious constraint,
trade-off, or workaround. A comment that restates the code it precedes is
deleted in review.

**Comments address the reader, not the request.** A comment never narrates the
work that produced it. No "added X as requested", "changed this to fix the bug",
"note: now uses Y instead of Z", no restating what a diff did, and no remark
addressed to whoever asked for the change. The codebase carries no memory of how
it came to be — that belongs in the commit message and the pull request. A
comment that only makes sense to someone who saw the conversation behind it is
deleted in review.

**Small units, single responsibility.** A function does one thing at one level
of abstraction; a class/module has one reason to change. Prefer extracting a
well-named function over adding a section comment. Deep nesting is flattened
with guard clauses and early returns.

**Fail fast and loudly.** Validate inputs at boundaries and throw immediately
with a message naming what was wrong and what was expected. Never silently
swallow an error, return a magic default on failure, or log-and-continue past a
broken invariant.

**DRY, applied with judgment.** Duplicate _knowledge_ is never acceptable — a
business rule lives in exactly one place. Incidental similarity between two
pieces of code is not duplication; wait for the rule of three before
abstracting, and prefer extending an existing tested abstraction over writing a
parallel one.

**SOLID as the default shape.** Depend on abstractions where a surface has more
than one consumer; extend existing behavior rather than modifying tested logic;
keep interfaces small and role-specific rather than one wide interface per
class.

**No dead or speculative code.** Delete unused code, commented-out blocks, and
"we might need it later" flexibility — version control remembers. Every line in
the repo is live, tested, and reachable.

**Tooling is the authority.** Formatting and lint rules are enforced by the
project's configured tools and run in CI; style is never debated in review.
Warnings are errors — a change that introduces a new warning does not merge.

**Every change is reviewed and tested.** No direct commits to the main branch. A
pull request is small enough to review in one sitting, describes _why_ as well
as _what_, includes the tests that prove the change (see the unit-testing
standards), and merges only green.

**Security hygiene by default.** No secrets in source or logs, all input treated
as untrusted at trust boundaries, dependencies pinned and updated deliberately,
and least privilege for every credential, token, and permission the code
requests.
