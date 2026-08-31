---
title: TypeScript 7でtsc --lspが使えるようになったので、NeovimのLSP設定をts_lsからtscに変更した
tags:
  - TypeScript
  - neovim
  - LSP
private: false
updated_at: ''
id: null
organization_url_name: qiita-inc
slide: false
ignorePublish: false
posting_campaign_uuid: null
agreed_posting_campaign_term: false
---

## はじめに

TypeScript 7から、`tsc`に`--lsp`というオプションが追加され、これまで別パッケージだった`typescript-language-server`を使わなくてもlanguage serverとして動くようになったようです。TypeScript 7を使うプロジェクトで、Neovimの設定をこれまでの`typescript-language-server`（nvim-lspconfigの`ts_ls`）から`tsc`に変えたので、内容をまとめます。

## TypeScript 7でtsc --lspが使えるようになった

TypeScript 7.0は2026-07-08にリリースされた、Goで書き直されたネイティブコンパイラです。

https://devblogs.microsoft.com/typescript/announcing-typescript-7-0/

> The new foundation is built on the Language Server Protocol (LSP) and is able to leverage multiple threads to serve simultaneous requests as quickly as possible.

> 新しい基盤はLanguage Server Protocol（LSP）の上に構築されており、複数のスレッドを活用して同時リクエストをできる限り高速に処理できます。
>
> _Claudeによる日本語訳_

ただし、公式ドキュメントや`tsc --help`には`--lsp`オプション自体の記載が見当たりません。nvim-lspconfigや他のツールのリポジトリでは`tsc --lsp --stdio`がTypeScript 7のlanguage serverとして扱われているので、TypeScript 7以降はこれを使うとよさそうです。

https://github.com/neovim/nvim-lspconfig/issues/4467
https://github.com/Yeachan-Heo/oh-my-claudecode/issues/3403

これまでは`tsserver`をラップする`typescript-language-server`をnpmで別途インストールし、nvim-lspconfigでは`ts_ls`として設定していました。`tsc --lsp --stdio`が使えるようになったことで、別パッケージが不要になっています。

nvim-lspconfig側にもこれに対応する`lsp/tsc.lua`が追加されました。

https://github.com/neovim/nvim-lspconfig/blob/master/lsp/tsc.lua

> The language server (`--lsp`) is only available in the native compiler, TypeScript 7.0 and newer. An older binary in `node_modules/.bin` is skipped in favour of one on `$PATH` that does support it; if no candidate qualifies, the server does not attach.

> language server（`--lsp`）はネイティブコンパイラ、つまりTypeScript 7.0以降でのみ利用できます。`node_modules/.bin`にある古いバイナリは、それをサポートする`$PATH`上のバイナリがあればそちらが優先され、どの候補も条件を満たさない場合はサーバーはattachしません。
>
> _Claudeによる日本語訳_

## 設定を変える

`vim.lsp.enable()`に渡すサーバー名のリストで、`ts_ls`を`tsc`に置き換えるだけです。

```lua:~/.config/nvim/lua/lsp.lua
-- Before
vim.lsp.enable({
  "ts_ls",
  -- 略...
})

-- After
vim.lsp.enable({
  "tsc",
  -- 略...
})
```

## 動かないときは

`node_modules/.bin`にも`$PATH`にも`--lsp`に対応したTypeScript 7以上の`tsc`（または`tsgo`）が1つも見つからない場合、以下の警告が出てサーバーはattachしません。

```
tsc: no binary supporting `--lsp` found (requires TypeScript 7.0+)
```

逆に言うと、プロジェクトの`node_modules`のTypeScriptが7未満でも、`$PATH`上にTypeScript 7以上の`tsc`があればそちらが使われ、警告なしにattachします。

これは`ts_ls`とは異なる挙動でした。試してみたところ、`ts_ls`はプロジェクトに`typescript`パッケージがインストールされていないと、明示的なエラーで起動に失敗しました。一方`tsc`は`--lsp`に対応するバイナリかどうかしか見ていないため、プロジェクトの`node_modules`のTypeScriptが古くても、`$PATH`上のTypeScript 7が代わりに使われてしまうことがあります。手元でグローバルにTypeScript 7をインストールしている場合は、気づかないうちにこれが起きるので注意してください。

## さいごに

TypeScript 7で`tsc --lsp`が使えるようになったことで、Neovim側の設定は`ts_ls`から`tsc`への1行の置き換えで済みました。
