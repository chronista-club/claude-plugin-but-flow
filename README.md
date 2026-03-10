# ccnav

Claude Code Navigator — `ccws` を使った並行開発ワークフローの Claude Code プラグイン。
Gitクローンベースのワークスペース分離で、複数セッションを安全に並列実行。

## インストール

```bash
/plugin install chronista-club/claude-plugin-ccnav
```

## 概要

```
メインリポ (~/repos/my-project)    <-- 安定。汚さない
|
+-- ccws new issue-42 feature/issue-42
|   -> ~/.local/share/ccws/issue-42/    <-- Agent A
|
+-- ccws new issue-43 feature/issue-43
    -> ~/.local/share/ccws/issue-43/    <-- Agent B
```

## `ccws` CLI コマンド

`ccws` はシステムにインストールするCLIツール（`~/.cargo/bin/ccws`）。プラグインのコマンドではない。

| コマンド | 説明 |
|---|---|
| `ccws new <name> <branch>` | ワーカー環境を作成（clone + symlink + setup） |
| `ccws fork <name> <branch> [--force]` | dirty state（未コミット変更）を含めてワーカー環境をフォーク |
| `ccws ls` | 全ワーカー一覧 |
| `ccws path <name>` | ワーカーのパスを出力 |
| `ccws rm <name>` | ワーカーを削除 |

## Claude Code コマンド

プラグインが提供するスラッシュコマンド:

| コマンド | 説明 |
|---|---|
| `/ccnav:new [issue番号 or タスク名]` | ワーカー環境を作成。Issue番号指定で自動命名、フリーモードも可 |
| `/ccnav:fork [issue番号 or タスク名]` | 未コミット変更を含む現在の状態からワーカー環境をフォーク |
| `/ccnav:parallel [worker-names...]` | tmuxで複数ワーカーのClaude Codeセッションを並列起動（ccwire自動登録付き） |
| `/ccnav:dashboard` | Vantage Pointに並列開発ダッシュボードを表示 |
| `/ccnav:status` | 全ワーカーの状態一覧（ブランチ、変更数、PR状態） |
| `/ccnav:cleanup` | マージ済み・不要なワーカーを検出して一括削除 |
| `/ccnav:conflict-check` | 並列ワーカーの変更ファイル重複を検知し、コンフリクトを早期警告 |

## Leader-Worker パターン

```
Leader (メインリポ, ccwire: "lead")
+-- /ccnav:new 42
+-- /ccnav:new 43
+-- /ccnav:parallel issue-42 issue-43
+-- wire_status で進捗監視
+-- wire_receive でワーカーからの質問に回答
```

## ワークフロー

```bash
ccws new issue-42 feature/issue-42
cd $(ccws path issue-42) && claude
gh pr create ...
ccws rm issue-42
```

## 前提

- `ccws` コマンド（`~/.cargo/bin/ccws`）
- [Claude Code](https://claude.ai/code)

## License

MIT
