---
title: Node.js Test Runnerでモジュールのモックをしたい
tags:
  - JavaScript
  - Node.js
private: false
updated_at: '2023-07-13T00:28:51+09:00'
id: 7145f083828df7f0e6e4
organization_url_name: qiita-inc
---

## はじめに

プライベートで書いているスクリプトのテストをしたいが、メンテナンスするパッケージを増やしたくないと思い、
Node.js v18 から標準で使えるようになった[Node.js Test Runner](https://nodejs.org/api/test.html) を使ってテストできないかを調べていました。

調べてみたところ、モジュールのモックをする方法で少し詰まったので、その解決方法を書きます。
なお、今回試すのは CJS です。

## やりたいテスト

今回やりたかったテストは、テスト対象のモジュール内で使用されている別のモジュールをモックするようなテストです。

例として、以下のスクリプトを使います。

```js:add.js
exports.add = (a, b) => {
  return a + b;
};
```

```js:mul.js
const { add } = require("./add");

exports.mul = (a, b) => {
  let result = 0;
  for (let i = 0; i < b; i++) {
    result = add(result, a);
  }
  return result;
};
```

そして、今回テストしたいのは`mul`関数で、`add`関数をモックしたいとします。

この場合であれば、[`mock.method`](https://nodejs.org/api/test.html#mockmethodobject-methodname-implementation-options) 関数を使い、以下のようにモックすることができます。

```js:mul.test.js
const { test, mock } = require("node:test");
const assert = require("node:assert");

// `mul`よりも先に`add`をモックする必要がある
const addModule = require("./add");
mock.method(addModule, "add", (a, b) => {
  return 1;
});

const { mul } = require("./mul");

test("mul", () => {
  assert.equal(mul(2, 3), 1);
});

```

同様にビルトインモジュールなどもモックできます。

```js:readFileSync.test.js
const { test, mock } = require("node:test");
const assert = require("node:assert");
const fs = require("node:fs");

mock.method(fs, "readFileSync", (path) => {
  return "hoge";
});

test("readFileSync", () => {
  assert.equal(fs.readFileSync("sample.txt"), "hoge");
});
```

なお、モックしたいモジュールが`module.exports`で定義されている場合、`mock.method`ではモックできないようです。（いろいろ試したけど、できる方法を見つけれなかった）

## Refs

- [Node 18 に Test Runner なるものがあったので試してみた（Experimental） - Qiita](https://qiita.com/ringtail003/items/5fda350bb273bab7e1c9)
- [Spies and mocking with Node test runner (node:test) | Željko Šević | Node.js Developer](https://sevic.dev/notes/spies-mocking-node-test-runner/)
- [Read Node.js Source Code to Deeply Understand the CJS Module System - Alibaba Cloud Community](https://www.alibabacloud.com/blog/read-node-js-source-code-to-deeply-understand-the-cjs-module-system_599765)
