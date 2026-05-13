# research-agent

`research-agent` は、研究プロジェクトに特化したマルチエージェント運用を
生成するためのメタプロンプト kernel です。このリポジトリは実行済みの
エージェントプロンプトやプロジェクト固有スクリプトを配布する場所ではなく、
各プロジェクトが自分の文脈に合わせて `prompts/agents-{env}/`、
`prompts/skills/`、`docs/00_GLOBAL_RULES.md`、`AGENTS.md` などを生成する
ための source of truth です。

## 入手方法

推奨は、利用先プロジェクトの `prompts/meta` に Git submodule として置く方法です。

```sh
mkdir -p prompts
git submodule add https://github.com/t0m0h1r0/research-agent.git prompts/meta
git submodule update --init --recursive
git add .gitmodules prompts/meta
git commit -m "chore: add research-agent metaprompt submodule"
```

submodule を含むプロジェクトを新しく clone する場合:

```sh
git clone --recurse-submodules <project-url>
```

既に clone 済みの場合:

```sh
git submodule update --init --recursive
```

`research-agent` を更新する場合:

```sh
git submodule update --remote prompts/meta
git add prompts/meta
git commit -m "chore: update research-agent metaprompt"
```

## エージェントデプロイ方法

デプロイは `kernel-deploy.md` の Stage 1--4 に従って、受け入れ先プロジェクト内で
実行します。上流からコピーするのは `kernel-*.md` のメタプロンプトだけです。
生成済み agent prompt、skill、template、script は各プロジェクトの派生成果物です。

### Codex

Codex 用プロンプトを生成する場合は、受け入れ先プロジェクトで次を実行する意図で
ResearchArchitect / EnvMetaBootstrapper に渡します。

```text
Execute EnvMetaBootstrapper Using prompts/meta/kernel-deploy.md Target Codex
```

主な生成先:

- `prompts/agents-codex/`
- `prompts/skills/`
- `AGENTS.md`
- `docs/00_GLOBAL_RULES.md`
- `docs/03_PROJECT_RULES.md`
- `schema_resolution_report.json`
- `token_telemetry_report.json`

Codex 生成では、patch-oriented work、worktree-first commits、明示的な user request
がある場合だけの no-ff main merge、既存の `prompts/agents-codex/_base.yaml`
保持を重視します。

### Claude

Claude 用プロンプトを生成する場合:

```text
Execute EnvMetaBootstrapper Using prompts/meta/kernel-deploy.md Target Claude
```

主な生成先:

- `prompts/agents-claude/`
- `prompts/skills/`
- `AGENTS.md`
- `docs/00_GLOBAL_RULES.md`
- `docs/03_PROJECT_RULES.md`
- `schema_resolution_report.json`
- `token_telemetry_report.json`

Claude 生成では、明示的制約、role narrative、traceability emphasis を重視します。

### その他の LLM / 実行環境

新しい LLM 環境を使う場合は、まず `kernel-deploy.md §ENVIRONMENT PROFILES` に
その環境のプロファイルを定義し、`prompts/agents-{env}/` を project-local output
として生成します。生成規則は同じです。

```text
Execute EnvMetaBootstrapper Using prompts/meta/kernel-deploy.md Target <env>
```

環境ごとの差分は base prompt と runtime constraints に閉じ込め、role contract、
HAND schema、STOP 条件、P-E-V-A workflow は kernel から共有します。

## 各ファイルの役割

| File | Role |
|---|---|
| `kernel-constitution.md` | 研究エージェント系の憲法。φ1--φ7、A1--A11、authority、isolation、source of truth、破れた対称性の検査などを定義します。 |
| `kernel-roles.md` | ResearchArchitect、TaskPlanner、各 domain agent、Gatekeeper などの role contract、HAND schema、CoVe mandate を定義します。 |
| `kernel-ops.md` | HAND、GIT、LOCK、AUDIT、K operations、STOP codes など、実行時の手順とプロトコルを定義します。 |
| `kernel-domains.md` | T/L/E/A/Q/K/P などの research domain、write territory、interface contract、micro-agent 原則を定義します。 |
| `kernel-workflow.md` | PLAN -> EXECUTE -> VERIFY -> AUDIT の P-E-V-A loop、task classification、dynamic replanning、STOP recovery を定義します。 |
| `kernel-deploy.md` | EnvMetaBootstrapper の仕様。kernel から project-local docs、agent prompts、skills、validation reports を生成する手順を定義します。 |
| `kernel-antipatterns.md` | reviewer hallucination、verification theater、scope creep などの anti-pattern catalogue と注入規則を定義します。 |
| `kernel-project.md` | 利用先プロジェクトの identity、PR-1..PR-6、パス規約、実行規約などを定義する差し替え可能な project profile です。 |

## `kernel-project.md` の扱い

`kernel-project.md` は、この kernel をどの研究プロジェクトに適用するかを決める
プロジェクト固有レイヤです。目的に応じて必ず見直してください。

編集すべき代表項目:

- project identity
- domain-specific rules
- source / experiment / paper / artifact paths
- validation commands
- merge and worktree policy
- generated docs に反映したい PR-1..PR-6

重要な原則:

- rule を変えたいときは、生成済み `docs/00_GLOBAL_RULES.md` や
  `prompts/agents-{env}/` を直接直さず、まず `kernel-project.md` または該当
  `kernel-*.md` を直してから再デプロイします。
- プロジェクトごとに異なる `kernel-project.md` が必要な場合は、submodule を
  project-specific branch / fork / pinned revision として管理します。
- `kernel-project.md` は受け入れ先プロジェクトの目的を表すための profile であり、
  universal kernel の本文にプロジェクト固有ルールを混ぜ込む場所ではありません。

## 設計思想

research-agent の中心思想は、研究作業を「会話の記憶」ではなく、追跡可能な
外部 artifact と git history に載せることです。

- **Single Source of Truth**: rule 変更は kernel から始め、生成物は再生成します。
- **Project-Local Derivation**: 上流は metaprompt だけを配り、agent prompt や skill
  は各プロジェクトの制約に合わせて生成します。
- **Broken Symmetry**: 重要な検証は、作成者と独立した verifier が行います。
- **P-E-V-A**: material output は PLAN、EXECUTE、VERIFY、AUDIT を通します。
- **Worktree-First**: 変更は隔離された worktree / branch / lock / coherent commit で
  進めます。
- **Traceability**: claim、equation、code、experiment、paper、wiki の対応を
  artifact と ledger に残します。
- **Token Discipline**: 生成 agent prompt には full operation body を詰め込まず、
  SkillID と JIT reference で必要時に読む構造にします。

この repository は「完成した agent そのもの」ではなく、「プロジェクトごとに正しく
agent system を作るための kernel」です。受け入れ先プロジェクトでは、まず
`kernel-project.md` を目的に合わせ、その後 `kernel-deploy.md` に従って環境別に
デプロイしてください。
