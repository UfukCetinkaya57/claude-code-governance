[Turkce](README.tr.md)

# Backend Governance Framework for Claude Code

**A multi-agent engineering governance framework that turns Claude Code into a structured Team Lead — not a solo code writer.**

Instead of asking Claude to write everything itself, this framework makes Claude act as a coordinator that delegates work to specialized sub-agents, enforces quality through a tiered pipeline, and escalates risks to the human engineer at the right moment.

---

## What This Is

When you drop this framework into a project and open Claude Code, Claude no longer operates as a generalist assistant. It becomes a **Team Lead** with a defined role: understand the task, select the appropriate pipeline, delegate to the right agent, and verify the output before marking the task complete.

The goal is to bring structure to AI-assisted backend development — reducing the chance of skipped validations, security gaps, or architectural drift that can occur when prompting a general-purpose LLM directly.

---

## Key Concepts

### 5 Specialized Agents

| Agent | Responsibility |
|-------|---------------|
| `backend-developer` | Writes code — endpoints, services, migrations, validations |
| `security-reviewer` | Reviews for auth issues, injection risks, CORS, brute force |
| `quality-gate` | Runs an 11-point quality checklist and evaluates test coverage |
| `architect` | Evaluates architectural decisions, YAGNI violations, and ADRs |
| `devops` | Handles deployment, Docker, monitoring, logging, rollback |

### 3-Tier Pipeline

The pipeline tier is selected automatically based on task characteristics:

| Tier | When Used | Agents Involved |
|------|-----------|-----------------|
| **Light** | Single-file changes, no auth, no DB, no new endpoints | 1 (backend-developer) |
| **Normal** | Standard features — the default path | 2 (backend-developer + quality-gate) |
| **Full** | Auth, payments, migrations, public API changes, security surface | 3-4 (all relevant agents) |

### 4 Engineering Modes

| Mode | When | Strictness |
|------|------|------------|
| `explore` | Experimental, PoC, spike | Low — quality-gate optional |
| `build` | Standard features | Normal — default mode |
| `harden` | Auth, payments, migrations, security | High — full pipeline required |
| `incident` | Live bugs, data loss, emergency rollback | Critical |

If you don't specify a mode, Claude selects it automatically based on signals in your request.

### Feedback Loop

Sub-agents cannot communicate directly. All handoffs go through the Team Lead. If an agent (e.g., `security-reviewer`) finds a problem, the loop restarts with `backend-developer` — up to a maximum of 3 iterations. After 3 loops with unresolved issues, Claude escalates to you with a clear description of the problem and the decision it needs.

### Conflict Priority

When agent outputs conflict, this order is enforced:

**Security > Correctness > Simplicity > Performance**

---

## Directory Structure

```
backend-governance/
├── CLAUDE.md                  # Main governance agreement — Team Lead instructions
├── .claude/agents/            # Agent definitions
│   ├── backend-developer.md
│   ├── security-reviewer.md
│   ├── quality-gate.md
│   ├── architect.md
│   ├── devops.md
│   └── qa-engineer.md
├── api/CLAUDE.md              # API conventions (URL patterns, response format, pagination, rate limiting)
├── backend/CLAUDE.md          # Backend layer rules (controller/service/repository separation, DI)
├── guvenlik/CLAUDE.md         # Security rules
├── kalite/CLAUDE.md           # Quality standards
├── karar/CLAUDE.md            # Architectural Decision Records (ADR) process
├── mimari/CLAUDE.md           # Architecture rules
├── operasyon/CLAUDE.md        # Operations, monitoring, performance targets
├── proje/                     # Project profiles
│   ├── CLAUDE.md              # How project profiles work
│   └── SABLON.md              # Template for creating a new project profile
├── qa/CLAUDE.md               # QA process
├── stack/                     # Stack-specific rules
│   ├── CLAUDE.md              # How stack profiles are used
│   ├── dotnet.md              # .NET conventions and tooling
│   ├── nodejs.md              # Node.js conventions and tooling
│   └── laravel.md             # Laravel conventions and tooling
├── surec/                     # Processes and workflows
│   ├── CLAUDE.md              # Pipeline details, feedback loop diagrams, handoff rules
│   ├── proje-kesfi.md         # Project discovery process
│   └── deployment.md          # Deployment process
├── test/CLAUDE.md             # Testing strategy, mock rules, coverage targets
└── veri/CLAUDE.md             # Data and database rules (naming, migrations, indexes, transactions)
```

---

## Quick Start

**Requirements:** [Claude Code CLI](https://docs.anthropic.com/en/docs/claude-code) and a backend project.

### 1. Clone this repository

```bash
git clone https://github.com/UfukCetinkaya57/claude-code-governance.git
```

### 2. Link to your project via symlink

The governance directory must be a symlink — never copy it. This ensures governance updates propagate to all linked projects.

```bash
# Linux / macOS
ln -s /path/to/backend-governance /path/to/your-project/backend-governance

# Windows (run as Administrator)
mklink /D "C:\path\to\your-project\backend-governance" "C:\path\to\backend-governance"
```

### 3. Copy the main CLAUDE.md to your project root

```bash
cp /path/to/backend-governance/CLAUDE.md /path/to/your-project/CLAUDE.md
```

Claude Code picks up `CLAUDE.md` automatically from the project root.

### 4. Create a project profile

```bash
cp proje/SABLON.md proje/my-project.md
```

Fill in the project profile with your stack, database, architecture, and any domain-specific constraints. Claude will read this at the start of each session.

### 5. Start working

Open Claude Code in your project directory. Claude will read the governance files, detect your stack, and operate as Team Lead from the first message.

---

## How a Task Flows Through the Framework

1. You describe a task to Claude Code
2. Claude reads memory and the project profile to understand the codebase context
3. Claude selects an engineering mode (`build`, `harden`, etc.) and a pipeline tier (Light / Normal / Full)
4. Claude delegates implementation to `backend-developer`
5. If the task touches auth, payments, or security surface — `security-reviewer` runs next
6. `quality-gate` runs its checklist
7. Claude reviews the output as Team Lead (business logic, edge cases, readability)
8. Claude presents a **Project Status Report** — current phase, next step, any blockers

You don't need to manage the pipeline manually. Claude reports which tier it selected and why.

---

## Customization

- **Add a new stack:** Create `stack/yourstack.md` following the pattern in existing stack files
- **Modify agent behavior:** Edit agent definitions in `.claude/agents/`
- **Add domain rules:** Create a project profile in `proje/` with domain-specific constraints
- **Adjust pipeline triggers:** Edit tier selection criteria in `surec/CLAUDE.md`
- **Extend quality checks:** Modify `kalite/CLAUDE.md` and the `quality-gate` agent definition

---

## Multi-Project Setup

If you maintain multiple backend services, each project gets its own profile file in `proje/`. All projects share the same governance symlink — governance updates apply everywhere at once.

See `memory/` (gitignored) for session-level memory that persists across Claude Code conversations within a project.

---

## Notes

**Language:** Internal governance files are written in Turkish. Claude Code understands any language, so this has no effect on functionality. Fork and translate if you prefer your own language.

**Symlinks are mandatory.** Copying the governance directory breaks the update propagation model. If you copy instead of symlinking, governance changes in one project won't reach others.

**The `memory/` directory and `.claude/settings.local.json` are gitignored.** They store session-specific data and should not be committed.

**Team Lead does not write application code.** The `CLAUDE.md` governance agreement explicitly prohibits Team Lead from using Edit/Write tools on application code. Code writing is always delegated to `backend-developer`. This is a design constraint, not a limitation.

---

## License

MIT
