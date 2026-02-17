# cwflow

Claude Workers（`cw`コマンド）を使った並行開発ワークフローのClaude Codeプラグイン。
Gitクローンベースのワークスペース分離で、複数セッションを安全に並列実行。

## インストール

```bash
/plugin install chronista-club/claude-plugin-cwflow
```

## 概要

```
メインリポ (~/repos/my-project)    ← 安定。汚さない
│
├── cw new issue-42 feature/issue-42
│   → ~/.cache/creo-workers/issue-42/    ← Agent A
│
└── cw new issue-43 feature/issue-43
    → ~/.cache/creo-workers/issue-43/    ← Agent B
```

### コマンド

| コマンド | 説明 |
|---|---|
| `cw new <name> <branch>` | ワーカー環境を作成（clone + symlink + setup） |
| `cw ls` | 全ワーカー一覧 |
| `cw path <name>` | ワーカーのパスを出力 |
| `cw rm <name>` | ワーカーを削除 |

### ワークフロー

```bash
# 1. ワーカー作成
cw new issue-42 feature/issue-42

# 2. Claude Codeセッション起動
cd $(cw path issue-42) && claude

# 3. 作業完了 → PR → マージ
gh pr create ...

# 4. 削除
cw rm issue-42
```

### worker-files.kdl

各プロジェクトの `.claude/worker-files.kdl` でワーカー環境の設定を定義:

```kdl
symlink ".env"
symlink ".mcp.json"
symlink ".claude/settings.local.json"
symlink-pattern "**/*.local.*"
post-setup "bun install"
```

## 前提

- `cw` コマンド（`~/.cargo/bin/cw`）
- [Claude Code](https://claude.ai/code)

## License

MIT
