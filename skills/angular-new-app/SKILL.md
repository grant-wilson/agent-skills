---
name: angular-new-app
description: Creating a new Angular v22 application with the Angular CLI — the `ng new` flow, non-interactive flags, plain-CSS and cascade-layer setup, zoneless bootstrap, analytics-off configuration, and the generators used for every subsequent artifact. Use when starting a new Angular application or workspace from scratch; for work inside an existing Angular project, use the angular-developer skill instead.
---

# Angular New App

Creates a new Angular application. Everything this skill sets up must satisfy the
House Standards in the `angular-developer` skill — read that skill for how to
write the code that follows; this one only covers getting the workspace on disk.

## 1. Confirm the CLI

Check for the Angular CLI: `which ng` on \*nix, `where ng` (or `gcm ng` in
PowerShell) on Windows. If it is missing, ask whether to install it globally with
`npm install -g @angular/cli`, or just use `npx` for the create step.

## 2. Create the application

Suggest a name based on the user's description, or ask for one. Then:

```bash
npx ng new <app-name> --style=css --interactive=false --ai-config=claude
```

Required flags, not preferences:

- `--style=css` — plain CSS is the only style language. Never `scss` or `less`.
- `--interactive=false` — no prompts in CI or local runs.
- `--ai-config=<agent>` — prefer `claude`; use the option matching the user's
  environment (`agents`, `copilot`, `cursor`, `gemini`, `jetbrains`, `windsurf`).
  Read the generated config file so subsequent code matches it.

Other flags to consider from the user's requirements:

- `--routing` — routing setup; add it unless the app is genuinely single-view.
- `--ssr` — server-side rendering.
- `--prefix=<prefix>` — component selector prefix.

**Never pass `--skip-tests`**, even if asked to move fast. Every component and
service ships with tests.

## 3. Configure the workspace

Immediately after `ng new`, before writing any feature code:

1. In `angular.json`, set `"cli": { "analytics": false }` so `ng` never blocks on
   an analytics prompt, and confirm the `schematics` style default is `css`.
2. Confirm `app.config.ts` uses `provideZonelessChangeDetection()` and that no
   `zone.js` polyfill is listed in `angular.json`.
3. Declare the cascade layer order once at the top of `src/styles.css`, before any
   component style loads:

   ```css
   @layer reset, base, components, utilities;
   ```

4. Define the design tokens the app will use as custom properties on `:root` in
   the same file — colors, spacing, radii, typography. Component styles reference
   them with `var(--…)` and never hard-code values.
5. Create `src/testing/` for shared TestBed setup builders, fakes, and data
   builders. Specs import from here rather than defining local helpers.

Do not add Tailwind CSS. Do not add a CSS preprocessor.

## 4. Generate every subsequent artifact with the CLI

Never hand-create these files — the schematics wire up configuration that manual
creation misses.

| Artifact    | Command                                   |
| :---------- | :---------------------------------------- |
| Component   | `npx ng generate component <name>`        |
| Service     | `npx ng generate service <name>`          |
| Pipe        | `npx ng generate pipe <name>`             |
| Directive   | `npx ng generate directive <name>`        |
| Guard       | `npx ng generate guard <name>`            |
| Interceptor | `npx ng generate interceptor <name>`      |
| Resolver    | `npx ng generate resolver <name>`         |
| Interface   | `npx ng generate interface <name>`        |
| Enum        | `npx ng generate enum <name>`             |
| Class       | `npx ng generate class <name>`            |

Note the path each command prints so you know exactly where the new files are,
then augment the generated code to meet the application's needs.

## 5. Build features test-first

Red–green–refactor for each behavior: the failing test, then the implementation,
then the refactor. Before reporting any feature as done, `npx ng test` passes and
`npx ng build` succeeds with zero warnings.

Do not start the dev server on your own — ask the user whether they want it
running. `npx ng build` is enough to check for errors.

---

_The Angular CLI bundles an MCP server with additional guidance, available via
`npx ng mcp` and its `get_best_practices` tool. Where its advice conflicts with
the House Standards in `angular-developer`, the House Standards win._
