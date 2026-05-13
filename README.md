# research-agent

`research-agent` is a metaprompt kernel for deploying project-local research
agent systems. It is not a distribution of ready-made agent prompts or
project-specific scripts. Instead, each receiving project uses this kernel as
the source of truth for generating local artifacts such as
`prompts/agents-{env}/`, `prompts/skills/`, `docs/00_GLOBAL_RULES.md`,
`docs/03_PROJECT_RULES.md`, and `AGENTS.md`.

The intended layout in a receiving project is:

```text
prompts/meta/          # this repository as a Git submodule
prompts/agents-codex/  # generated locally
prompts/agents-claude/ # generated locally
prompts/skills/        # generated locally
docs/                  # generated or project-maintained local state
```

## Getting The Kernel

The recommended installation method is to add this repository as a Git
submodule at `prompts/meta` in the receiving project.

```sh
mkdir -p prompts
git submodule add https://github.com/t0m0h1r0/research-agent.git prompts/meta
git submodule update --init --recursive
git add .gitmodules prompts/meta
git commit -m "chore: add research-agent metaprompt submodule"
```

When cloning a project that already contains the submodule:

```sh
git clone --recurse-submodules <project-url>
```

For an existing clone:

```sh
git submodule update --init --recursive
```

To update the kernel later:

```sh
git submodule update --remote prompts/meta
git add prompts/meta
git commit -m "chore: update research-agent metaprompt"
```

Pinning the submodule commit is intentional. It makes prompt-system changes
reviewable in the receiving project's own history.

## Deploying Agents

Agent deployment is performed inside the receiving project, following
`kernel-deploy.md` Stage 1 through Stage 4. The upstream boundary is strict:
this repository supplies metaprompt sources only. Generated agent prompts,
skills, templates, scripts, and project docs are local derived outputs.

### Codex

To generate Codex-facing prompts and local support artifacts, invoke
ResearchArchitect / EnvMetaBootstrapper with:

```text
Execute EnvMetaBootstrapper Using prompts/meta/kernel-deploy.md Target Codex
```

Typical generated or updated outputs:

- `prompts/agents-codex/`
- `prompts/skills/`
- `AGENTS.md`
- `docs/00_GLOBAL_RULES.md`
- `docs/03_PROJECT_RULES.md`
- `schema_resolution_report.json`
- `token_telemetry_report.json`

Codex deployment emphasizes executable clarity, patch-oriented work,
worktree-first commits, preservation of local runtime settings such as
`prompts/agents-codex/_base.yaml`, and explicit user approval for no-ff
`main` merges.

### Claude

To generate Claude-facing prompts and local support artifacts, invoke:

```text
Execute EnvMetaBootstrapper Using prompts/meta/kernel-deploy.md Target Claude
```

Typical generated or updated outputs:

- `prompts/agents-claude/`
- `prompts/skills/`
- `AGENTS.md`
- `docs/00_GLOBAL_RULES.md`
- `docs/03_PROJECT_RULES.md`
- `schema_resolution_report.json`
- `token_telemetry_report.json`

Claude deployment emphasizes explicit constraints, role narrative, traceability,
and careful separation between specialist work and independent verification.

### Other LLM Runtimes

For another LLM or execution environment, first define an environment profile in
`kernel-deploy.md` under `ENVIRONMENT PROFILES`, then generate a local
`prompts/agents-{env}/` directory:

```text
Execute EnvMetaBootstrapper Using prompts/meta/kernel-deploy.md Target <env>
```

Keep runtime-specific differences in the environment base prompt and runtime
constraints. Role contracts, HAND schemas, STOP conditions, and the P-E-V-A
workflow should continue to come from the shared kernel files.

## Kernel File Roles

| File | Role |
|---|---|
| `kernel-constitution.md` | The system foundation: phi principles, A-axioms, authority order, isolation levels, source-of-truth rules, and core safety constraints. |
| `kernel-roles.md` | Role contracts for ResearchArchitect, TaskPlanner, domain agents, gatekeepers, and auditors, including HAND schemas and verification mandates. |
| `kernel-ops.md` | Operational procedures: HAND, GIT, LOCK, AUDIT, knowledge operations, STOP codes, and reusable execution protocols. |
| `kernel-domains.md` | Research domain registry, write territories, interface contracts, domain routing, and micro-agent principles. |
| `kernel-workflow.md` | Process model: PLAN -> EXECUTE -> VERIFY -> AUDIT, task classification, dynamic replanning, debate, and recovery flow. |
| `kernel-deploy.md` | EnvMetaBootstrapper specification for generating local docs, agent prompts, skill capsules, templates, scripts, and validation reports. |
| `kernel-antipatterns.md` | Compact anti-pattern catalogue and injection rules for failures such as reviewer hallucination, verification theater, and scope creep. |
| `kernel-project.md` | Swappable project profile: project identity, PR-1..PR-6, paths, validation commands, and project-specific research constraints. |

## Editing `kernel-project.md`

`kernel-project.md` is the project-specific profile. Review and edit it for the
purpose of the receiving project before deploying or redeploying agents.

Common fields to adapt:

- project identity and research focus
- project-specific rules and forbidden shortcuts
- source, paper, experiment, artifact, and wiki paths
- validation commands and remote/local execution policy
- worktree, lock, branch, commit, and merge policy
- the PR-1 through PR-6 rules that should appear in generated project docs

Important rules:

- Do not change generated files such as `docs/00_GLOBAL_RULES.md` or
  `prompts/agents-{env}/` to modify policy. Edit `kernel-project.md` or the
  relevant `kernel-*.md` file, then redeploy.
- Keep project-specific constraints in `kernel-project.md`; do not mix them
  into the universal kernel unless the rule is genuinely reusable across
  projects.
- If different projects need different profiles, use a project-specific branch,
  fork, or pinned submodule revision.
- Treat `kernel-project.md` as part of the receiving project's contract, not as
  a disposable configuration sample.

## Design Philosophy

The kernel is built around the idea that research work should live in durable
artifacts and Git history, not in the private memory of a chat session.

- **Single Source of Truth**: policy changes start in the kernel, then derived
  prompts and docs are regenerated.
- **Project-Local Derivation**: upstream distributes metaprompt sources only;
  each project generates its own agents, skills, docs, templates, and scripts.
- **Broken Symmetry**: important claims are verified by an independent role, not
  by the same specialist that produced them.
- **P-E-V-A**: material work passes through PLAN, EXECUTE, VERIFY, and AUDIT.
- **Worktree-First Operation**: writes happen in isolated worktrees, branches,
  locks, and coherent commits.
- **Traceability**: claims, equations, code, experiments, paper text, and wiki
  knowledge should be linked through external artifacts.
- **Token Discipline**: generated agent prompts should carry compact contracts
  and JIT references, while full operation bodies stay in kernel files or skill
  capsules.

In short: this repository is not the finished agent system. It is the kernel for
building a project-specific research-agent system. Adapt `kernel-project.md`,
run the deployment workflow for the target LLM environment, validate the
generated artifacts, and commit the result in the receiving project.
