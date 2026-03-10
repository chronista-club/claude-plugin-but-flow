---
description: 未コミット変更を含む現在の状態からワーカー環境をフォーク（dirty stateの非破壊転送）
---

# CC Nav Fork — dirty state フォーク

## 手順

1. `git status --short` で現在の変更を確認し、表示する:
   - 変更がない場合は「dirty state がありません。`/ccnav:new` でクリーンなワーカーを作成してください」と案内して終了

2. ユーザーの指定を判定:

   **A) Issue番号が指定された場合** → Issue 連携モード
   - `gh issue view <number> --json number,title,labels` で情報取得
   - ワーカー名: `issue-<number>` （例: `issue-42`）
   - ブランチ名: `feature/issue-<number>` （bug/fix系なら `fix/issue-<number>`）

   **B) タスク名が直接指定された場合** → フリーモード
   - ワーカー名: そのまま使用（例: `auth-refactor`, `experiment-new-api`）
   - ブランチ名: `feature/<name>` （例: `feature/auth-refactor`）

   **C) 何も指定されない場合** → 選択を促す
   - 「ワーカー名とブランチ名を入力してください」と案内

3. 実行前に確認を表示:
   ```
   Worker: auth-refactor  (リポ名自動prefix: nexus-auth-refactor)
   Branch: feature/auth-refactor
   Path:   ~/.local/share/ccws/nexus-auth-refactor/
   Dirty:  3 files (2 modified, 1 untracked)

   元のワーキングツリーは変更されません。フォークしますか？
   ```

4. `ccws fork <name> <branch>` を実行
   - 既存ワーカーが存在してエラーになった場合は `--force` オプションの使用を案内

5. 成功したら:
   - ワーカーのパスを表示
   - 「`cd $(ccws path <name>) && claude` でセッションを開始できます」と案内
   - 「元のワーキングツリーの変更はそのまま残っています」と補足

## 引数

- `<issue-number>`: GitHub Issue番号（省略可）
- `<task-name>`: 自由なタスク名（省略可、Issue番号と排他）

## 使用例

```bash
# 今の変更をIssue連携でフォーク
/ccnav:fork 42

# フリーモード
/ccnav:fork experiment-wip

# 対話選択
/ccnav:fork
```

## 注意

- カレントディレクトリがgitリポジトリのルートであることを確認
- 元のワーキングツリーは変更されない（非破壊）
- binary ファイルも含めて dirty state が転送される
- 既存ワーカーがある場合は `--force` が必要
- `/ccnav:new` との違い: `new` はクリーンな状態から開始、`fork` は現在の未コミット変更を引き継ぐ
