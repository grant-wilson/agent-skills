# agent-skills

This repository publishes coding standards as Agent Skills, distributed through
[skills.sh](https://skills.sh). It contains no application code — every change
here is an edit to a skill.

## Layout

- `skills/<skill-name>/SKILL.md` — one skill per directory; the directory name
  matches the `name` in the frontmatter.
- `skills.sh.json` — how the skills are grouped on the repository's skills.sh
  page. Adding a skill means adding it to a grouping here, otherwise it falls to
  the ungrouped section at the bottom.
- `README.md` — install instructions and the skill index.

## Authoring a skill

Frontmatter carries exactly two keys, both required by the skills spec:

```markdown
---
name: kebab-case-identifier
description: What the skill covers, then when to use it.
---
```

The `name` follows the repository's two-tier suffix scheme:

- `<topic>-standards` for the universal skills and the per-language ones —
  `typescript-standards`, `css-standards`, `unit-testing-standards`.
- `<stack>-developer` for a framework or runtime, where the skill also covers
  scaffolding, tooling, and generated code, not just how to write the source —
  `angular-developer`, `dotnet-developer`, `deno-developer`.

A skill that performs a task rather than stating a standard is named for the task
(`angular-new-app`). Renaming a published skill breaks
`npx skills add … --skill <old-name>` for anyone already installing it, so treat
it as a breaking change and note it in the README.

Do not add other frontmatter keys. Fields like `paths` or `globs` are not part
of the spec, are ignored by the CLI and by most agents, and give a false
impression that scoping is enforced — put the scope in the `description`
instead, since that text is the only thing an agent sees when deciding whether
to load the skill. Lead the description with the substance the skill covers and
end with the trigger ("Use when writing or reviewing SQL…", "…or when working in
a project that contains angular.json").

The body is the standard itself, written as directives an agent can follow:

- Prefer short imperative bullets that state the rule and the reason, not
  background prose. Every bullet should be checkable in review.
- Add a fenced code example only where the rule is easier to show than to say;
  keep it minimal and idiomatic.
- Say what is forbidden as plainly as what is required — "`@ts-ignore` is
  forbidden" beats "avoid ignoring type errors where possible".
- Keep a skill focused on one language, runtime, or concern. Anything that
  applies everywhere belongs in `general-coding-standards`,
  `architecture-standards`, or `unit-testing-standards`, not repeated per
  language.
- Standards go stale; state the current stable idiom rather than hedging across
  old versions, and update the skill when the ecosystem moves.

## Checklist for a change

1. `SKILL.md` has `name` matching its directory, and a description that ends
   with when to use it.
2. New skills are listed in `skills.sh.json` and in the `README.md` index.
3. `npx skills add . --list` resolves the repository and shows every skill.
