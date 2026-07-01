---
title: cmux markdownコマンドでMarkdownファイルのプレビューする
tags:
  - Markdown
  - ClaudeCode
  - cmux
private: false
updated_at: '2026-07-01T19:19:39+09:00'
id: f3ac44e867dbff29bc2f
organization_url_name: qiita-inc
slide: false
ignorePublish: false
---

最近Claude CodeでMarkdownファイルを書いたり読んだりする機会が増えてきました。
そんな中で、cmux markdownコマンドを使うと手元でサッとプレビューできて便利だったので、紹介します。

## cmux markdownコマンドの説明

cmux markdownコマンドは、Markdownファイルを整形された状態のビューアで開いてくれるコマンドです。
ファイルの変更を監視してくれるので、編集して保存すればプレビュー側もリアルタイムに更新されます。

```console
$ cmux markdown --help
cmux markdown

Usage: cmux markdown open <path> [options]
       cmux markdown <path>       (shorthand for 'open')

Open a markdown file in a formatted viewer panel with live file watching.
The file is rendered with rich formatting (headings, code blocks, tables,
lists, blockquotes) and automatically updates when the file changes on disk.

Options:
  --workspace <id|ref|index>   Target workspace (default: $CMUX_WORKSPACE_ID)
  --surface <id|ref|index>     Source surface to split from (default: focused surface)
  --window <id|ref|index>      Target window
  --direction <left|right|up|down>  Split direction (default: right)

Examples:
  cmux markdown open plan.md
  cmux markdown ~/project/CHANGELOG.md
  cmux markdown open ./docs/design.md --workspace 0
  cmux markdown open plan.md --direction down
```

以下のようにMarkdownファイルのパスを渡すだけで、cmux内でプレビューが閲覧できます。

```console
$ cmux markdown path/to/article.md
```

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/352836/764c27f8-c986-470f-9280-10a0839e1965.png)

## Claude Codeのrulesと組み合わせて自動プレビューさせる

毎回手で`cmux markdown`を打つのは面倒なので、Claude Codeのrules機能と組み合わせると便利です。
私は以下のように、`~/.claude/rules/markdown-preview.md`のようなルールファイルを用意して、cmux markdownコマンドを使うように教えておいてます。
（今だとruleが呼び出されないケースがあるため、もう少し改善できそう）

```markdown:~/.claude/rules/markdown-preview.md
---
paths:
  - "**/*.md"
---

# Markdownファイルに関するルール

Markdownファイルをユーザーに見てもらいたいときは、`cmux markdown`コマンドでプレビューを開いてください。
```

## さいごに

cmux markdownコマンド単体でも便利ですが、Claude Codeのrulesと組み合わせると、こちらが意識しなくても勝手にプレビューしてくれるようになって快適です。
