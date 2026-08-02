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
| [angular-conventions](skills/angular-conventions/SKILL.md)   | Projects with `angular.json`        |
| [dotnet-conventions](skills/dotnet-conventions/SKILL.md)     | `global.json`, `*.sln`, `*.csproj`  |
| [deno-conventions](skills/deno-conventions/SKILL.md)         | Projects with `deno.json(c)`        |

## Contributing

See [CLAUDE.md](CLAUDE.md) for how skills in this repository are authored.
