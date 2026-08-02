# agent-skills

Coding standards as [Agent Skills](https://skills.sh) — architecture, general
coding, and testing standards that apply everywhere, plus per-language and
per-stack conventions that load only when they are relevant.

## Install

Everything:

```sh
npx skills add grant-wilson/agent-skills
```

Or pick individual skills:

```sh
npx skills add grant-wilson/agent-skills --list
npx skills add grant-wilson/agent-skills --skill typescript-standards --skill unit-testing-standards
```

## Skills

### Universal standards

Apply to every change, in every language and framework.

| Skill                                                             | Covers                                                                              |
| ----------------------------------------------------------------- | ----------------------------------------------------------------------------------- |
| [architecture-standards](skills/architecture-standards/SKILL.md)   | Dependency rule, ports and adapters, composition root, bounded contexts, operability |
| [general-coding-standards](skills/general-coding-standards/SKILL.md) | Naming, small units, fail fast, DRY, SOLID, no dead code, review and security hygiene |
| [unit-testing-standards](skills/unit-testing-standards/SKILL.md)   | TDD, Arrange–Act–Assert, public-surface testing, fakes over mocks, FIRST, coverage    |

### Languages

| Skill                                                             | Applies to             |
| ----------------------------------------------------------------- | ---------------------- |
| [csharp-standards](skills/csharp-standards/SKILL.md)               | `*.cs`                 |
| [typescript-standards](skills/typescript-standards/SKILL.md)       | `*.ts`                 |
| [sql-standards](skills/sql-standards/SKILL.md)                     | `*.sql`                |
| [html-standards](skills/html-standards/SKILL.md)                   | `*.html` and templates |
| [css-standards](skills/css-standards/SKILL.md)                     | `*.css`, `*.scss`      |

### Frameworks and runtimes

| Skill                                                       | Applies to                          |
| ----------------------------------------------------------- | ----------------------------------- |
| [angular-developer](skills/angular-developer/SKILL.md)       | Projects with `angular.json`        |
| [angular-new-app](skills/angular-new-app/SKILL.md)           | Starting a new Angular workspace    |
| [dotnet-developer](skills/dotnet-developer/SKILL.md)         | `global.json`, `*.sln`, `*.csproj`  |
| [deno-developer](skills/deno-developer/SKILL.md)             | Projects with `deno.json(c)`        |

`angular-developer` carries the Angular standards plus reference guides derived
from the Angular team's skill of the same name — see
[NOTICE.md](skills/angular-developer/NOTICE.md).

## Contributing

See [CLAUDE.md](CLAUDE.md) for how skills in this repository are authored.
