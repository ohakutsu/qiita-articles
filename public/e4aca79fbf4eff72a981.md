---
title: RubyKaigi 2023 参加レポート「Revisiting TypeProf - IDE support as a primary feature」
tags:
  - Ruby
  - RubyKaigi
  - TypeProf
  - Ruby_記事投稿キャンペーン
private: false
updated_at: '2023-05-16T15:31:36+09:00'
id: e4aca79fbf4eff72a981
organization_url_name: qiita-inc
slide: false
---
この記事は [【RubyKaigi 2023連動イベント】みんなでRubyの知見を共有しよう - Qiita](https://qiita.com/official-events/4d2a9eed42dbe0a4af21) の参加記事です。

https://qiita.com/official-events/4d2a9eed42dbe0a4af21

## はじめに

執筆時点（2023/05/12）で開催中のRubyKaigi 2023に現地組として参加しております！
本日も面白いセッションがいくつもありました！
その中の以下のセッションについてまとめます。

## 追記

スライドが公開されていたので、ぜひそちらもご覧ください！

https://twitter.com/mametter/status/1658324122391437313?s=20

## Revisiting TypeProf - IDE support as a primary feature

https://rubykaigi.org/2023/presentations/mametter.html#day2

現状、TypeProfがうまく使われないことの考察から始まり、新しくIDEサポートを重視してつくっているTypeProf v2のデモを見せながらTypeProf v2の実装について話していました。
デモでは、VS Code上でメソッドや定数の定義ジャンプ、ifでのnilガード等がありました。

現在のTypeProf（v1）は、もともとRubyに型推論を入れることを目指し、推論の速度・不完全なコードでの動作は別にしていたようです。
しかし、実際に必要とされているのはIDEサポートで、推論の速度・不完全なコードでの動作が重要でした。

そこで、TypeProf v2の目標を「~10K, ~100K LOCの中規模プロジェクトでは、0.1秒以内に解析できる」にし、新しい解析アルゴリズム等を使い開発しているようです。

推論時間を短縮するために、新しい解析アルゴリズムでは全体のデータフローグラフから変更があった部分だけを更新する差分更新を採用しています。
（詳しい解析アルゴリズムは後日スライド等が公開されると思うので、そちらをご確認ください）

それによって、TypeProf v2自身のコード（7000行程度）では変更時に0.1秒を切る速度で解析ができるようになっているそうです！
（まだ、未実装の部分もあるようなのでパフォーマンスが変わる可能性もあるとのことでした）

## さいごに

他の言語を触っているとRubyのIDEサポートが充実していると良いなと思うタイミングは多々あるので、TypeProf v2でよりRubyを楽しく書けるようになるといいなと思います。
