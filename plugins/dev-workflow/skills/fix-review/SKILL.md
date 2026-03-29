---
name: fix-review
description: PR のレビュー指摘（未解決のインラインコメント）に対してコード修正を行う。PR 番号を指定する。
argument-hint: "<pr-number>"
---

# Fix Review Skill

PR のレビュー指摘（未解決のインラインコメント）を読み取り、コード修正を行う。

**このスキルはレビュー指摘への対応のみを行う。** プランに基づく新規実装は `/implement-plan` スキルを使用すること。

## 前提条件

- `gh` CLI が認証済みであること
- 対象の PR に未解決のレビュースレッドが存在すること

## 引数

```
$ARGUMENTS = <pr-number>
```

引数なしの場合はエラーとする。

## 手順

### 1. 未解決のレビュー指摘を取得

#### 1-1. 未解決のレビュースレッドを取得

```bash
gh api graphql -f query='
{
  repository(owner: "{owner}", name: "{repo}") {
    pullRequest(number: <pr-number>) {
      reviewThreads(first: 100) {
        nodes {
          isResolved
          comments(first: 10) {
            nodes {
              body
              path
              line
              diffHunk
              author { login }
            }
          }
        }
      }
    }
  }
}'
```

- `isResolved: false` のスレッドのみを対象とする
- bot コメント（CI 等）は除外する
- 各指摘の `path`, `line`, `diffHunk` から修正箇所を特定する
- 解釈不能な指摘はスキップし、対応できなかった旨をユーザーに報告する

未解決の指摘が存在しない場合は、その旨をユーザーに報告して終了する。

#### 1-2. PR のブランチ情報を取得

```bash
gh pr view <pr-number> --json headRefName,baseRefName
```

### 2. プロジェクト固有情報の確認

`.claude/skills/references/project-context.md` の「実装ガイド」セクションを読み込み、以下を把握する:

- ビルド・フォーマットコマンドと注意事項
- 言語固有の実装規約
- テストの配置ルール
- CI に委ねてよい項目

### 3. PR のブランチにチェックアウト

PR の `headRefName` にチェックアウトし、最新を pull する。

```bash
git checkout <headRefName>
git pull origin <headRefName>
```

### 4. 指摘に基づくコード修正

各レビュー指摘に対してコード修正を行う。

#### 修正時の注意事項

- 各指摘の `path` と `line` から対象箇所を正確に特定すること
- `diffHunk` を参考に、指摘の文脈を正しく理解すること
- 指摘の意図を汲み取り、表面的な修正ではなく本質的な改善を行うこと
- 既存のコードベースのパターン・命名規則に従うこと
- project-context.md に記載された言語固有の実装規約に従うこと
- 指摘に関連するテストの追加・修正が必要であれば対応すること

#### 対応できない指摘の扱い

以下の場合は指摘をスキップし、後でユーザーに報告する:

- 指摘の意図が不明瞭で解釈できない場合
- 指摘の対象ファイル・行が現在のコードと一致しない場合
- 指摘の対応に設計判断が必要で、自動的に判断できない場合

### 5. ビルドとフォーマットの確認

project-context.md の「ビルド・フォーマットコマンド」セクションに記載されたコマンドを実行する。

- フォーマッターは必ずコミット前に実行すること
- ビルドやテストが失敗した場合は原因を特定し修正すること。修正後に再度実行し、成功するまで繰り返す
- CI に委ねてよい項目として記載されたテストはスキップしてよい

### 6. コミット

変更内容をコミットする。

- コミットメッセージは修正内容を簡潔に記載すること
  - 例: `fix: improve error handling and add missing tests`
- 複数の論理的なまとまりがある場合は、適切にコミットを分割すること
- Co-Authored-By には実行時のモデル情報を使用すること

```bash
git add <files>
git commit -m "$(cat <<'EOF'
<commit message>

Co-Authored-By: <実行中のモデル名> <noreply@anthropic.com>
EOF
)"
```

### 7. Push と報告

1. リモートに Push する

```bash
git push origin <headRefName>
```

2. 対応結果をユーザーに報告する:
   - **対応した指摘**: 各指摘に対してどのような修正を行ったかの要約
   - **対応できなかった指摘**: スキップした指摘とその理由

## 注意事項

- レビュー指摘の意図を正確に理解し、過不足のない修正を行うこと
- 推測ではなく、実際のコードを読んで確認した事実に基づいて修正すること
- 指摘されていない箇所への変更は最小限に留めること（指摘対応に必要な範囲のみ）
- ビルドが通らない状態でコミット・Push しないこと
- project-context.md に記載されたフォーマッターを忘れずに実行すること
