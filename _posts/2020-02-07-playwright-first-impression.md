---
layout: post
title: "Playwrightファーストインプレッション"
date: 2020-02-07 09:00:00 +0900
tags: Selenium テスト テスト自動化 Puppeteer Playwright
---

## Playwrightとは

2020年の1月末頃から話題になっているNode.js向けのブラウザ自動化ライブラリです。GithubのMicrosoftリポジトリで公開されています。

<https://github.com/Microsoft/playwright>

READMEの記述[^1]によると、GoogleでPuppeteerを開発していたチームがその後移って来て開発を開始したそうです。
Puppeteerとの差異としてよりtesting-friendlyなAPIを目指すこと、ChromeだけでなくFirefox、Webkitにも単一のAPIで対応すること、そのために互換性を持たない変更が必要になったためPuppeteerからコードベースを作り直しているたことなどが表明されています。

この記事ではPlaywrightをさわってみた所感をPuppeteerやWebDriverと比較をしながら書いていこうと思います。

Playwrightのバージョンは執筆時で最新の`v0.10.0`、Puppeteerは同じく最新の`v2.1.0`です。

## Puppeteerからの変更点

Puppeteerとの比較は同じテストシナリオを実装する際のAPI（メソッド）の違いを中心に見ていきます。

### 今回のシナリオ

サンプルとして使用するテストシナリオです。PlaywrightのREADMEに記載されているサンプルとほぼ同様の内容にしています。

1. ブラウザを起動してGoogle Pixel 2と同じ画面解像度・UserAgentに設定する
2. 現在地の位置情報を新宿駅の緯度経度に設定する
3. Googleマップを開く
4. 「現在地」のボタンを押す
5. Ajax通信の完了を待機する
6. スクリーンショットを取得する
7. ブラウザを閉じる

### 今回のサンプルコード

まず、今回のシナリオをPuppeteerを使って書くと以下のようなコードになります。

```javascript
const puppeteer = require('puppeteer');
const pixel2 = puppeteer.devices['Pixel 2'];

(async () => {
  const browser = await puppeteer.launch();
  await browser.defaultBrowserContext().overridePermissions('https://www.google.co.jp', ['geolocation']);
  const page = await browser.newPage();
  await page.setViewport(pixel2.viewport);
  await page.setUserAgent(pixel2.userAgent);
  await page.setGeolocation({ latitude: 35.690833, longitude: 139.700278 });

  await page.goto('https://www.google.co.jp/maps');
  await page.waitForXPath('//*[text()="現在地"]', { visible: true });
  const handle = await page.$x('//*[text()="現在地"]');
  await handle[0].click();
  await page.waitForRequest(req => /.*pwa\/net.js.*/.test(req.url()));
  await page.screenshot({ path: 'shinjuku-android-1.png' });
  await browser.close();
})();
```

続いて同じシナリオをPlaywrightで書くと以下のようになります。

```javascript
const playwright = require('playwright');
const pixel2 = playwright.devices['Pixel 2'];

(async () => {
  const browser = await playwright.chromium.launch();
  const context = await browser.newContext({
    viewport: pixel2.viewport,
    userAgent: pixel2.userAgent,
    geolocation: { latitude: 35.690833, longitude: 139.700278 },
    permissions: { 'https://www.google.co.jp': ['geolocation'] }
  });

  const page = await context.newPage('https://www.google.co.jp/maps');
  await page.click('text="現在地"');
  await page.waitForRequest(/.*pwa\/net.js.*/);
  await page.screenshot({ path: 'shinjuku-android-2.png' });
  await browser.close();
})();
```

以下、順に見ていきます。

### `BrowserContext`クラス

```javascript
// Puppeteer
const pixel2 = puppeteer.devices['Pixel 2'];
await browser.defaultBrowserContext().overridePermissions('https://www.google.co.jp', ['geolocation']);
const page = await browser.newPage();
await page.setViewport(pixel2.viewport);
await page.setUserAgent(pixel2.userAgent);
await page.setGeolocation({ latitude: 35.690833, longitude: 139.700278 });

// Playwright
const pixel2 = playwright.devices['Pixel 2'];
const context = await browser.newContext({
  viewport: pixel2.viewport,
  userAgent: pixel2.userAgent,
  geolocation: { latitude: 35.690833, longitude: 139.700278 },
  permissions: { 'https://www.google.co.jp': ['geolocation'] }
});
```

`devices`は主要なモバイル端末の解像度やUserAgentの情報が入っているオブジェクトです。これは双方にあります。[^2] [^3]
大きく違うのはその後のブラウザの初期設定をするところです。設定ができる項目は同じですが、散らばっていた設定がPlaywrightでは`newContext`メソッドに集約されているのがわかると思います。
Playwrightでは`BrowserContext`クラスが拡張され、エミュレーションなどの設定が`Page`クラスよりも一つ下のレイヤでできるようになっています。Playwrightはこの`BrowserContext`クラスをブラウザ操作の基点として使うようです。

### Selector enginesとDefault wait

```javascript
// Puppeteer
await page.goto('https://www.google.co.jp/maps');
await page.waitForXPath('//*[text()="現在地"]', { visible: true });
const handle = await page.$x('//*[text()="現在地"]');
await handle[0].click();

// Playwright
const page = await context.newPage('https://www.google.co.jp/maps');
await page.click('text="現在地"');
```

PlaywrightではCSS SelectorとXPathに加えて、書式の違う複数のセレクタが使用可能になっており、現在利用可能なセレクタは[Selector engines](https://github.com/microsoft/playwright/blob/v0.10.0/docs/selectors.md)にリストされています。
サンプルではテキストノードの値とマッチさせる`text`セレクタを使用しています。 PuppeteerではCSS SelectorとXPathで別のメソッド（`$`と`$x`）となっており、`click`メソッドではCSS Selectorのみ使用可能でした。Playwrightでは`selector name=selector body`の書式で複数のセレクタに対応するようになっています。
また、Playwrightでは`click`メソッドがデフォルトで対象要素が`visible`になるまで待機するようになっています。`waitForXPath`が無くても相当する機能が`click`に含まれているという感じです。

### `waitForRequest`

```javascript
// Puppeteer
await page.waitForRequest(req => /.*pwa\/net.js.*/.test(req.url()));

// Playwright
await page.waitForRequest(/.*pwa\/net.js.*/);
```

`waitForRequest`は指定したURLのリクエストが完了するまで待機するメソッドです。このメソッドは双方にありますが、小さな変更点としてPlaywrightではURLの正規表現を直接引数に取れるように変更されています。
このメソッドの存在は後述のWebDriverとの大きな違いとなります。

## Selenium WebDriverとの比較

ここからはWebDriverとの比較を行います。

### サポートされているWebブラウザの機能

前述のサンプルコードの通り、Playwrightはgeolocationの設定やモバイル端末のエミュレーション機能などが実装されており、かつこれをブラウザの種類を問わず同じAPIで提供することを目標にしています。
WebDriverはブラウザの基本操作を以外でAPIが定義されているのはCookieの読み書き[^4]くらいです。モバイル端末のエミュレーションは存在していますがChromeDriverの独自の機能となっており統一されたAPIは提供されていません。[^5]

### イベント駆動API

Playwrightは`waitForRequest`の他にもPage上の`load`、`domcontentloaded`、`close`などのイベントをきっかけにする`waitForEvent`などのブラウザ側から起動するAPIが実装されています。これはPlaywrightがブラウザと双方向通信をしているため可能になっています。これらのAPIによってPlaywrightはAjax通信の終了や他のJavaScript処理の終了を確実に検知してからテストスクリプトを実行することができます。
WebDriverはこういったAPIがないため、イベントの完了を検知するには画面上の変化（ボタンの状態、メッセージの表示）をポーリングすることになります。

### クラウド環境での実行

一方でWebDriverは物理的／仮想的に別のマシンにあるブラウザを操作することができます。これはWebDriverが特定のプログラミング言語のライブラリではなく、HTTPプロトコルだから可能になっています。テストスクリプトとブラウザ間の通信がHTTPなので、テストスクリプトとブラウザが同じ環境にある必要がありません。これを利用して[Sauce Labs](https://saucelabs.com/)や[BrowserStack](https://www.browserstack.com/)のようにクラウド環境にブラウザを用意しSelenium WebDriverで接続するサービスが提供されています。
Playwrightは現在このような機能はありませんが、前述の`BrowserContext`クラスを基点にクラウドネイティブに使えるようにする計画があるようです。

## まとめ

Playwrightについて駆け足で追ってきました。まだ見れていない機能がたくさんあるため機会を見て記事を書いていきたいです。
新しいツールが出てくるとこれまでのツールはどうなるんだと考えるのですが、それぞれ影響しあってより洗練されていくのが望ましいですね。

---

[^1]: <https://github.com/microsoft/playwright/blob/v0.10.0/README.md#q-how-does-playwright-relate-to-puppeteer>
[^2]: <https://github.com/microsoft/playwright/blob/v0.10.0/src/deviceDescriptors.ts>
[^3]: <https://github.com/puppeteer/puppeteer/blob/v2.1.0/lib/DeviceDescriptors.js>
[^4]: <https://www.w3.org/TR/webdriver/#cookies>
[^5]: <https://chromedriver.chromium.org/mobile-emulation>