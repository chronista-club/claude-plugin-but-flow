---
description: 並列ワーカーの変更ファイルを比較し、マージコンフリクトの可能性を早期検知する。
---

# CW Conflict Check — コンフリクト早期検知

## 手順

1. `cw ls` でアクティブなワーカー一覧を取得

2. 各ワーカーの変更ファイル一覧を収集:
   ```bash
   # 各ワーカーディレクトリで
   cd $(cw path <name>)
   git diff --name-only main...HEAD
   ```

3. 変更ファイルの重複を検出:
   - 2つ以上のワーカーが同じファイルを変更している場合 → ⚠️ 警告

4. 結果を表示:

   ```
   ## コンフリクト検知結果

   ✅ issue-42 × issue-43: 重複なし
   ⚠️ issue-42 × issue-44: 2ファイルが重複
     - src/auth/middleware.ts
     - src/auth/types.ts

   推奨: issue-44 のマージは issue-42 のマージ後に行ってください。
   ```

5. ccwire 登録済みなら、コンフリクトがある場合に該当ワーカーへ wire_send で警告:
   ```
   wire_send(to: "worker-issue-44", content: "⚠️ src/auth/middleware.ts が issue-42 と競合の可能性があります", type: "conflict_warning")
   ```

## 引数

なし（全ワーカーを自動チェック）

## 推奨利用タイミング

- Wave 切り替え前
- PR 作成前
- リーダーが `/cwflow:dashboard` を更新する際
