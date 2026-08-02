---
name: deno-developer
description: Deno standards — JSR and @std dependencies pinned in the deno.json import map, the built-in toolchain (fmt, lint, check, test, task), web-platform and Deno-native APIs over node compat, least-privilege permission flags with no --allow-all, deliberate module surfaces, @std/testing for BDD tests, fake time, and mocks, and how to scope Deno's tooling and VS Code extension in a repository shared with another toolchain such as Angular. Use when writing or reviewing code in a project that contains deno.json or deno.jsonc, and when deciding which formatter or linter owns a directory in a mixed Deno repository.
---

- Prefer Deno's `@std/*` standard library and JSR packages over npm imports;
justify any `npm:` specifier in review. All dependencies are declared in the
`deno.json` import map with pinned versions — no bare URLs scattered through
source files.
- Use Deno's own toolchain end to end: `deno fmt` (sole formatting authority),
`deno lint`, `deno check`, `deno test --coverage`, `deno task` for scripts. No
Node, npm scripts, bundlers, or transpile steps — TypeScript runs directly.
- Reach for Web Platform and Deno-native APIs — `fetch`, `URL`, `crypto.subtle`,
`ReadableStream`, `Deno.readTextFile`, `Deno.Command` — never Node-compat
(`node:*`) equivalents unless a dependency forces it.
- Every script and task declares the narrowest permission set it needs — scoped
flags like `--allow-read=./config --allow-net=api.example.com` — and `deno.json`
tasks encode them so nobody runs with `-A`. `--allow-all` is forbidden
everywhere, including CI.

```json
{
  "tasks": {
    "start": "deno run --allow-net=:8000 --allow-read=./static main.ts",
    "test": "deno test --coverage --allow-read=./tests/fixtures"
  }
}
```
- Each package exposes a deliberate public surface through `mod.ts` (or the
`exports` map in `deno.json`); internal modules are not imported across package
boundaries. Co-locate `<module>.test.ts` next to `<module>.ts`.
- Tests use `@std/testing/bdd` (`describe`/`it`) with `@std/assert`, mock time
with `@std/testing/time`'s `FakeTime`, and fake dependencies with
`@std/testing/mock` (`spy`/`stub`). The sanitizers for resources, ops, and exits
stay enabled — a test that leaks is a failing test.

## Sharing a repository with another toolchain

A repository can hold Deno packages next to a project owned by a different
toolchain — most often an Angular workspace, which is formatted by Prettier and
linted by `ng lint`. Each toolchain owns its own files exclusively. Nothing is
shared, and neither one is ever run over the other's source.

- Scope Deno with the top-level `exclude` in `deno.json`, which applies to `fmt`,
  `lint`, `check`, `test`, and the LSP at once. List every non-Deno project
  there, so that running `deno fmt` from the repository root is safe.

```json
{
  "exclude": ["apps/admin-web/", "apps/storefront/"]
}
```

- Never run `deno fmt`, `deno lint`, or `deno check` against an Angular project,
  and never hand one an explicit path into one — `deno fmt apps/admin-web`
  bypasses `exclude` entirely. `deno fmt` disagrees with Prettier on defaults and
  cannot parse Angular templates, so a single run rewrites the whole project and
  loses the argument with CI.
- Configure the editor to match. In `.vscode/settings.json`, list every non-Deno
  project in `deno.disablePaths` so the Deno language server stands down there
  and the Angular Language Service takes over:

```json
{
  "deno.enable": true,
  "deno.disablePaths": ["apps/admin-web", "apps/storefront"]
}
```

- Where Deno is the smaller part of the repository, invert it: leave
  `deno.enable` off and name only the Deno packages in `deno.enablePaths`.
  Either way the editor's paths and `exclude` in `deno.json` must describe the
  same split — a directory covered by one but not the other yields editor
  diagnostics that CI never reproduces, or the reverse.
- `editor.defaultFormatter` is per-language and has no path scoping, so one
  workspace root cannot send some TypeScript to `denoland.vscode-deno` and the
  rest to Prettier. When both stacks are substantial, use a multi-root
  `.code-workspace` file and set the formatter in each folder's own `settings`
  block.
- Dependencies stay separate too. Angular's packages belong in its `package.json`
  and are installed with npm; they never enter the `deno.json` import map, and
  `deno install` is not run against an Angular project. Sharing code between the
  two goes through a published package or a `deno`-owned module the Angular build
  consumes via npm — not by reaching across directories.
