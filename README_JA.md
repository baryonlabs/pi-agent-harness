<p align="center">
  <img src="harness_banner.png" alt="Harness Banner" width="600">
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Version-2.1.0-brightgreen.svg" alt="Version">
  <a href="LICENSE"><img src="https://img.shields.io/badge/License-Apache_2.0-blue.svg" alt="License"></a>
  <img src="https://img.shields.io/badge/pi-Package-purple.svg" alt="pi Package">
  <img src="https://img.shields.io/badge/Patterns-6_Architectures-orange.svg" alt="6 Architecture Patterns">
  <img src="https://img.shields.io/badge/Modes-single%20%7C%20parallel%20%7C%20chain-green.svg" alt="Delegation Modes">
  <a href="https://github.com/revfactory/harness/stargazers"><img src="https://img.shields.io/github/stars/revfactory/harness?style=social" alt="GitHub Stars"></a>
</p>

<p align="center">
  <a href="#カテゴリー--harness-はどこに位置するか"><img src="https://img.shields.io/badge/Layer-L3%20Meta--Factory-orange" alt="Layer"></a>
  <a href="#カテゴリー--harness-はどこに位置するか"><img src="https://img.shields.io/badge/Sub--layer-Team--Architecture%20Factory-teal" alt="Sub-layer"></a>
  <a href="#"><img src="https://img.shields.io/badge/README-EN%20%7C%20KO%20%7C%20JA-lightgrey" alt="i18n"></a>
</p>

# pi-agent-harness — pi のためのチームアーキテクチャファクトリー

[English](README.md) | [한국어](README_KO.md) | **日本語**

> **pi-agent-harness は [pi](https://github.com/earendil-works/pi)（ミニマルなターミナルコーディングエージェント）向けのチームアーキテクチャファクトリーです。** **「ハーネスを構成して」** (日本語) · **"build a harness for this project"** (English) · **"하네스 구성해줘"** (한국어) と伝えるだけで、ドメイン記述を pi のエージェント定義（`.pi/agents/`）、それらが使うスキル（`.pi/skills/`）、そしてそれらを駆動するオーケストレーションプロンプト（`.pi/prompts/`）へと変換します — あらかじめ定義された 6 種類のチームアーキテクチャパターンから 1 つを選んで。

> **Claude Code からの移植。** 本パッケージはオリジナルの [Harness](https://github.com/revfactory/harness)（Claude Code プラグイン）を **pi 専用に移植** したものです。同じファクトリーを pi のプリミティブ（`.pi/agents/`、`.pi/skills/`、`.pi/prompts/`、同梱の `subagent` 拡張）へマッピングしています。pi は意図的に **ビルトインのサブエージェント・MCP・プランモード・TODO を持たない** ためです。

## 概要

pi は意図的にミニマルです。モデルには 4 つのツール（`read`、`write`、`edit`、`bash`）のみを与え、それ以外はスキル・プロンプトテンプレート・拡張・パッケージで追加します。本ハーネスはその拡張性を活用し、複雑なタスクを専門エージェントへ分解・統制します。「ハーネスを構成して」と伝えるだけで、次を生成します:

- **エージェント定義** (`.pi/agents/*.md`) — `tools`/`model` の frontmatter を持つ専門家ペルソナ。それぞれ独立した `pi` プロセスで実行される
- **スキル** (`.pi/skills/*/SKILL.md`) — Agent Skills 標準の手続き的知識
- **オーケストレーションプロンプト** (`.pi/prompts/*.md`) — `subagent` ツールを `single`/`parallel`/`chain` モードで駆動する `/コマンド`

同梱の **`subagent` 拡張**（`extensions/subagent/`）が pi に存在しない委譲ツールを供給するため、生成されたハーネスは追加設定なしで動作します。

## カテゴリー — Harness はどこに位置するか

Harness は **L3 メタファクトリー** 層 — 他のハーネスを *生成* する層 — に位置します。L3 の中で特定のサブ層、**チームアーキテクチャファクトリー** を選びます。本パッケージは **pi ランタイム** を対象とします。

| 層 | 役割 | 隣接プロジェクト |
|----|------|-----------------|
| **L3 — メタファクトリー / チームアーキテクチャファクトリー**（当プロジェクト） | ドメイン一文 → pi エージェントチーム + スキル + オーケストレーションプロンプト、6 パターンで | — |
| L3 — メタファクトリー / ランタイム構成ファクトリー | 決定論的・再現可能なランタイム構成 | [coleam00/Archon](https://github.com/coleam00/Archon) |
| L3 — メタファクトリー / Claude Code オリジナル | 同一コンセプト、Claude Code ランタイム | [revfactory/harness](https://github.com/revfactory/harness) |
| L2 — クロスハーネスワークフロー | 複数ハーネス間でスキル/ルール/フックを標準化 | [affaan-m/ECC](https://github.com/affaan-m/everything-claude-code) |

## 主な機能

- **エージェント設計** — 6 種のアーキテクチャパターン: パイプライン、ファンアウト/ファンイン、エキスパートプール、生成-レビュー、スーパーバイザー、階層的委譲 — pi の `single`/`parallel`/`chain` 委譲モードへマッピング
- **スキル生成** — 効率的なコンテキスト管理のための Progressive Disclosure を備えた Agent Skills 標準スキルを自動生成
- **オーケストレーション** — `{previous}` チェイニング、`_workspace/` ファイルの受け渡し、エラーハンドリングを備えた `subagent` ツールプロンプト
- **モデルティアリング** — エージェントごとのモデル選択（偵察=`claude-haiku-4-5`、実装=`claude-sonnet-4-5`、高難度推論=`claude-opus-4-7`）
- **検証** — トリガー検証、ドライラン、`subagent` ツールによるスキル有無の比較テスト

## ワークフロー

```
Phase 1: ドメイン分析
    ↓
Phase 2: アーキテクチャ設計 (single / parallel / chain 委譲)
    ↓
Phase 3: エージェント定義生成 (.pi/agents/)
    ↓
Phase 4: スキル生成 (.pi/skills/)
    ↓
Phase 5: オーケストレーションプロンプト (.pi/prompts/) + AGENTS.md ポインター
    ↓
Phase 6: 検証・テスト
    ↓
Phase 7: ハーネス進化 (フィードバック駆動)
```

## インストール

pi パッケージとしてインストールすると、同梱の `subagent` 拡張と `harness` スキルが自動で読み込まれます:

```bash
# git から
pi install npm:@baryonlabs/pi-agent-harness

# git
pi install git:github.com/baryonlabs/pi-agent-harness

# またはローカルチェックアウト
pi install /absolute/path/to/pi-agent-harness

# プロジェクトローカルインストール（.pi/settings.json でチーム共有可能）
pi install -l /absolute/path/to/pi-agent-harness
```

インストールせずに試す:

```bash
pi -e /absolute/path/to/pi-agent-harness
```

そして pi の中でトリガーします:

```
ハーネスを構成して
Build a harness for this project
```

## パッケージ構成

```
pi-agent-harness/
├── package.json                    # pi マニフェスト (extensions + skills)
├── extensions/
│   └── subagent/                   # 同梱の委譲ツール (single/parallel/chain)
│       ├── index.ts
│       ├── agents.ts
│       └── README.md
├── skills/
│   └── harness/
│       ├── SKILL.md                # メインスキル (7 フェーズワークフロー)
│       └── references/
│           ├── agent-design-patterns.md   # 6 パターン + Claude→pi マッピング
│           ├── orchestrator-template.md   # オーケストレーションプロンプトテンプレート
│           ├── team-examples.md           # 5 つの実例構成
│           ├── skill-writing-guide.md     # スキル作成ガイド
│           ├── skill-testing-guide.md     # テスト・評価方法論
│           └── qa-agent-guide.md          # QA エージェント統合ガイド
└── README.md
```

## 使い方

### 委譲モード

pi にはビルトインのエージェントチームやリアルタイムメッセージングがありません。協働は、メインエージェントが同梱の `subagent` ツールで委譲し、`_workspace/` のファイルで結果をやり取りすることで実現されます。オーケストレーションプロンプトは必ず `agentScope: "both"` を渡し、プロジェクトレベルの `.pi/agents/` が発見されるようにします。

| モード | ツールパラメータ | 用途 |
|--------|-----------------|------|
| **single** | `{ agent, task }` | 単発の隔離タスク、エキスパートプール選択 |
| **parallel** | `{ tasks: [...] }` (最大 8、同時 4) | ファンアウト/ファンインの独立作業; メインが統合 |
| **chain** | `{ chain: [...] }` + `{previous}` | 順次パイプライン、生成-レビューループ |

<p align="center">
  <img src="harness_team.png" alt="Harness Agent Team" width="500">
</p>

### アーキテクチャパターン → pi モード

| パターン | pi マッピング |
|---------|--------------|
| パイプライン | `chain` |
| ファンアウト/ファンイン | `parallel` → メインが統合 |
| エキスパートプール | `single`（選択的） |
| 生成-レビュー | `chain`（worker → reviewer → worker） |
| スーパーバイザー | メインエージェントの動的 `parallel` ループ |
| 階層的委譲 | 2 段階委譲 |

## 出力

Harness がプロジェクトに生成するファイル:

```
your-project/
├── .pi/
│   ├── agents/          # エージェント定義 (name, description, tools, model)
│   │   ├── analyst.md
│   │   ├── builder.md
│   │   └── qa-inspector.md
│   ├── skills/          # スキルファイル
│   │   ├── analyze/
│   │   │   └── SKILL.md
│   │   └── build/
│   │       ├── SKILL.md
│   │       └── references/
│   └── prompts/         # オーケストレーションプロンプト (/コマンド)
│       └── build-feature.md
└── AGENTS.md            # ハーネスポインター (トリガールール + 変更履歴)
```

## ユースケース — このプロンプトを試してみてください

インストール後、pi の中でどれでもトリガーしてください:

**ディープリサーチ** — `ディープリサーチ用のハーネスを作って: 並列エージェントが公式・メディア・コミュニティの観点からトピックを調査し、メインエージェントがクロス検証してレポートを作成。`

**Web サイト開発** — `フルスタック Web サイト開発のハーネスを chain で作って: 設計 → フロントエンド(React/Next.js) → バックエンド API → QA テスト。`

**コードレビュー** — `包括的なコードレビューのハーネスを作って: 並列エージェントがアーキテクチャ・セキュリティ・パフォーマンスを点検し、単一レポートに統合。`

**Webtoon 制作** — `Webtoon エピソードのハーネスを作って: artist エージェントがパネルを生成し、reviewer エージェントがスタイルの一貫性を強制する生成-レビューチェーン。`

**マーケティングキャンペーン** — `マーケティングキャンペーンのハーネスを作って: 市場調査、広告コピー作成、ビジュアルコンセプト設計、A/B テスト計画を反復レビュー付きで。`

## 要件

- [pi コーディングエージェント](https://github.com/earendil-works/pi) のインストール（`npm install -g --ignore-scripts @earendil-works/pi-coding-agent`）
- pi で認証されたモデルプロバイダー（`/login` または `ANTHROPIC_API_KEY` などの API キー）
- 同梱の `subagent` 拡張（本パッケージのインストールで自動）— 追加設定は不要

> **セキュリティに関する注意:** `subagent` ツールは `agentScope: "both"`/`"project"` で呼び出された場合のみプロジェクトローカルのエージェント（`.pi/agents/*.md`）を実行し、インタラクティブモードでは確認を求めます。信頼できるリポジトリでのみ有効にしてください。`extensions/subagent/README.md` を参照。

## Harness で作られたもの（Claude Code オリジナル）

オリジナルの Claude Code ハーネスは統制された A/B 実験で検証され、大規模カタログを公開しました。それらの成果物は Claude Code を対象としますが、方法論はこの pi 移植版にもそのまま適用されます。

- **[revfactory/harness-100](https://github.com/revfactory/harness-100)** — 10 ドメインにわたる本番対応エージェントチームハーネス 100 種（EN + KO）。
- **[revfactory/claude-code-harness](https://github.com/revfactory/claude-code-harness)** — A/B 実験（n=15）: 平均品質 +60%（49.5 → 79.3）、勝率 15/15、分散 −32%（著者測定、第三者による再現待ち）。

> 論文全文: *Hwang, M. (2026). Harness: Structured Pre-Configuration for Enhancing LLM Code Agent Output Quality.*

## FAQ

<details>
<summary><b>Q1. オリジナルの Harness と何が違いますか？</b></summary>

**A.** 同じファクトリー、異なるランタイムです。オリジナルの [revfactory/harness](https://github.com/revfactory/harness) は `.claude/agents/` + `.claude/skills/` を生成し、Claude Code のネイティブなエージェントチーム（`TeamCreate`/`SendMessage`/`TaskCreate`）に依存する Claude Code プラグインです。本パッケージは **pi 専用移植** です: `.pi/agents/` + `.pi/skills/` + `.pi/prompts/` を生成し、同梱の `subagent` 拡張（`single`/`parallel`/`chain`）+ `_workspace/` ファイルの受け渡しで協働を実装します。pi にはビルトインのサブエージェントもリアルタイムのチームメッセージングも無いためです。
</details>

<details>
<summary><b>Q2. pi にサブエージェントが無いのに、エージェントチームはどう動くのですか？</b></summary>

**A.** 本パッケージは委譲ごとに独立した `pi` プロセス（隔離されたコンテキストウィンドウ）を起動する `subagent` 拡張を同梱します。メインエージェントがコーディネーターの役割を果たします: `parallel` でファンアウトし、`chain` で依存ステップを繋ぎ（`{previous}` を渡す）、結果を自ら統合します（pi にはチーム合意のチャネルが無いため）。完全な Claude→pi マッピングは `skills/harness/references/agent-design-patterns.md` を参照。
</details>

<details>
<summary><b>Q3. 既存の Claude Code スキルを再利用できますか？</b></summary>

**A.** はい。pi は `settings.json` にディレクトリを追加することで他ハーネスのスキルを読み込めます（例: `"skills": ["~/.claude/skills"]`）。Agent Skills 標準の SKILL.md ファイルは Claude Code と pi の間でおおむね移植可能です。
</details>

## ライセンス

Apache 2.0
