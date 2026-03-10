---
name: ccnav
description: >
  Claude Code Navigator — ccws を使った並行開発ワークフロー。
  Gitクローンベースのワークスペース分離で、複数のClaude Codeセッションを
  安全に並列実行するためのスキル。

  このスキルは以下のような場面で使う：
  - 「並行開発したい」「複数Issueを同時に進めたい」
  - 「ccwsコマンドの使い方」「ワーカー環境の作り方」
  - 「worker-files.kdlの設定方法」
  - 「Claude Codeで安全に並列セッションを走らせたい」
  - Issue単位の作業環境分離
---

# CC Nav — Claude Code Workspace 並行開発ワークフロー

`ccws`（Claude Code Workspace）はGitクローンベースのワークスペースマネージャー。
メインリポジトリを汚さず、Issue単位で完全に独立した作業環境を作成する。

## コンセプト

```
メインリポ (~/repos/my-project)
│  ← 安定。直接作業しない or リーダーのみ
│
├── ccws new issue-42 feature/issue-42
│   → ~/.local/share/ccws/issue-42/    ← Agent A が作業
│
├── ccws new issue-43 feature/issue-43
│   → ~/.local/share/ccws/issue-43/    ← Agent B が作業
│
└── ccws new hotfix-1 fix/urgent-bug
    → ~/.local/share/ccws/hotfix-1/    ← Agent C が作業
```

各ワーカーは:
- **完全なgitクローン** — ブランチ、コミット、pushが完全に独立
- **secretsはsymlink共有** — `.env`, `.mcp.json` 等は元リポと同期
- **使い捨て** — 作業完了・マージ後に `ccws rm` で削除

## コマンドリファレンス

| コマンド | 説明 |
|---|---|
| `ccws new <name> <branch>` | ワーカー環境を作成（clone + symlink + setup） |
| `ccws fork <name> <branch> [--force]` | dirty state（未コミット変更）を含めてワーカー環境をフォーク |
| `ccws ls` | 全ワーカー一覧（名前、ブランチ、パス） |
| `ccws path <name>` | ワーカーのパスを出力 |
| `ccws rm <name>` | ワーカーを削除 |
| `ccws rm --all --force` | 全ワーカーを削除 |

## worker-files.kdl 設定

各プロジェクトの `.claude/worker-files.kdl` に、ワーカー環境へ配置するファイルを定義する。

```kdl
symlink ".env"
symlink ".mcp.json"
symlink ".claude/settings.local.json"
symlink-pattern "**/*.local.*"
symlink-pattern "**/*.local"
copy "config/dev.toml"
post-setup "bun install"
```

## 並行開発パターン

### パターン1: 手動並列
```bash
cd $(ccws path issue-42) && claude
cd $(ccws path issue-43) && claude
```

### パターン2: Leader-Worker（ccnav + ccwire 統合）
```
Leader (メインリポ, ccwire: "lead")
├── ccws new issue-42 feature/issue-42
├── /ccnav:parallel issue-42 issue-43
│   ├── tmux pane 0: worker-issue-42
│   └── tmux pane 1: worker-issue-43
├── wire_status で進捗監視
└── wire_receive で質問に回答
```

## clone vs worktree

| 方式 | ccws (clone) | EnterWorktree (CC標準) |
|------|-----------|----------------------|
| 独立性 | 完全独立（別 .git） | .git 共有 |
| 適用場面 | 長期並列開発 | 短期 isolated 作業 |

## ストレージ

`~/.local/share/ccws/`（XDG準拠）。`CCWS_WORKERS_DIR` 環境変数で変更可能。
