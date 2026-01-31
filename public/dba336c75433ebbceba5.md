---
title: BigQueryのデータ分析で変数を使うと便利
tags:
  - BigQuery
private: false
updated_at: '2026-01-31T19:03:19+09:00'
id: dba336c75433ebbceba5
organization_url_name: qiita-inc
slide: false
ignorePublish: false
---

BigQueryを使ってユーザーの分析などをよくしているのですが、クエリを書く際に特定の日付やユーザーのidなどを扱うことはしばしばあると思います。
その際に、普段コードを書くときみたいに定数を使うことでクエリがスッキリするので紹介します。

## DECLAREを使う

https://cloud.google.com/bigquery/docs/reference/standard-sql/procedural-language#declare

`DECLARE`を使うことで定数を定義できます。

簡単な例が以下です。

```sql:例1
DECLARE target_user_id INT64 DEFAULT 123;

SELECT * FROM users WHERE id = target_user_id;
```

```sql:例2
DECLARE target_user_name STRING DEFAULT 'qiitan';

SELECT * FROM users WHERE name = target_user_name;
```

このように、クエリ中でマジックナンバーになりがちな値に名前をつけることができるので、長いクエリになってくるとより重宝します。

また、INT64やSTRING以外も使えるので活用できると便利です。
私がよく使うのはARRAYです。
例が以下です。

```sql:例3
DECLARE target_tag_names ARRAY<STRING> DEFAULT ['Claude', 'ClaudeCode'];

SELECT *
FROM tags
WHERE name IN (
  SELECT * FROM UNNEST(target_tag_names)
);
```

このようにIN句もスッキリしとても良いです。

## 最後に

半分くらいは自己満足で使っている部分もありますが、普段書いているコードと同じように意図が伝わるようなクエリを書けるのは気持ちが良いです。
ぜひお試しいただけると幸いです。
