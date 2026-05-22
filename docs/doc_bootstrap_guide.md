# PHL — How to Start a New Project

This guide explains exactly how to start a new project in the PHL ecosystem using the `manual-SDD` framework.

The goal: every new project starts with the **same anti-spaghetti discipline** LIDR designed, plus the **PHL ecosystem wiring** (contracts in `phlara/apis`).

---

## Quick Path (Recommended)

Open Claude Code in a **fresh empty folder for the new project** and say:

```
Iniciá un nuevo proyecto siguiendo manual-SDD
```

That's all. You don't need to give the project name, the stack, or any other detail in the first message.

Claude will:
1. Clone `github.com/phlara/manual-SDD` to `/tmp` and copy the framework into your folder
2. Ask you the project name (slug)
3. Ask questions about the project's nature, what it consumes/exposes in PHL, deployment target
4. **Recommend a primary stack with strong rationale** (not "here are options, you pick")
5. Mention 1–2 alternatives only as trade-off references
6. Wait for your explicit confirmation before touching any file

The user's instruction in memory: "tengo conocimiento, pero soy más propenso a errores yo. Claude debe sugerir basado en el proyecto." So Claude leads, the user confirms.

Alternatively, the slash command (available once the framework is copied):

```
/bootstrap-phl-project <nombre> [stack-opcional]
```

Without the stack argument, the command goes through the same propose-and-confirm dialogue.

The command handles everything based on where you run it:

| Where you run it | What happens |
|---|---|
| Inside the `manual-SDD` repo | Creates `../<nombre>/` as a sibling folder, copies the framework there, then adapts |
| Inside an empty/new folder | Clones `manual-SDD` framework into current folder, then adapts |
| Inside a project that already has the framework | Adapts in place |

The command will:
- Confirm the destination with you before copying anything
- Ask for stack confirmation if not given
- Adapt the 5 stack-specific files for your chosen tech
- Inject the PHL ecosystem block into `CLAUDE.md`
- Update `docs/doc_architecture.md` with this project's context
- Create the contract file in `phlara/apis` (clone, edit, commit, push)
- Initialize the git repo of the new project

---

## What Gets Copied vs Adapted vs Untouched

When you copy `manual-SDD` into a new project:

| Path | What happens | Why |
|------|-------------|-----|
| `ai-specs/.agents/` | Copied **as-is** | Stack-agnostic — same agents work for every project |
| `ai-specs/.commands/` | Copied **as-is** | Stack-agnostic — same commands work for every project |
| `ai-specs/specs/base-standards.mdc` | Copied + PHL block appended | LIDR universal rules + PHL ecosystem snippet |
| `ai-specs/specs/backend-standards.mdc` | **Adapted** to project stack | LIDR structure preserved; technology examples replaced |
| `ai-specs/specs/frontend-standards.mdc` | **Adapted** to project stack | Same as above |
| `ai-specs/specs/development_guide.md` | **Adapted** to project setup | Replace LTI Docker/Node setup with project setup |
| `ai-specs/specs/api-spec.yml` | **Reset** to skeleton | Each project starts with only `/healthz` |
| `ai-specs/specs/data-model.md` | **Reset** to skeleton | Domain is built incrementally |
| `ai-specs/specs/documentation-standards.mdc` | Copied **as-is** | Documentation rules are universal |
| `ai-specs/changes/` | Empty | This is where plans get written by the SDD workflow |
| `docs/doc_architecture.md` | Copied + "This Project" section prepended | PHL ecosystem context + this project's role |
| `.cursor/`, `.claude/`, `.codex/` | Copied **as-is** | Symlinks to canonical resources |
| `CLAUDE.md`, `AGENTS.md`, `GEMINI.md`, `codex.md` | Copied **as-is** | Symlinks to `base-standards.mdc` |
| `.git/`, `README.md` | **Not copied** | Each project has its own |

---

## Manual Path (Step-by-Step)

If you prefer to do it without the bootstrap command, here is exactly what to do.

### Step 1 — Create the project repo

```bash
gh repo create phlara/<project-slug> --private --clone
cd <project-slug>
```

### Step 2 — Copy the framework

```bash
git clone git@github.com:phlara/manual-SDD.git /tmp/manual-SDD
rsync -av --exclude='.git' --exclude='README.md' /tmp/manual-SDD/ ./
```

### Step 3 — Define the stack in `CLAUDE.md`

`CLAUDE.md` is a symlink to `ai-specs/specs/base-standards.mdc`. Open that file and add a new section "### Project stack" listing:
- Language and version
- Web framework
- ORM / data access
- Frontend framework + UI library
- Test framework

### Step 4 — Adapt the 5 stack-specific specs

For each of these files, **keep every section heading and every rule** from LIDR. Only swap the technology-specific examples:

1. `ai-specs/specs/backend-standards.mdc`
2. `ai-specs/specs/frontend-standards.mdc`
3. `ai-specs/specs/development_guide.md`
4. `ai-specs/specs/api-spec.yml` → reset to skeleton (only `/healthz`)
5. `ai-specs/specs/data-model.md` → reset to skeleton

### Step 5 — Wire the project into `phlara/apis`

```bash
git clone git@github.com:phlara/apis.git /tmp/phlara-apis
cd /tmp/phlara-apis

# Create the contract file based on urgara-newsletter.md structure
cp urgara-newsletter.md <project-slug>.md
# Edit <project-slug>.md — clear all endpoints except /healthz

# Add the row to the ecosystem table in CLAUDE.md
# (edit manually, find the table under "Mapa del ecosistema")

git add <project-slug>.md CLAUDE.md
git commit -m "contrato(<project-slug>): contrato inicial — endpoint /healthz"
git push origin main
```

### Step 6 — Add the PHL ecosystem block to `base-standards.mdc`

Append this block to the end of `ai-specs/specs/base-standards.mdc`:

```markdown
## 4. PHL Ecosystem Integration

This project is part of the PHL ecosystem. API contracts live in `github.com/phlara/apis`.
This project's contract file: `apis/<project-slug>.md`

When adding/modifying/removing any public endpoint, update `apis/<project-slug>.md` in the same work cycle.
When deciding a future cross-project integration, add an item to `apis/PENDIENTES.md`.
```

### Step 7 — Initialize the git repo

```bash
# Add a .gitignore appropriate for the stack
git add .
git commit -m "chore: bootstrap from manual-SDD framework adapted for <stack>"
git push origin main
```

---

## Specific Procedure for Dashboard PHL

Dashboard PHL is a special case: the stack is **already decided** by operational consistency with Dashboard Nivel 1 (PHP + vanilla JS on Hostinger, since both run on `artemis-tech.io`).

For Dashboard PHL specifically you can skip the propose-options step:

```
Iniciá un proyecto PHL llamado "dashboard-phl" con stack PHP + vanilla JS (igual que Nivel 1)
```

Claude will still ask you to confirm before adapting anything, but will not propose alternative stacks. The reasoning is documented and can be referenced when needed.

For **any other project** that does not have a pre-decided stack, do NOT pass a stack — let Claude propose options based on the project's nature.

### 4. Confirm the stack with the bootstrap command

The agent will propose the adaptation. Review and approve.

### 5. After bootstrap is done, start the first ticket

Dashboard PHL needs to consume:
- urgara-newsletter (`/healthz`, `/status`, `/alerts`)
- pds-newsletter (`/healthz`, `/status`, `/tickets`)
- dashboard-nivel1 (`/api/healthz`, `/api/phl-status`)
- boostflow (`/healthz`, `/phl-status`)

The first plan should be the **landing page that shows the status of all four**.

```
/plan-frontend-ticket DASH-001-status-landing
```

The frontend agent will:
1. Read `CLAUDE.md` to identify PHP + vanilla JS stack
2. Read `docs/doc_architecture.md` for ecosystem map
3. Verify the endpoints it plans to consume exist in `phlara/apis/`
4. Save the plan to `ai-specs/changes/DASH-001-status-landing_frontend.md`

After you approve the plan:

```
/develop-frontend DASH-001-status-landing
```

---

## What Must Never Happen

- A new project that does not have its contract file in `phlara/apis` after the first endpoint ships
- A new project that modifies the LIDR agents or commands in `ai-specs/` (those are framework-level — if they need to change, change them in `manual-SDD` and pull into projects)
- A new project that deletes LIDR's anti-spaghetti rules from any spec file
- Implementing a feature without going through `/plan-*-ticket` → review → `/develop-*`

---

## When the Framework Itself Improves

If a project reveals a missing rule or a better way to do something at the framework level:
1. Open a branch in `manual-SDD`
2. Add the improvement (purely additive — do not break existing projects)
3. Document the change
4. Existing projects pull the new rule in the next sync

Never edit framework files directly inside a project repo for cross-project changes. That fragments the ecosystem.
