Title: Claude Code Project Structure: Every File and Folder Explained
Date: 2026-04-19
Category: GenAI
Tags: claude, claude-code, project-structure, anthropic, ai-tools, llm, developer-tools
Slug: claude-code-project-structure-every-file-explained

# Claude Code Project Structure: Every File and Folder Explained

> *Most Claude Code projects start with a `CLAUDE.md` and nothing else. Here's the full structure that turns Claude from a coding assistant into an engineering partner.*

---

## Table of Contents

- [Why Structure Matters](#why-structure-matters)
- [The Complete Directory](#the-complete-directory)
- [File-by-File Breakdown](#file-by-file-breakdown)
  - [1. CLAUDE.md — The Session Brain](#1-claudemd--the-session-brain)
  - [2. CLAUDE.local.md — Your Personal Overrides](#2-claudelocalmd--your-personal-overrides)
  - [3. .mcp.json — External Tool Connections](#3-mcpjson--external-tool-connections)
  - [4. .claude/settings.json — Permissions & Model Control](#4-claudesettingsjson--permissions--model-control)
  - [5. .claude/rules/ — Contextual Coding Standards](#5-clauderules--contextual-coding-standards)
  - [6. .claude/commands/ — Repeatable Slash Workflows](#6-claudecommands--repeatable-slash-workflows)
  - [7. .claude/skills/ — Context-Aware Capability Packs](#7-claudeskills--context-aware-capability-packs)
  - [8. .claude/agents/ — Specialized Sub-Agents](#8-claudeagents--specialized-sub-agents)
  - [9. .claude/hooks/ — Automated Guardrails](#9-claudehooks--automated-guardrails)
- [Putting It All Together](#putting-it-all-together)
- [Starter Template](#starter-template)
- [Key Principles](#key-principles)

---

## Why Structure Matters

The quality of Claude Code's output is **directly proportional to the quality of your project structure**.

A raw Claude Code session is powerful. But without structure, every session starts from zero — you re-explain conventions, re-define standards, re-specify what "done" looks like. That cognitive overhead compounds over weeks and across teams.

A well-structured project means:
- Claude understands your codebase architecture from session start
- Coding standards are enforced automatically, not re-stated in every prompt
- Workflows run with a single slash command instead of multi-paragraph instructions
- Sub-agents handle specialized tasks without polluting the main context
- Hooks catch unsafe operations before they run

Invest in the structure once. Every session benefits.

---

## The Complete Directory

```
your-project/
├── CLAUDE.md                        # Session context — loaded at start
├── CLAUDE.local.md                  # Personal overrides (gitignored)
├── .mcp.json                        # MCP tool integrations (shared via git)
└── .claude/
    ├── settings.json                # Permissions, model, hooks config
    ├── settings.local.json          # Personal settings overrides (gitignored)
    ├── rules/
    │   ├── code-style.md            # Code formatting & style standards
    │   ├── testing.md               # Testing patterns & requirements
    │   └── api-conventions.md       # API design rules
    ├── commands/
    │   ├── review.md                # /project:review — full code review workflow
    │   └── fix-issue.md             # /project:fix-issue — issue resolution steps
    ├── skills/
    │   └── deploy/
    │       ├── SKILL.md             # Deployment procedures
    │       └── deploy-config.md     # Environment & config details
    ├── agents/
    │   ├── code-reviewer.md         # Dedicated review agent
    │   └── security-auditor.md      # Security-focused analysis agent
    └── hooks/
        └── validate-bash.sh         # Pre-execution bash validation
```

---

## File-by-File Breakdown

### 1. CLAUDE.md — The Session Brain

**Loaded automatically at every session start.**

This is the single most important file in your project. It gives Claude the context it needs to be immediately useful — without you having to re-explain anything.

A well-written `CLAUDE.md` covers:

```markdown
# Project: Acme API

## Overview
A REST API for the Acme SaaS platform. Handles auth, billing, and user management.

## Tech Stack
- Runtime: Node.js 20, TypeScript 5.3
- Framework: Fastify
- Database: PostgreSQL 15 + Prisma ORM
- Auth: JWT + refresh tokens
- Testing: Vitest + Supertest
- CI: GitHub Actions

## Architecture
- src/routes/       — Route handlers (thin, delegate to services)
- src/services/     — Business logic
- src/repositories/ — Database layer (Prisma calls only here)
- src/middleware/   — Auth, validation, error handling

## Key Commands
- `npm run dev`     — Start dev server (port 3000)
- `npm run test`    — Run full test suite
- `npm run lint`    — ESLint + Prettier check
- `npm run migrate` — Run pending DB migrations

## Conventions
- All routes must have Zod input validation
- Services never import from other services directly
- Every new endpoint requires an integration test
- No raw SQL — use Prisma query builder

## Current Focus
Refactoring the billing module to support usage-based pricing.
```

**What goes in CLAUDE.md:**
- Project overview and purpose
- Tech stack with specific versions
- Directory structure and architectural decisions
- Key commands (dev, test, build, deploy)
- Non-obvious conventions the model shouldn't have to guess
- Current work context / active focus area

**What doesn't go in CLAUDE.md:**
- Personal preferences (use `CLAUDE.local.md`)
- Secrets or credentials (never)
- Verbose documentation that belongs in your actual docs

---

### 2. CLAUDE.local.md — Your Personal Overrides

**Gitignored. Never committed. Just for you.**

This file lets individual developers customize Claude's behavior without affecting teammates. It overrides or extends `CLAUDE.md` for a single person's environment.

```markdown
# Local Overrides — Sandipan

## Personal Preferences
- I prefer concise responses without extensive explanation unless asked
- When suggesting refactors, show the diff format, not just the new code
- My local DB runs on port 5433 (not the default 5432)

## Dev Environment
- Using Cursor as my editor — optimize suggestions for Cursor workflows
- Node version: 20.11.0 via nvm

## Current Task
Working on the PaymentWebhookHandler — focus suggestions here.
```

A teammate with different preferences runs the same project with their own `CLAUDE.local.md`. No conflicts, no git noise.

---

### 3. .mcp.json — External Tool Connections

**Shared via git. Controls every external tool your agent can reach.**

MCP (Model Context Protocol) is how Claude Code connects to external services — GitHub, JIRA, Slack, databases, and more. Your `.mcp.json` defines those connections in one place.

```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "${GITHUB_TOKEN}"
      }
    },
    "postgres": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-postgres"],
      "env": {
        "POSTGRES_CONNECTION_STRING": "${DATABASE_URL}"
      }
    },
    "slack": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-slack"],
      "env": {
        "SLACK_BOT_TOKEN": "${SLACK_BOT_TOKEN}"
      }
    }
  }
}
```

With this in place, Claude can:
- Read and create GitHub issues and PRs
- Query your database directly
- Post updates to Slack channels

Commit `.mcp.json` to git. Your whole team gets the same integrations. Store secrets in environment variables, never in the file itself.

---

### 4. .claude/settings.json — Permissions & Model Control

**Controls what Claude is allowed to do. Defaults to safe.**

```json
{
  "model": "claude-opus-4-5",
  "permissions": {
    "allow": [
      "bash:npm run *",
      "bash:git status",
      "bash:git diff *",
      "bash:git log *",
      "read:**",
      "write:src/**",
      "write:tests/**"
    ],
    "deny": [
      "bash:git push *",
      "bash:rm -rf *",
      "bash:curl *",
      "write:.env*"
    ]
  },
  "hooks": {
    "preToolUse": ".claude/hooks/validate-bash.sh"
  }
}
```

**Key things to configure:**
- `model` — specify which Claude model handles this project
- `allow` — explicit list of permitted tool uses
- `deny` — hard blocks (push to git, delete files, write to env)
- `hooks` — which scripts run before/after tool use

`settings.local.json` (gitignored) lets individual devs override their own model preference or permissions without touching the shared config.

---

### 5. .claude/rules/ — Contextual Coding Standards

**Modular. Targeted. Loaded only when relevant.**

Instead of dumping all your standards into `CLAUDE.md`, rules files let you organize conventions by topic and have Claude load them contextually — `code-style.md` when writing code, `testing.md` when generating tests.

**`.claude/rules/code-style.md`**
```markdown
# Code Style Rules

## TypeScript
- Explicit return types on all exported functions
- No `any` types — use `unknown` and narrow
- Prefer `const` over `let`; never use `var`
- Interfaces over type aliases for object shapes

## Naming
- Files: kebab-case (`user-service.ts`)
- Classes: PascalCase (`UserService`)
- Functions/variables: camelCase (`getUserById`)
- Constants: SCREAMING_SNAKE_CASE (`MAX_RETRY_COUNT`)

## Imports
- Group: external libs → internal modules → types
- No barrel imports from index files in the same module
- Absolute paths only (`@/services/user` not `../../services/user`)

## Error Handling
- Always use custom error classes from `src/errors/`
- Never swallow errors silently — log or rethrow
- Async functions must handle rejection
```

**`.claude/rules/testing.md`**
```markdown
# Testing Standards

## Requirements
- Every new route: at minimum one integration test
- Every service function: unit test with mocked dependencies
- Test files colocated with source: `user-service.test.ts`

## Patterns
- Use `describe` blocks matching the function/class name
- Test names: "should [behavior] when [condition]"
- No test should depend on another test's state
- Use factories from `tests/factories/` for test data

## Coverage
- Services: 90% line coverage minimum
- Routes: 80% minimum
- Utils: 100% — they're pure functions
```

**`.claude/rules/api-conventions.md`**
```markdown
# API Conventions

## Endpoints
- RESTful naming: /users, /users/:id, /users/:id/orders
- Versioned: all endpoints under /api/v1/
- Plural resource names always

## Request/Response
- All inputs validated with Zod schemas
- Success: { data: T, meta?: PaginationMeta }
- Error: { error: { code: string, message: string, details?: unknown } }
- HTTP 422 for validation errors, 409 for conflicts, 404 for not found

## Auth
- JWT in Authorization: Bearer header
- Refresh token in httpOnly cookie
- All endpoints authenticated unless marked @public
```

Rules files can also use glob patterns to target specific paths — Claude can be told to only apply `api-conventions.md` when working in `src/routes/`.

---

### 6. .claude/commands/ — Repeatable Slash Workflows

**Type `/project:review`. Claude runs your entire code review process.**

Commands let you encode multi-step workflows as slash commands — callable with a single `/project:name` invocation.

**`.claude/commands/review.md`**
```markdown
# Code Review Workflow

You are performing a thorough code review. Follow these steps in order:

1. **Understand the change**
   - Run `git diff main` to see all changes
   - Identify what problem this change solves

2. **Check correctness**
   - Does the logic handle edge cases?
   - Are error cases handled explicitly?
   - Any off-by-one errors or null pointer risks?

3. **Check standards compliance**
   - Apply rules from .claude/rules/code-style.md
   - Apply rules from .claude/rules/testing.md
   - Are all new endpoints following api-conventions.md?

4. **Check test coverage**
   - Run `npm test -- --coverage` and report gaps
   - Are happy path AND error paths tested?

5. **Security check**
   - Any user input used without sanitization?
   - Any secrets or credentials hardcoded?
   - Auth checks present on all new endpoints?

6. **Output**
   Provide a structured review with sections:
   - ✅ What's good
   - ⚠️ Minor issues (suggestions)
   - ❌ Must fix before merge
```

**`.claude/commands/fix-issue.md`**
```markdown
# Fix GitHub Issue Workflow

Given an issue number, follow these steps:

1. Fetch the issue: use GitHub MCP to get issue #$ARGUMENTS
2. Understand the bug — read related source files
3. Reproduce: identify the code path causing the issue
4. Fix: make the minimal change that resolves the issue
5. Test: write a test that would have caught this bug
6. Commit: `git commit -m "fix: [issue title] (#$ARGUMENTS)"`
7. Summary: explain what was wrong and what you changed
```

Invoke with `/project:fix-issue 247` — Claude fetches issue #247, diagnoses it, fixes it, and tests it.

---

### 7. .claude/skills/ — Context-Aware Capability Packs

**Auto-triggered based on task context. Loads only when needed.**

Skills are task-specific knowledge bundles that activate when Claude detects a relevant context. They keep `CLAUDE.md` lean while making specialized knowledge available on demand.

**`.claude/skills/deploy/SKILL.md`**
```markdown
# Deployment Skill

## When to activate
Load this skill when the user mentions: deploy, deployment, release, production, staging, rollback.

## Deployment Process

### Pre-deploy checklist
- [ ] All tests passing (`npm test`)
- [ ] No TypeScript errors (`npm run type-check`)
- [ ] Migrations reviewed and tested
- [ ] Feature flags configured for gradual rollout

### Deploy to staging
```bash
gh workflow run deploy.yml -f environment=staging -f version=$(git rev-parse HEAD)
```

### Deploy to production
Production deploys require two approvals in GitHub. Never deploy directly.
```bash
gh workflow run deploy.yml -f environment=production -f version=$(git rev-parse HEAD)
```

### Rollback procedure
```bash
# Get previous stable version
git log --oneline -10
# Trigger rollback
gh workflow run rollback.yml -f version=<previous-sha>
```

### Post-deploy verification
- Check error rate in Datadog (should be <0.1%)
- Verify key user flows in staging mirror
- Monitor p95 latency for 10 minutes
```

Claude won't load deployment procedures when you're writing unit tests. It loads them when context signals a deployment task. This keeps the context window efficient.

---

### 8. .claude/agents/ — Specialized Sub-Agents

**Isolated context. Custom tools. Specific roles.**

Agents are specialized Claude instances with their own context windows, system prompts, and tool access. They handle focused tasks without polluting the main conversation.

**`.claude/agents/code-reviewer.md`**
```markdown
---
name: code-reviewer
description: Performs thorough code reviews. Invoke when reviewing PRs or checking code quality.
model: claude-opus-4-5
tools:
  - read
  - bash
---

You are a senior engineer specializing in code review.

Your focus areas:
- Correctness and edge case handling
- Security vulnerabilities (injection, auth bypass, data exposure)
- Performance implications (N+1 queries, unnecessary allocations)
- Maintainability (naming, complexity, coupling)
- Test coverage and test quality

You are direct. You flag real issues clearly. You don't pad reviews with excessive praise.
Format: use ✅ ⚠️ ❌ to categorize findings.
```

**`.claude/agents/security-auditor.md`**
```markdown
---
name: security-auditor
description: Security-focused code analysis. Invoke for security reviews before major releases.
model: claude-opus-4-5
tools:
  - read
  - bash
---

You are a security engineer performing a threat-focused audit.

Check for:
- Injection vulnerabilities (SQL, command, LDAP)
- Authentication and authorization flaws
- Insecure data exposure (logging PII, unencrypted storage)
- Dependency vulnerabilities (`npm audit`)
- Secrets in code or git history
- OWASP Top 10 issues

Report severity: CRITICAL / HIGH / MEDIUM / LOW.
For each issue: describe the vulnerability, show the affected code, explain the attack vector, recommend the fix.
```

Each agent operates in isolation — the security auditor's findings don't bleed into your main coding session. You get focused, expert-mode output from a clean context.

---

### 9. .claude/hooks/ — Automated Guardrails

**Event-driven. Runs before or after Claude takes action.**

Hooks are shell scripts that execute automatically at defined trigger points. They're your last line of defense before Claude does something irreversible.

**`.claude/hooks/validate-bash.sh`**
```bash
#!/bin/bash
# Pre-execution hook — validates bash commands before Claude runs them

COMMAND="$1"

# Block destructive git operations
if echo "$COMMAND" | grep -qE "git push|git force|git reset --hard"; then
  echo "BLOCKED: Direct git push/force operations require manual execution."
  exit 1
fi

# Block production environment access
if echo "$COMMAND" | grep -qE "NODE_ENV=production|--env production"; then
  echo "BLOCKED: Production commands must be run through CI/CD pipeline."
  exit 1
fi

# Block deletion of critical files
if echo "$COMMAND" | grep -qE "rm -rf|rmdir /s"; then
  echo "BLOCKED: Recursive deletion requires manual confirmation."
  exit 1
fi

# Block direct database mutations in production
if echo "$COMMAND" | grep -qE "psql.*production|prisma migrate.*production"; then
  echo "BLOCKED: Production database operations require DBA approval."
  exit 1
fi

# Allow everything else
exit 0
```

**Hook trigger points:**
- `preToolUse` — runs before any tool execution (most common)
- `postToolUse` — runs after tool execution (for logging, formatting)
- `preFileWrite` — runs before writing to a file
- `postFileWrite` — auto-lint or format after Claude writes code

**Practical hook uses:**
- Auto-run Prettier after every file write
- Block writes to `.env` files
- Log all bash commands to an audit trail
- Run ESLint on modified files and report errors back to Claude

---

## Putting It All Together

Here's how a real session plays out with a fully structured project:

```
Developer: "Review the PR for the new payment webhook handler"

Claude:
1. Loads CLAUDE.md → understands it's a Fastify/Prisma project
2. Loads CLAUDE.local.md → knows to show diff format, use port 5433
3. MCP GitHub connection → fetches the PR diff automatically
4. Loads .claude/rules/code-style.md → applies TypeScript standards
5. Loads .claude/rules/api-conventions.md → checks endpoint structure
6. Invokes code-reviewer agent → isolated, focused review context
7. Hook: validate-bash.sh → validates any commands before running
8. Output: structured review with ✅ ⚠️ ❌ findings

Total prompting required from developer: one sentence.
```

Without structure, that same task requires several paragraphs of context, repeated for every session.

---

## Starter Template

Clone and adapt this minimal structure to get started:

```bash
mkdir -p .claude/{rules,commands,skills,agents,hooks}

# Create the essential files
touch CLAUDE.md
touch CLAUDE.local.md
echo "CLAUDE.local.md" >> .gitignore
echo ".claude/settings.local.json" >> .gitignore

touch .mcp.json
touch .claude/settings.json
touch .claude/rules/code-style.md
touch .claude/rules/testing.md
touch .claude/commands/review.md
touch .claude/hooks/validate-bash.sh
chmod +x .claude/hooks/validate-bash.sh
```

Then fill in `CLAUDE.md` with your project context. That's the highest-leverage starting point — everything else builds on it.

---

## Key Principles

**1. CLAUDE.md is infrastructure, not documentation.**
Write it for Claude, not for humans. It should enable immediate, correct action — not explain things a human already knows from the codebase.

**2. Separate shared from personal.**
`.md` → committed. `.local.md` → gitignored. Team standards stay consistent. Individual preferences stay personal.

**3. Keep context lean.**
Rules and skills load contextually for a reason. Don't dump everything into `CLAUDE.md`. A bloated context window dilutes attention on what matters.

**4. Hooks are guardrails, not restrictions.**
They block irreversible operations and enforce automation — they don't limit what Claude can help you think through. Block the action, not the thinking.

**5. Agents for isolation, commands for workflow.**
Complex multi-step workflows → commands. Tasks requiring focused, expert-mode reasoning → agents. Both beat re-explaining the same thing every session.

---

## Resources

- [Claude Code Documentation](https://docs.anthropic.com/en/docs/claude-code)
- [MCP (Model Context Protocol) Spec](https://modelcontextprotocol.io/)
- [Claude Code GitHub](https://github.com/anthropics/claude-code)
- [agentbuild.ai — Community learning Agentic AI](https://agentbuild.ai)

---

*Credit: Project structure diagram by [Sandipan Bhaumik](https://agentbuild.ai) — Data & AI Leader at agentbuild.ai*

*Found this useful? ⭐ Star the repo and share it with your team.*
*Have additions or corrections? Open an issue or submit a PR.*