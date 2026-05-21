# PHL — SDD Framework

This repository is the **Spec-Driven Development (SDD) framework for the PHL ecosystem**.

It provides a skills-first, reusable structure for running a complete SDD workflow with AI: from product requirements and planning to coding, verification, and code review. Conventions stay consistent across Codex, Cursor, and Claude.

**Operator:** phlara (phlara.engine@gmail.com)

---

## PHL Ecosystem Overview

PHL is a set of interconnected projects managed by phlara. All projects communicate via REST API authenticated with `X-API-Key`.

```
urgara-newsletter  ──┐
                     ├──► Dashboard Nivel 1 (Cristian)
pds-newsletter     ──┘              │
                                    │ estado / healthcheck
urgara-newsletter  ──┐              │
pds-newsletter     ──┼──────────────┴──► Dashboard PHL (phlara)
Dashboard Nivel 1  ──┘
```

| Project | Stack | VPS | Status |
|---------|-------|-----|--------|
| urgara-newsletter | Python, Docker, Nginx, Brevo | DigitalOcean `urgara.sytes.net` | Production |
| pds-newsletter | Python, Docker, Nginx, Brevo | DigitalOcean | Production |
| Dashboard Nivel 1 | React | TBD | Production |
| Dashboard PHL | React | TBD | In construction |

API contracts live in the companion repo: **`phlara/apis`**

---

## Repository Structure

```text
.
├── ai-specs/
│   ├── .agents/                 # Canonical agent role definitions
│   ├── .commands/               # Shared utility commands
│   └── skills/                  # Canonical skill definitions (main workflow entrypoint)
│
├── .codex/
│   ├── agents -> ../ai-specs/.agents
│   ├── commands -> ../ai-specs/.commands
│   └── skills -> ../ai-specs/skills
│
├── .cursor/
│   ├── agents -> ../ai-specs/.agents
│   ├── commands -> ../ai-specs/.commands
│   ├── skills -> ../ai-specs/skills
│   └── rules/
│
├── .claude/
│   ├── agents -> ../ai-specs/.agents
│   ├── commands -> ../ai-specs/.commands
│   └── skills -> ../ai-specs/skills
│
├── docs/                        # PHL technical context and reference docs
└── README.md
```

## Multi-Copilot Strategy

Single canonical source in `ai-specs/`, exposed to each copilot folder via symlinks.

- `.codex/*` links to canonical resources
- `.cursor/*` links to canonical resources
- `.claude/*` links to canonical resources

**Why:** one update propagates to all tools. No maintenance drift.

---

## Skills-First Workflow

Skills are the default entrypoint for recurring tasks.

Current skills:
- `ai-specs/skills/enrich-user-story/SKILL.md` — close a requirement before building
- `ai-specs/skills/write-pr-report/SKILL.md` — generate a clean PR description

---

## Mandatory SDD Workflow (Non-Negotiable)

Every feature or change must follow this sequence — no exceptions, even if the user explicitly asks to skip:

```
1. enrich-user-story  →  close all requirements (no open decisions)
2. High-Level Technical Contract  →  user approves in writing
3. tasks_for_AI/ task brief  →  generated only after contract approval
4. Implementation
```

**Hard-stop rule:** if the user asks to implement directly without a contract, the assistant must stop, explain the process, and ask for deliberate confirmation before proceeding. A casual "let's start building" is not a contract bypass.

---

## How to Bootstrap a PHL Project

1. Copy this structure into the project repo.
2. Keep `ai-specs/` as canonical — update agents and skills there.
3. Create symlinks from `.codex/`, `.cursor/`, `.claude/` to `ai-specs/`.
4. Replace `docs/` with project-specific architecture, planning, and verification references.
5. Build new reusable workflows as skills under `ai-specs/skills/`.
6. Add a `CLAUDE.md` with a hard-stop reference to this workflow — see example below.

---

## Customization Guidelines

- Update agent definitions in `ai-specs/.agents/`.
- Add or refine skills in `ai-specs/skills/`.
- Keep commands minimal — utility behavior only.
- Keep symlinks relative so the repo stays portable.
- Document project-specific technical context in `docs/`.

---

## Minimum CLAUDE.md Template for PHL Projects

Every project using this framework must include a `CLAUDE.md` at the repo root with at minimum:

```markdown
# <Project Name> — Claude Instructions

## Non-negotiable: SDD workflow

Before any implementation:
1. Read `docs/project_context.md`
2. Run `enrich-user-story` to close all open decisions
3. Generate and get approval on the High-Level Technical Contract
4. Only then: create task brief and implement

**The contract step cannot be skipped even if the user asks to implement directly.**
If the user says "start building" without a contract, stop and explain the process.

## Source of truth

GitHub is the source of truth. On every session start, run:
  git pull origin main

## Context

See `docs/project_context.md` for project and client context.
```
