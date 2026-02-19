# Skills — Reference

Skills extend Claude with custom instructions. Each skill is a directory containing `SKILL.md`.

## Where skills live

| Location   | Path                                             | Applies to        |
| :--------- | :----------------------------------------------- | :---------------- |
| Personal   | `~/.claude/skills/<skill-name>/SKILL.md`         | All your projects |
| Project    | `.claude/skills/<skill-name>/SKILL.md`           | This project only |

When skills share the same name, higher-priority locations win: enterprise > personal > project.

## Directory structure

```
my-skill/
├── SKILL.md           # Main instructions (required)
├── reference.md       # Detailed docs — loaded when needed
├── examples/
│   └── sample.md      # Example output
└── scripts/
    └── helper.sh      # Script Claude can execute
```

Reference supporting files from `SKILL.md` so Claude knows when to load them. Keep `SKILL.md` under 500 lines.

## Frontmatter reference

```yaml
---
name: my-skill
description: What this skill does
disable-model-invocation: true
allowed-tools: Read, Grep
---
```

| Field                      | Description                                                                                                      |
| :------------------------- | :--------------------------------------------------------------------------------------------------------------- |
| `name`                     | Slash-command name. Lowercase, numbers, hyphens (max 64 chars). Defaults to directory name.                      |
| `description`              | When to use this skill. Claude uses this for automatic loading. Defaults to first paragraph.                     |
| `argument-hint`            | Autocomplete hint, e.g. `[issue-number]`.                                                                        |
| `disable-model-invocation` | `true` → only you can invoke. Removes from Claude's context. Use for side-effectful workflows.                   |
| `user-invocable`           | `false` → hidden from `/` menu; only Claude can invoke. Use for background knowledge.                            |
| `allowed-tools`            | Tools Claude can use without per-use approval when this skill is active.                                         |
| `model`                    | Model to use when this skill is active.                                                                          |
| `context`                  | `fork` → run in an isolated subagent context.                                                                    |
| `agent`                    | Subagent type when `context: fork` is set (`Explore`, `Plan`, `general-purpose`, or custom). Default: `general-purpose`. |
| `hooks`                    | Hooks scoped to this skill's lifecycle.                                                                          |

## String substitutions

| Variable               | Description                                                                 |
| :--------------------- | :-------------------------------------------------------------------------- |
| `$ARGUMENTS`           | All arguments passed when invoking. Appended automatically if not present.  |
| `$ARGUMENTS[N]`        | Specific argument by 0-based index.                                         |
| `$N`                   | Shorthand for `$ARGUMENTS[N]` (e.g. `$0`, `$1`).                           |
| `${CLAUDE_SESSION_ID}` | Current session ID.                                                         |

## Types of skill content

**Reference content** — knowledge Claude applies inline (conventions, patterns, style guides):
```yaml
---
name: api-conventions
description: API design patterns for this codebase
---
Use RESTful naming. Return consistent error formats. Include request validation.
```

**Task content** — step-by-step instructions for an action. Add `disable-model-invocation: true` for workflows you want to trigger manually:
```yaml
---
name: deploy
description: Deploy the application to production
disable-model-invocation: true
---
1. Run tests
2. Build
3. Push to deployment target
```

## Invocation control

| Frontmatter                      | You can invoke | Claude can invoke | Loaded into context                                          |
| :------------------------------- | :------------- | :---------------- | :----------------------------------------------------------- |
| (default)                        | Yes            | Yes               | Description always in context; full skill loads when invoked |
| `disable-model-invocation: true` | Yes            | No                | Not in context; full skill loads when you invoke             |
| `user-invocable: false`          | No             | Yes               | Description always in context; full skill loads when invoked |

## Subagent execution (`context: fork`)

Add `context: fork` to run the skill in an isolated context. The skill content becomes the subagent's prompt — it won't have access to conversation history.

Only makes sense for skills with explicit task instructions (not pure reference/guidelines).

```yaml
---
name: deep-research
description: Research a topic thoroughly
context: fork
agent: Explore
---
Research $ARGUMENTS:
1. Find relevant files with Glob and Grep
2. Read and analyze the code
3. Summarize findings with file references
```

## Dynamic context injection

`!`command`` runs a shell command before the skill content is sent to Claude. The output replaces the placeholder:

```markdown
Current branch: !`git branch --show-current`
PR diff: !`gh pr diff`
```

This is preprocessing — Claude only sees the final rendered output.
