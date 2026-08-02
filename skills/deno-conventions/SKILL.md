---
description: deno-conventions conventions for this project. Applies to projects that contain deno.json or deno.jsonc.
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
