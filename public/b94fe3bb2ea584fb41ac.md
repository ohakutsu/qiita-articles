---
title: PlaywrightでChrome拡張機能のE2Eテストを行う
tags:
  - chrome-extension
  - e2e
  - Playwright
private: false
updated_at: '2026-01-31T19:03:19+09:00'
id: b94fe3bb2ea584fb41ac
organization_url_name: qiita-inc
slide: false
ignorePublish: false
---

## はじめに

とあるタイミングで、Playwrightを使ってChrome拡張機能のE2Eテストを実装しました。
Chrome拡張機能（Manifest v3）のE2Eテストを行う環境の構築方法と、Tipsをまとめます。

## E2Eテスト環境の構築

playwrightのインストール方法などは省略し、Chrome拡張機能のE2Eテスト固有の設定に絞って説明します。
なお、説明する内容はPlaywrightの公式ドキュメントに記載されているものと同じなので、詳細はそちらを参照してください。

https://playwright.dev/docs/chrome-extensions

### fixtureの作成

Chrome拡張機能をロードした環境を用意するための[fixture](https://playwright.dev/docs/test-fixtures)を作成します。

1. `context` fixture
   - テスト対象のChrome拡張機能をロードしたブラウザコンテキストを作成しておく
2. `extensionId` fixture
   - 拡張機能のポップアップなどにアクセスするためのIDを取得できるようにしておく

```typescript:fixtures.ts
import { test as base, chromium, type BrowserContext } from '@playwright/test';
import path from 'node:path';

export const test = base.extend<{
  context: BrowserContext;
  extensionId: string;
}>({
  context: async ({ }, use) => {
    const pathToExtension = path.join(__dirname, 'my-extension'); // ビルド済みのChrome拡張機能のパスを指定する
    const context = await chromium.launchPersistentContext('', {
      channel: 'chromium',
      args: [
        `--disable-extensions-except=${pathToExtension}`,
        `--load-extension=${pathToExtension}`,
      ],
    });
    await use(context);
    await context.close();
  },
  extensionId: async ({ context }, use) => {
    // for manifest v3:
    let [background] = context.serviceWorkers();
    if (!background)
      background = await context.waitForEvent('serviceworker');

    const extensionId = background.url().split('/')[2];
    await use(extensionId);
  },
});
export const expect = test.expect;
```

また、以下のようなfixtureも用意しておくと、ポップアップのテストのたびにURLを手動で指定する必要がなくなるので便利です。

```diff_typescript:fixtures.ts
export const test = baseTest.extend<{
  context: BrowserContext;
  extensionId: string;
+ extensionUrl: (
+   pathname: string,
+   searchParams?: { [key: string]: string }
+ ) => string;
}>({
  // 略
+ extensionUrl: async ({ extensionId }, use) => {
+   await use((pathname, searchParams) => {
+     const url = new URL(pathname, `chrome-extension://${extensionId}`);
+     if (searchParams) {
+       url.search = new URLSearchParams(searchParams).toString();
+     }
+
+     return url.toString();
+   });
+ },
});
export const expect = test.expect;
```

### テストの実装

テストを実装する際は、先ほど作成したfixtureを使う以外は通常のPlaywrightのテストと同じです。

```typescript:sample.spec.ts
import { test, expect } from './fixtures';

test('example test', async ({ page }) => {
  await page.goto('https://example.com');
  await expect(page.locator('body')).toHaveText('Changed by my-extension');
});

test('popup page', async ({ page, extensionId }) => {
  await page.goto(`chrome-extension://${extensionId}/popup.html`);
  await expect(page.locator('body')).toHaveText('my-extension popup');
});

// extensionUrlを使う場合
test('popup page', async ({ page, extensionUrl }) => {
  await page.goto(extensionUrl('popup.html'));
  await expect(page.locator('body')).toHaveText('my-extension popup');
});
```

## Chrome拡張機能のE2EテストのTips

### アクティブなタブを含んだ機能のテストをしたい場合

以下のようにアクティブなタブを取得して処理を行う機能の場合、テストでは任意のタブをアクティブなタブとしたいと思います。

```typescript
async function getActiveTab() {
  const tabs = await chrome.tabs.query({
    active: true,
    currentWindow: true,
  });

  return tabs[0];
}
```

[Chrome 拡張機能のエンドツーエンド テスト | Chrome Extensions | Chrome for Developers](https://developer.chrome.com/docs/extensions/how-to/test/end-to-end-testing?hl=ja) によると、

> 現時点では、別のページのコンテキストで広告表示オプションのポップアップを開くことはできません。代わりに、新しいタブでポップアップの URL を開きます。ポップアップでアクティブなタブを使用している場合は、タブ ID をポップアップに明示的に渡すオーバーライドの実装を検討してください。例:

とありますが、テスト側で任意のタブのIDを取得する方法が見つけられませんでした。
そのため、以下のようにアクティブなタブが開いているページのURLをクエリパラメータで指定しタブを取得する方法が、できる範囲でのベストではと思っています。

```typescript
async function getActiveTab() {
  const urlParams = new URLSearchParams(window.location.search);
  const activeTabUrlForTest = urlParams.get('test-active-tab-url');
  if (activeTabUrlForTest) {
    const tabs = await chrome.tabs.query({
      url: activeTabUrlForTest,
    });
    return tabs[0];
  }

  const tabs = await chrome.tabs.query({
    active: true,
    currentWindow: true,
  });

  return tabs[0];
}
```

テスト側では以下のようになります。

```typescript
test('popup page', async ({ context, extensionUrl }) => {
  const activeTabPage = await context.newPage();
  const activeTabUrl = 'https://example.com';
  await activeTabPage.goto(activeTabUrl);

  const extensionPage = await context.newPage();
  await extensionPage.goto(extensionUrl('popup.html', { 'test-active-tab-url': activeTabUrl }));

  await expect(...)
});
```

## さいごに

Playwrightを使ったChrome拡張機能のE2Eテストの環境構築とTipsについてまとめました。
Chrome拡張機能のE2Eテストの知見があまりなく、苦労した部分も多かったので、同じようなことをしている方の参考になれば幸いです。
