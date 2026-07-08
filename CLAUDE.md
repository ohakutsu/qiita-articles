# CLAUDE.md

## リポジトリ概要

[ohakutsu - Qiita](https://qiita.com/ohakutsu) の記事を [qiita-cli](https://github.com/increments/qiita-cli) で管理するリポジトリ。ソースコードは含まず、記事の Markdown ファイルのみを扱う。

- `public/*.md`: 各記事。ファイル名は Qiita の記事 ID（新規記事は publish 時に ID へリネームされる）
- 各記事の先頭に YAML frontmatter（`title`, `tags`, `private`, `id`, `updated_at` など）がある。`id` と `updated_at` は qiita-cli が管理するため手動で書き換えない

## セットアップ

ツール（Node.js / qiita-cli）は mise で管理されている。

```bash
./devtools/scripts/setup-development-environment.sh  # mise trust && mise install
```

## よく使うコマンド

```bash
qiita new <ファイルのベース名>  # 新規記事の雛形を public/ に作成
qiita preview                   # ブラウザでプレビュー（http://localhost:8888）
qiita pull                      # Qiita 上の記事をローカルに取り込む
```

## 公開フロー

main ブランチへの push をトリガーに GitHub Actions（`.github/workflows/publish.yml`）が qiita-cli の publish アクションを実行し、Qiita へ反映する。ローカルから `qiita publish` を直接実行する必要はない。記事の作成・更新は PR を作って main にマージする。
