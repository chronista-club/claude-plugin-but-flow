---
name: gitbutler-workflow
description: >
  GitButler CLI（butコマンド）を使ったブランチ戦略の設計と運用。Claude Codeとの連携ワークフロー、
  Virtual Branches（並行開発）、スタック管理（依存ブランチ）、レビュー・PR連携をカバー。
  ソロ開発者がClaude Codeの複数セッションを並列で走らせながら、整理されたブランチ構造を
  自動的に維持するためのスキル。

  このスキルは以下のような場面で使う：
  - 「GitButlerでブランチ戦略を組みたい」「butコマンドの使い方」
  - 「Claude Codeで並列開発したい」「複数セッションのブランチ管理」
  - 「stacked PRの運用方法」「PRレビューのワークフロー」
  - 「but absorb / but rub の使い方」
  - プロジェクトのブランチ構成の設計・提案
  - git-flowやtrunk-basedからGitButler Flowへの移行相談
---

# GitButler Workflow — Claude Code連携ブランチ戦略

GitButler CLI（`but`コマンド）とClaude Codeを組み合わせた、ソロ開発者向けのブランチ戦略スキル。

## 前提知識

GitButlerは従来のGit操作を置き換える新しいSCM。核となる概念は3つ：

1. **Virtual Branches** — 同一ワーキングディレクトリで複数ブランチを同時に「apply」できる。checkoutもstashも不要
2. **Stacked Branches** — ブランチ間の依存関係を明示し、底のコミット修正時に上が自動rebase
3. **Operations Log** — 全操作がスナップショットされ、いつでも`but undo`で巻き戻せる

## コマンドリファレンス

頻出コマンドのクイックリファレンス。詳細は `references/commands.md` を参照。

### 基本

| コマンド | 説明 |
|---|---|
| `but` / `but status` | 状態確認（ブランチ一覧 + 未コミット変更） |
| `but status -f` | ファイル単位の変更も表示 |
| `but init` | プロジェクトをGitButlerで初期化 |
| `but config` | 設定の確認・変更 |

### ブランチ操作

| コマンド | 説明 |
|---|---|
| `but branch new <n>` | 新しい並行ブランチを作成 |
| `but branch new -a <anchor> <n>` | スタック（依存）ブランチを作成 |
| `but branch list` | ブランチ一覧 |
| `but branch list --review` | PR/MRステータス付きで一覧 |
| `but apply <branch>` | ブランチをワークスペースに適用 |
| `but unapply <branch>` | ブランチをワークスペースから解除 |
| `but branch delete <branch>` | ブランチ削除 |

### コミット操作

| コマンド | 説明 |
|---|---|
| `but commit -m "msg"` | コミット（全未割当変更） |
| `but commit -m "msg" -b <branch>` | 指定ブランチにコミット |
| `but commit --files <id> -m "msg"` | 特定ファイルだけコミット |
| `but commit --ai` | AIでコミットメッセージ生成 |
| `but commit empty --before <id>` | 空コミットを挿入（分割用） |
| `but absorb` | 未コミット変更を最適なコミットに自動吸収 |
| `but rub <src> <dst>` | 2つのエンティティを結合（squash, amend等） |
| `but describe <commit> -m "msg"` | コミットメッセージ編集 |
| `but squash <range>` | コミットの圧縮 |
| `but move <commit> <target>` | コミットの移動 |

### Forge連携

| コマンド | 説明 |
|---|---|
| `but push` | リモートにプッシュ |
| `but push --dry-run` | プッシュ内容のプレビュー |
| `but publish` | PR/MRを作成・更新 |
| `but forge` | Forge（GitHub/GitLab）操作 |

### 履歴・リカバリ

| コマンド | 説明 |
|---|---|
| `but oplog` | 操作履歴 |
| `but undo` | 直前の操作を元に戻す |
| `but restore <snapshot>` | 特定スナップショットに復元 |
| `but snapshot -m "msg"` | 手動スナップショット作成 |

## Claude Code連携パターン

Claude Codeとの連携は3つの方式がある。プロジェクトの性質に応じて選択する。

### 方式比較

| | Hooks方式 | Plugin方式 | MCP方式 |
|---|---|---|---|
| 粒度 | Edit/Write毎 | タスク完了時 | 明示的 or ルール |
| 自動度 | 高 | 最高 | 低 |
| 並列セッション | ✅ | ✅ | △ |
| 推奨場面 | 細かい制御が欲しい | とにかく手軽に | 既存MCP環境がある |

### Hooks方式（推奨）

`.claude/settings.json`（プロジェクト固有）または`~/.claude/settings.json`（グローバル）に設定：

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Edit|MultiEdit|Write",
        "hooks": [
          { "type": "command", "command": "but claude pre-tool" }
        ]
      }
    ],
    "PostToolUse": [
      {
        "matcher": "Edit|MultiEdit|Write",
        "hooks": [
          { "type": "command", "command": "but claude post-tool" }
        ]
      }
    ],
    "Stop": [
      {
        "matcher": "",
        "hooks": [
          { "type": "command", "command": "but claude stop" }
        ]
      }
    ]
  }
}
```

### Plugin方式

```
/plugin marketplace add gitbutlerapp/claude
/plugin install gitbutler
```

### MCP方式

```bash
claude mcp add gitbutler but mcp
```

CLAUDE.mdに追記：
```markdown
## Development Workflow
- When you're done with a task, run the gitbutler update_branches MCP tool.
- Never use git commit directly.
```

### CLAUDE.mdへの共通ルール

どの方式でも、CLAUDE.mdに以下を追記してClaude Codeの直接git操作を防ぐ：

```markdown
## Development Workflow
- Never use the git commit command. GitButler handles all commits.
- Do not run git add, git commit, or git push directly.
- Use `but push` for pushing and `but publish` for creating PRs.
```

## ブランチ戦略パターン

プロジェクトの性質に応じた戦略パターン。詳細は `references/strategies.md` を参照。

### パターン1: Feature-Parallel（デフォルト推奨）

複数の独立した機能を並行開発するパターン。ソロ開発の基本形。

```
origin/main (base)
├── feature-auth      （認証機能）
├── feature-api       （APIエンドポイント）
└── fix-memory-leak   （バグ修正）
```

**使い方：**
```bash
but branch new feature-auth
but branch new feature-api
but branch new fix-memory-leak
# 3つのccセッションを同時実行
# GitButlerが変更を自動でブランチに振り分け
```

**PRフロー：** 各ブランチ独立で`but publish` → レビュー → マージ

### パターン2: Stacked-Feature（依存機能の段階的開発）

機能A → 機能B → 機能C と依存する場合。

```
origin/main (base)
└── feature-data-model     （データ層）
    └── feature-api        （API層、data-modelに依存）
        └── feature-ui     （UI層、apiに依存）
```

**使い方：**
```bash
but branch new feature-data-model
# data-modelの作業後...
but branch new -a feature-data-model feature-api
# api の作業後...
but branch new -a feature-api feature-ui
```

**PRフロー：** 下から順に`but publish` → bottom-upでレビュー・マージ。
底のコミットを修正しても上が自動rebaseされるので安心。

### パターン3: Hybrid（並行 + スタック混在）

実践的にはこれが最も多い。独立したfeatureとstacked featureが共存。

```
origin/main (base)
├── fix-urgent-bug         （独立した修正）
├── feature-auth           （独立した機能）
└── feature-data-model     （スタックの根）
    └── feature-api        （data-modelに依存）
```

## ワークフロー：ソロ開発の1日

典型的なソロ開発フローの例。

### 朝：計画と準備

```bash
but pull                    # upstream更新を取り込み
but status                  # 現在の状態確認
but branch list --review    # PR状況確認
```

### 作業中：Claude Code並列セッション

ターミナルマルチプレクサ（tmux/Zellij）で複数ペインを開く：

```bash
# pane 1: 新機能
cd ~/project && cc "feature: ユーザー認証をJWTで実装"

# pane 2: バグ修正
cd ~/project && cc "fix: /api/users のメモリリークを修正"

# pane 3: リファクタ
cd ~/project && cc "refactor: データベース層のエラーハンドリング統一"
```

GitButlerが各セッションの変更を自動で別ブランチに振り分ける。

### 作業後：整理とPR

```bash
but status -f               # 全ブランチの状態確認
but absorb                   # 未コミット変更を最適コミットに吸収

# コミット整理が必要なら
but squash <range>           # 不要な中間コミットを圧縮
but describe <commit> -m "better message"  # メッセージ改善

# PRに出す
but push                     # リモートにプッシュ
but publish                  # PR作成・更新
```

### トラブル時

```bash
but undo                     # 直前の操作を取り消し
but oplog                    # 操作履歴を確認
but restore <snapshot-id>    # 特定地点に復元
```

## 設計の原則

ブランチ戦略を設計する際の判断基準：

1. **1セッション = 1ブランチ** — Claude Codeの各セッションが独立したブランチになるよう設計する。混在させない
2. **スタックは3段まで** — 依存が深くなりすぎるとrebaseコンフリクトのリスクが増大する。3段を超えたらマージを検討
3. **absorb優先** — 手動でのコミット割り当てより`but absorb`を信頼する。GitButlerのhunk解析は賢い
4. **undoを躊躇わない** — Operations Logがあるので、失敗を恐れず試行錯誤できる
5. **PRは小さく** — stacked branchesの恩恵を活かし、大きな機能も小さなPRの連鎖にする

## JSON出力の活用

スクリプトやCI/CDとの連携には`--json`/`-j`オプションを使う：

```bash
but status -j | jq '.branches[].name'
but branch list -j | jq '.[] | select(.has_pr == true)'
but show <commit> -j
```

## 追加リファレンス

より詳細な情報は以下のファイルを参照：

- `references/commands.md` — 全コマンドの詳細オプション
- `references/strategies.md` — ブランチ戦略パターンの詳細と判断フロー
- `references/claude-code-integration.md` — Claude Code連携の設定詳細とトラブルシューティング
