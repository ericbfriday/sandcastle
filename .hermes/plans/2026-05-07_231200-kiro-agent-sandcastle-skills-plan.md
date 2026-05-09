# Plan: Build Out Kiro CLI Agent Skills, Commands, and Support for Sandcastle

## Goal

Create a comprehensive `.kiro/` directory structure with custom agents, skills, and configuration that guides developers to effectively use Sandcastle — covering initialization, workflow selection, prompt engineering, debugging, and advanced multi-agent orchestration patterns.

## Current Context / Assumptions

- **Sandcastle** is a TypeScript library by Matt Pocock that orchestrates AI coding agents (Claude Code, Kiro, Codex, Pi, OpenCode) inside isolated sandboxes (Docker, Podman, Vercel).
- **Kiro CLI agent provider** has already been added to Sandcastle (commits on this branch: `feat: add Kiro CLI agent provider`, resume support, etc.).
- Kiro CLI supports **custom agents** (`.kiro/agents/`) and **skills** (`.kiro/skills/`) with a `SKILL.md` frontmatter+markdown format.
- Kiro headless mode uses `kiro-cli chat --no-interactive --trust-all-tools` with `KIRO_API_KEY` env var.
- Sandcastle's core patterns are: simple-loop, sequential-reviewer, parallel-planner, and parallel-planner-with-review.
- The project uses Effect.ts, vitest for tests, tsgo for builds, and changesets for versioning.
- Key concepts: sandbox providers, branch strategies, prompt templates with `{{KEY}}` args and `!`command`` shell expressions, hooks, worktrees, iterations, completion signals.

## Proposed Approach

Build a layered `.kiro/` directory providing:

1. **Custom Agents** — purpose-built agent personas for different Sandcastle tasks
2. **Skills** — portable instruction packages covering Sandcastle workflows, best practices, and troubleshooting
3. **Steering files** — project-level context that enriches all agent interactions

## Directory Structure

```
.kiro/
├── agents/
│   ├── sandcastle-contributor.json      # For contributing code to Sandcastle itself
│   ├── sandcastle-user-guide.json       # For users setting up/using Sandcastle in their projects
│   ├── prompt-engineer.json             # For writing effective prompts/templates
│   └── orchestration-architect.json     # For designing multi-agent workflows
├── skills/
│   ├── sandcastle-quickstart/
│   │   └── SKILL.md                     # Init, configure, first run
│   ├── prompt-engineering/
│   │   ├── SKILL.md                     # Writing effective agent prompts
│   │   └── references/
│   │       ├── prompt-patterns.md       # Common prompt patterns
│   │       └── anti-patterns.md         # Things to avoid
│   ├── branch-strategy-guide/
│   │   └── SKILL.md                     # Choosing and configuring branch strategies
│   ├── workflow-patterns/
│   │   ├── SKILL.md                     # Overview of orchestration patterns
│   │   └── references/
│   │       ├── simple-loop.md           # Single-agent issue loop
│   │       ├── sequential-review.md     # Implement-then-review
│   │       ├── parallel-execution.md    # Plan-execute-merge
│   │       └── custom-orchestration.md  # createSandbox, createWorktree
│   ├── sandbox-configuration/
│   │   ├── SKILL.md                     # Provider selection, Dockerfile, hooks
│   │   └── references/
│   │       ├── docker-setup.md
│   │       ├── podman-setup.md
│   │       └── custom-providers.md
│   ├── debugging-sandcastle/
│   │   └── SKILL.md                     # Troubleshooting common issues
│   ├── ci-cd-integration/
│   │   ├── SKILL.md                     # Running Sandcastle in CI/CD
│   │   └── references/
│   │       └── github-actions.md
│   ├── contributing-to-sandcastle/
│   │   ├── SKILL.md                     # Development workflow for the repo
│   │   └── references/
│   │       ├── architecture.md
│   │       └── testing-guide.md
│   └── kiro-with-sandcastle/
│       └── SKILL.md                     # Using Kiro CLI as the agent in Sandcastle
└── steering/
    └── project-context.md               # Always-loaded context about this repo
```

## Step-by-Step Plan

### Phase 1: Foundation — Steering & Project Context

**File: `.kiro/steering/project-context.md`**

Content summary:

- Sandcastle terminology (from CONTEXT.md — abbreviated)
- Key files and their purposes
- Build/test/lint commands
- Contribution conventions (changesets, CLAUDE.md rules)

### Phase 2: Skills — Core User Workflows

#### 2.1 `sandcastle-quickstart` skill

- Prerequisites (Node.js, Git, Docker/Podman)
- `npm install --save-dev @ai-hero/sandcastle`
- `npx sandcastle init` walkthrough
- Choosing agent (kiro, claude-code, etc.)
- Choosing template
- Configuring `.sandcastle/.env`
- First run with `npx tsx .sandcastle/main.mts`
- Common first-run errors and fixes

#### 2.2 `prompt-engineering` skill

- Prompt anatomy: Context → Task → Execution → Verification → Done
- Shell expressions (`!`command``) for dynamic context
- Prompt arguments (`{{KEY}}`) for reusability
- Built-in args (`{{SOURCE_BRANCH}}`, `{{TARGET_BRANCH}}`)
- Completion signals (`<promise>COMPLETE</promise>`)
- Reference: patterns (issue-driven, exploration, review, migration)
- Reference: anti-patterns (too vague, too long, no verification, no exit condition)

#### 2.3 `branch-strategy-guide` skill

- Head (default for bind-mount): fast iteration, no safety net
- Merge-to-head: safe automation, throwaway branches
- Branch: explicit named branches for PRs
- Decision matrix based on use case (dev, CI, parallel agents, review pipelines)
- When to use `createWorktree()` vs `run()` vs `createSandbox()`

#### 2.4 `workflow-patterns` skill

- Overview/decision tree: which pattern for which situation
- Reference files for each pattern with code examples:
  - Simple loop: one agent, sequential issues
  - Sequential review: implement + review gate
  - Parallel planner: plan → N parallel agents → merge
  - Custom: reusable sandboxes, worktree-first, mixed agents

#### 2.5 `sandbox-configuration` skill

- Choosing a provider (Docker vs Podman vs Vercel vs custom)
- Dockerfile customization (adding deps, language runtimes, tools)
- Hooks: `host.onWorktreeReady`, `host.onSandboxReady`, `sandbox.onSandboxReady`
- Mounts, `copyToWorktree`, env vars
- Reference: Docker-specific setup
- Reference: Podman-specific (SELinux, rootless)
- Reference: writing custom providers

#### 2.6 `debugging-sandcastle` skill

- Run logs (`.sandcastle/logs/`)
- Common errors: missing env vars, Docker not running, image not built
- Agent hangs/timeouts (idle timeout, stuck on permission prompts)
- Branch conflicts and worktree issues
- Session capture and resume debugging

#### 2.7 `ci-cd-integration` skill

- GitHub Actions workflow for Sandcastle
- Secret management (`ANTHROPIC_API_KEY`, `KIRO_API_KEY`)
- Running AFK agents on PR events
- Auto-closing issues, auto-merging branches
- Reference: complete GitHub Actions YAML template

#### 2.8 `contributing-to-sandcastle` skill

- Dev setup: `npm install`, `npm run build`, `npm test`
- TypeScript with Effect.ts patterns
- Test patterns with vitest
- ADR conventions
- Changeset workflow
- Branch naming conventions (from existing branch names)
- PR workflow
- Reference: architecture overview (Effect services, providers, orchestrator)

#### 2.9 `kiro-with-sandcastle` skill

- Setting up Kiro as the agent provider
- `KIRO_API_KEY` configuration
- Headless mode specifics
- Session resume (server-side, no host file needed)
- Kiro custom agents + Sandcastle = nested agent composition
- When to choose Kiro over Claude Code

### Phase 3: Custom Agents

#### 3.1 `sandcastle-contributor.json`

- Prompt: expert in TypeScript, Effect.ts, Sandcastle internals
- Tools: read, write, shell, grep, glob
- Resources: project context, contributing skill, CONTEXT.md, ADRs
- Focus: writing code, tests, fixes for Sandcastle itself

#### 3.2 `sandcastle-user-guide.json`

- Prompt: helpful guide for setting up and using Sandcastle in any project
- Tools: read, grep, glob (no write — advisory only)
- Resources: quickstart skill, workflow patterns, prompt engineering, sandbox config
- Focus: answering "how do I..." questions

#### 3.3 `prompt-engineer.json`

- Prompt: specialist in writing effective agent prompts for Sandcastle
- Tools: read, write, grep
- Resources: prompt-engineering skill, templates as reference
- Focus: writing and reviewing `.sandcastle/prompt.md` files

#### 3.4 `orchestration-architect.json`

- Prompt: expert in designing multi-agent orchestration with Sandcastle
- Tools: read, write, shell
- Resources: workflow patterns, branch strategy, sandbox config
- Focus: designing `main.mts` orchestration scripts

### Phase 4: README / Index

Add a `.kiro/README.md` documenting what's available and how to use it.

## Files Likely to Change

New files only (no modifications to existing Sandcastle source):

- `.kiro/agents/*.json` (4 files)
- `.kiro/skills/*/SKILL.md` (9 skills)
- `.kiro/skills/*/references/*.md` (~12 reference files)
- `.kiro/steering/project-context.md`
- `.kiro/README.md`

## Tests / Validation

- Verify all `SKILL.md` files have valid YAML frontmatter (`name` + `description`)
- Verify agent JSON files are valid and parseable
- Test with `kiro-cli --agent sandcastle-user-guide` to confirm agent loads
- Test skill activation with `/context show` in Kiro CLI
- Verify `skill://` URIs in agent configs resolve correctly

## Risks, Tradeoffs, and Open Questions

### Risks

- **Staleness**: Skills that reference specific APIs or options may drift as Sandcastle evolves rapidly. Mitigation: keep skills focused on stable concepts, reference README for latest options.
- **Token overhead**: Too many skills or too-large reference files may consume context. Mitigation: keep SKILL.md concise, put details in references/ that are loaded on demand.

### Tradeoffs

- **Scope**: We could make skills generic (applicable to any agent tool) or Sandcastle-specific. Decision: Sandcastle-specific since that's the goal.
- **Agent count**: More agents = more choice paralysis. 4 focused agents seems right for now.
- **Workspace vs Global**: Skills in `.kiro/skills/` are workspace-scoped. Users wanting them globally can symlink to `~/.kiro/skills/`.

### Open Questions

1. Should we include a `.kiro/hooks/` directory for Kiro-specific hooks (e.g., auto-format on save)?
2. Should any skills reference MCP servers that could provide Sandcastle-specific tooling?
3. Should we commit the `.kiro/` directory to this fork, or create it as a separate installable package?
4. Do we want a `sandcastle-reviewer` agent specifically for PR review workflows?
