---
layout: post
title: "SeleniumとBrowserMob ProxyでBASIC認証のページをテストする"
date: 2019-07-16 09:00:00 +0900
tags: Selenium テスト テスト自動化
---

この記事ではBASIC認証付きのページをSeleniumでテストする方法を紹介します。

## テストケース例

例として作成したテストケースです。テスト対象のサイトには[The Internet](https://the-internet.herokuapp.com/)のBASIC認証のテストページを使います。

```java
private WebDriver driver;

@BeforeEach
void before() {
  ChromeOptions options = new ChromeOptions();
  driver = new ChromeDriver(options);
}

@AfterEach
void after() {
  driver.quit();
}

@Test
void testBasicAuthPage() {
  driver.get("https://the-internet.herokuapp.com/basic_auth");
  assertEquals("Basic Auth", driver.findElement(By.tagName("h3")).getText());
}
```

このテストケースはページのアクセス時に認証ダイアログがでてストップしてしまいます。このダイアログはSeleniumから操作することができません。

## BrowserMob Proxy

BASIC認証への対応方法はとしてはURLにユーザ名とパスワードを埋め込む方法が知られています（`https://admin:admin@the-internet.herokuapp.com/`のような形式）が、ここでは別の手段としてプロキシーを使ってみたいと思います。

使用するのは[BrowserMob Proxy](https://github.com/lightbody/browsermob-proxy)です。これはJavaで書かれたProxyアプリケーションで、スタンドアロンで動作する他にライブラリとしてプログラム上から使用することが可能になっています。[^1]

## Seleniumへの組み込み

まず依存関係にライブラリを追加します。ここではGradleの例を記載します。

```gradle
dependencies {
  ...
  implementation group: 'net.lightbody.bmp', name: 'browsermob-core', version: '2.1.5'
  ...
}
```

つづいてWebDriverのセットアップ部分を以下のように書き換えます。

```java
private WebDriver driver;
private BrowserMobProxy proxy;

@BeforeEach
void before() {
  proxy = new BrowserMobProxyServer();
  proxy.start();
  ChromeOptions options = new ChromeOptions();
  Proxy seleniumProxy = ClientUtil.createSeleniumProxy(proxy);
  options.setProxy(seleniumProxy);
  driver = new ChromeDriver(options);
}

@AfterEach
void after() {
  driver.quit();  
  proxy.stop();
}
```

コンストラクタ `new BrowserMobProxyServer()` で初期化して、`ClientUtil.createSeleniumProxy()` でSelenium用の設定に変換します。ここではChromeの例ですが、Firefoxでも同じ手順で利用が可能です。

次にテスト本体を以下のように書き換えます。

```java
@Test
void testBasicAuthPage() {
  proxy.autoAuthorization("the-internet.herokuapp.com", "admin", "admin", AuthType.BASIC);
  driver.get("https://the-internet.herokuapp.com/basic_auth");
  assertEquals("Basic Auth", driver.findElement(By.tagName("h3")).getText());
  proxy.stopAutoAuthorization("the-internet.herokuapp.com");
}
```

`autoAuthorization()` がプロキシーでの認証を設定するメソッドです。ここに認証するドメイン名とユーザ名・パスワード、認証方式を設定します。

以上です。このテストを実行するとダイアログが表示されずに認証がかかったページを開くことができます🎉

---

[^1]: 『[Selenium実践入門](https://gihyo.jp/book/2016/978-4-7741-7894-3)』でもHTTPステータスコードの取得での利用が解説されています