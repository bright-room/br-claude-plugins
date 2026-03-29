# br-claude-plugins

Claude Code のスキルをプラグインとしてまとめたリポジトリ。
リポジトリ横断で再利用できる開発ワークフローを提供する。

## Plugins

| プラグイン | 概要 |
|-----------|------|
| [dev-workflow](./plugins/dev-workflow/) | GitHub Issue を起点にプラン作成・実装・レビュー・棚卸しまでを一貫して扱うワークフロースキル群 |

## dev-workflow の使い方

### できること

- `/create-plan #N` — Issue からコードベースを調査して実装プランを作成
- `/update-plan #N` — フィードバックを反映してプランを更新
- `/implement-plan #N` — プランに従い実装して PR を作成
- `/fix-review <PR>` — PR の未解決レビューコメントに対応してコードを修正
- `/review <PR>` — PR の差分をレビューしてインラインコメント投稿
- `/triage` — Issue の一括棚卸し（クローズ・ラベル・マイルストーン整理）

### ユースケース

**新機能を開発する**

```
1. GitHub Issue を作成
2. /create-plan #42          → プランが Issue コメントに投稿される
3. レビュー後 /implement-plan #42  → ブランチ作成 → 実装 → PR 作成
4. /review 123                → PR にレビューコメントが付く
5. 指摘があれば /fix-review 123    → 未解決コメントに対応して修正・プッシュ
```

**バグ修正やドキュメント更新にも同じフローが使える。**

**定期的な棚卸し**

```
/triage → 完了済み Issue のクローズ、ラベル整理、マイルストーン平準化をまとめて実行
```

**ローカルの設計ドキュメントから実装する**

```
/implement-plan ./docs/design.md --branch feat/new-api
```

### セットアップ

1. Claude Code でこのプラグインをインストール
2. `gh` CLI を認証しておく
3. 各リポジトリに `.claude/skills/references/project-context.md` を配置してプロジェクト固有の設定を行う（[テンプレート](./plugins/dev-workflow/skills/references/project-context-template.md)）

詳細は [dev-workflow の README](./plugins/dev-workflow/README.md) を参照。
