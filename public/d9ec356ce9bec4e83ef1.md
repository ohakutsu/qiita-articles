---
title: QiitaのMarkdownパーサー変更にともなう、コードブロック・コードスパンについてのいろいろな書き方と注意点
tags:
  - Qiita
  - Markdown
  - GitHubFlavoredMarkdown
private: false
updated_at: '2022-02-15T11:05:49+09:00'
id: d9ec356ce9bec4e83ef1
organization_url_name: qiita-inc
slide: false
---
## 記事投稿シリーズについて

QiitaでGitHub Flavored Markdown (以下GFM) に準拠したMarkdownパーサーが[ベータ版として公開されました！](https://blog.qiita.com/replace-markdown-parser-beta/)

[Qiitaアカウントによる説明記事](https://qiita.com/Qiita/items/e87ebe3138f62b0e8b58)にも書いてあるのですが、今までと記法が変わっている箇所があるので注意が必要です。
せっかく書いた記事の表示が崩れてしまうのは困るので、可能な限り防ぎたいと考えています！

そこで、記事を書く時に注意すべき記法などについて、Qiita運営から5日間に分けて10記事のシリーズで投稿することになりました！
少しでも皆様が執筆する際の助けになればと思います。

詳しくはQiitaアカウントによる説明記事に、投稿シリーズの説明や、他の記事一覧が載っていますのでぜひ他の記事も読んでみてください！

https://qiita.com/Qiita/items/e87ebe3138f62b0e8b58

この記事では[GFMの記法](https://github.github.com/gfm/)のうち、「コードブロック」と「コードスパン」について説明します。

## コードブロック

### 基本の書き方

コードブロックを書くときの基本形は以下のようになります。

    ```
    puts "hello"
    ```

見え方は以下のようになります。

```
puts "hello"
```

### 書き方の種類

GFMのコードブロックは大きく分けて2種類の書き方があります。

1. Fenced code blocks
1. Indented code blocks

### Fenced code blocks

`Fenced code blocks`はよく知られているコードブロックの書き方です。

**バッククォート（<code>\`</code>）**　もしくは**チルダ（`~`）** を3つ以上つなげたコードフェンスで囲います。

    ```
    puts "hello"
    ```

    もしくは

    ~~~
    puts "hello"
    ~~~

<code>```</code> `~~~`がフェンス（柵）のように見えるため、コードフェンスと呼ばれています。

#### コードブロックに情報を付与する

始まりのコードフェンスの後ろには`info string`と呼ばれるテキストを追加することができます。

    ```ruby
    puts "hello"
    ```

    もしくは

    ~~~ruby
    puts "hello"
    ~~~

[GitHub](https://github.com/)や[Qiita](https://qiita.com/)などではこの`info string`にコードブロック内の言語を指定して、シンタックスハイライトできるようになっています。
（正確には、GitHub・Qiita側で`info string`の中身を見てシンタックスハイライトを有効にする処理が用意されている）

例）

    ```ruby
    puts "hello"
    ```

↓ シンタックスハイライトが有効になる

```ruby
puts "hello"
```

また、Qiitaではシンタックスハイライトの他に、

- ファイル名
    - `info string`を`言語名:ファイル名`とする
- diff
    - `info string`を`diff`とする
    - `diff_言語名`とするとシンタックスハイライトも有効になる

が使えます。

例1）

    ```ruby:main.rb
    puts "hello"
    ```

↓ ファイル名がつく

```ruby:main.rb
puts "hello"
```

例2）

    ```diff_ruby
    def foo
    +  "foo"
    -  "bar"
    end
    ```

↓ `+` `-`を使うとdiffを表現できる

```diff_ruby
def foo
+  "foo"
-  "bar"
end
```

例3）

    ```diff_ruby:main.rb
    def foo
    +  "foo"
    -  "bar"
    end
    ```

↓ 組み合わせ技も可能

```diff_ruby:main.rb
def foo
+  "foo"
-  "bar"
end
```

#### :pencil: Tips

##### :white_check_mark: バッククォート・チルダは3つである必要はない

> **バッククォート（<code>\`</code>）** もしくは **チルダ（`~`）** を3つ以上つなげたコードフェンスで囲います。

の通り、 **3つ以上** であれば何個でも大丈夫です。

例1）

    ``````
    puts "hello"
    ``````

↓

``````
puts "hello"
``````

例2）

    ~~~~~~~~~~
    ```
    puts "hello"
    ```
    ~~~~~~~~~~

↓

~~~~~~~~~~
```
puts "hello"
```
~~~~~~~~~~

##### :white_check_mark: コードフェンスの先頭にスペースを3つまで入れられる

あまり使うイメージが湧きませんが、 **始まりのコードフェンスの先頭にスペースを3つまで入れることができます。**

例）

~~~
   ```
   puts "hello"
   ```
~~~

↓

   ```
   puts "hello"
   ```

始まりのコードフェンスの先頭にあるスペース数だけ、コードブロックの各行の先頭から切り取られます。

詳しくは、https://github.github.com/gfm/#fenced-code-blocks のExampleを見るとわかりやすいです。

##### :white_check_mark: コードブロックの前後に空行が不要

下の方で説明する`Indented code blocks`では前後に空行が必要ですが、`Fenced code blocks`では空行が不要です。

例）

~~~
hoge
```
foobar
```
fuga
~~~

↓

hoge
```
foobar
```
fuga

#### :warning: 注意点

##### :exclamation: 始まりと終わりのコードフェンスを同じフェンスにする必要がある

**始まりと終わりのコードフェンスは同じ文字列** でなければなりません。

    # OK
    ```
    puts "hello"
    ```

    # OK
    ~~~~~~
    puts "hello"
    ~~~~~~

    # NG （フェンスの種類が違う）
    ```
    puts "hello"
    ~~~

    # NG （数が違う）
    ```
    puts "hello"
    ``````

### Indented code blocks

`Indented code blocks`はバッククォートやチルダを使わず、 **4つのスペースでインデントさせる方法** です。

```
    puts "hello"
```

#### :warning: 注意点

##### :exclamation: コードブロックの前後に空行が必要

`Indented code blocks`の **前後の行には空行が必要** です。

```
# OK
（空行）
    puts "hello"
（空行）

# NG
foobar
    puts "hello"
```

また、`Indented code blocks`内で空行を含めたい場合は、スペース4つのみの行を用意することで実現できます。

例）

```
    def foo
      "foo"
    end
    
    def bar
      "bar"
    end
```

↓

    def foo
      "foo"
    end
    
    def bar
      "bar"
    end

##### :exclamation: `info string`をつけれない

`Fenced code blocks`で使える **`info string`をつけることはできません。**
そのため、GitHubやQiitaなどでの **シンタックスハイライト・ファイル名をつけることができない** です。


##### :exclamation: 4つより多いスペースは切り取られない

4つより多くのスペースがある場合は、 **先頭4つのスペースのみが切り取られ、残りのスペースはそのまま** になります。

```
    スペース4つ

     スペース5つ
```

↓ `スペース5つ`の方の行頭にスペースが1つ含まれている

    スペース4つ

     スペース5つ

## コードスパン

### 基本の書き方

コードスパンを書くときの基本形は以下のようになります。

~~~
`Ruby on Rails`
~~~

見え方は以下のようになります。

`Ruby on Rails`

### 定義

コードスパンは、 **バッククォート（<code>\`</code>）** を1つ以上つなげたバックティック文字列で囲います。
上記の場合、バックティック文字列は<code>\`</code>になります。

### :pencil: Tips

#### :white_check_mark: バッククォートは1つである必要はない

> コードスパンは、 **バッククォート（<code>\`</code>）** を1つ以上つなげたバックティック文字列で囲います。

の通り、1つ以上であれば何個でも大丈夫です。

例1）

~~~
``foobar``
~~~

↓ バックティック文字列は` `` `になる

``foobar``

例2）

~~~
```　コードスパンは`こうやってかく` ```
~~~

↓ バックティック文字列は` ``` `になる

`` コードスパンは`こうやってかく` ``

例3）

~~~
` `` ``` `` ``` ` ``
~~~

↓ バックティック文字列は<code>\`</code>になる（先頭から解析される）

` `` ``` `` ``` ` ``

#### :white_check_mark: コードスパン記法内で改行ができる

バックティック文字列で囲まれたテキストは改行をすることができます。
ただし、 **改行は半角スペースに変換される** ため注意が必要です。

例1）

~~~
`foo
bar`
~~~

↓ 改行が半角スペースになります

`foo
bar`

例2）

~~~
``
foo
bar
``
~~~

↓ コードブロックに似てるけどコードスパン

`
foo
bar
`

### :warning: 注意点

#### :exclamation: 始まりと終わりのバックティック文字列を同じ文字列にする必要がある

**始まりと終わりのバックティック文字列は同じ文字列** でなければなりません。

~~~
# OK
`Ruby on Rails`

# OK
``foo`bar``

# NG （始まりと終わりのバックティック文字列が違う）
``foo bar```
~~~

## 最後に

他の記法についてもまた別の記事でまとめていますので、合わせて読んでみてください！
詳しくは[Qiitaアカウントによる説明記事](https://qiita.com/Qiita/items/e87ebe3138f62b0e8b58)をご覧ください！

---

↓記事投稿シリーズまとめ

https://qiita.com/Qiita/items/e87ebe3138f62b0e8b58

↓次の記事

https://qiita.com/aizawa213/items/9c7b1d3a41293e0e3812

↓前の記事

https://qiita.com/kyntk/items/4fd09e6fb041edeac2d9
