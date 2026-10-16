# backend-dev-kit

A Claude Code plugin for backend development. It takes a task end to end, from analysis to code review.

## Contents

### Skills
| Skill | What it does |
|---|---|
| `task-analysis` | Turns a raw task into something ready to code and produces a **roadmap** in a fixed template, without asking for more input |
| `task-implementation` | Implements the roadmap, reports problems and gaps found while coding, and runs a **completeness check** when done |
| `task-review` | Checks and tests the implemented task, does a **code review**, and gives prioritized feedback |

### Agents
| Agent | What it does |
|---|---|
| `researcher` | Sourced research and comparisons on libraries, APIs and architecture (read-only) |
| `security-auditor` | Security scan based on the OWASP Top 10 and API security (read-only) |
| `test-engineer` | Runs the tests, analyzes coverage, and writes missing tests |
| `backend-coder` | Implements roadmap steps in the style of the existing codebase and tests them |

## Workflow

```
task text ───► task-analysis ──────► roadmap
                   (researcher)
roadmap ─────► task-implementation ─► code + implementation report
                   (backend-coder)
code ────────► task-review ─────────► review report + feedback
                   (test-engineer + security-auditor, in parallel)
```

## Installation

### Option 1: Load from a directory (for trying it out)
```bash
claude --plugin-dir /path/to/backend-dev-kit
```

### Option 2: Install from GitHub as a marketplace (persistent)
Inside Claude Code:
```
/plugin marketplace add Yusufcandir/backend-dev-kit
/plugin install backend-dev-kit@backend-dev-kit
```
A local path also works in place of `Yusufcandir/backend-dev-kit`. After installing, restart Claude Code or run `/reload-plugins`.

### Update to the latest version
```
/plugin marketplace update backend-dev-kit
```
Then restart Claude Code or run `/reload-plugins`.

### Validate
```bash
claude plugin validate /path/to/backend-dev-kit
```

## Usage examples

```
/backend-dev-kit:task-analysis  Users should be able to reset their password by email. The link must expire after 30 minutes.
/backend-dev-kit:task-implementation  docs/roadmap/password-reset.md
/backend-dev-kit:task-review
```

Skills also trigger from plain requests such as "analyze this task and give me a roadmap", "implement the roadmap" or "review the code".
To call an agent directly, ask for it by name, for example "scan this module with the security-auditor agent".

## Using the analysis skill with another AI

`skills/task-analysis/SKILL.md` is self-contained. Give its content (without the frontmatter) to another AI as its
system instructions, then send the task text, and it produces the roadmap directly without asking follow-up questions.
Unknowns go into the "Assumptions" and "Open Questions" sections.

## Directory layout

```
backend-dev-kit/
├── .claude-plugin/
│   ├── plugin.json
│   └── marketplace.json
├── skills/
│   ├── task-analysis/SKILL.md
│   ├── task-implementation/SKILL.md
│   └── task-review/SKILL.md
├── agents/
│   ├── researcher.md
│   ├── security-auditor.md
│   ├── test-engineer.md
│   └── backend-coder.md
└── README.md
```

## License

MIT
