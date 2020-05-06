---
title: "Selenium 4新機能: Chrome DevTools Protocol"
date: 2019-12-28 09:00:00 +0900
tags: Selenium テスト テスト自動化 Chrome
---

この記事は[Selenium/Appium Advent Calendar 2019](https://qiita.com/advent-calendar/2019/selenium_and_appium)の14日目の記事です。[^1]

[3日目の記事]({% post_url 2019-12-03-selenium4-new-features-alpha %})ではSelenium 4の新機能をざっと紹介しました。[11日目の記事]({% post_url 2019-12-11-selenium4-relative-locator %})では新機能のひとつであるRelative Locatorについて解説しました。
この記事ではもう一つの新機能であるChrome DevTools Protocol連携について解説したいと思います。

**この記事の内容はalpha版にもとづいています。正式リリースまでに内容が変更となる可能性がありますのでご注意ください。**

## Chrome DevTools Protocolとは

Chrome DevTools Protocol (CDP)とはChromeの開発者向け機能を呼び出すためのAPIです。Chromeにバンドルされているデベロッパーツールの各機能や、SeleniumのChromeDriverやGoogleの自動化ツールであるPuppeteerも裏側でこのAPIを使用しています。

[Chrome DevTools Protocol Viewer](https://chromedevtools.github.io/devtools-protocol/)

CDPの詳細と各APIの解説は上記のサイトにまとまっています。

## `ChromeDriver` クラスのメソッド

ここからはCDPに関連した各メソッドを解説していきます。

### `executeCdpCommand`

`executeCdpCommand`メソッドは`ChromeDriver`から直接呼び出し可能なメソッドです。CDPのコマンドを指定して実行できます。
以下のコードでは [Emulation.setGeolocationOverride](https://chromedevtools.github.io/devtools-protocol/tot/Emulation#method-setGeolocationOverride) メソッドを呼び出して位置情報の偽装を行っています。[^2]

```java
@Test
void testGeolocation() {
  driver.get("https://the-internet.herokuapp.com/geolocation")
  driver.findElement(By.cssSelector("div#content > div > button")).click();
  assertEquals(35, Double.parseDouble(driver.findElement(By.id("lat-value")).getText()), 0.9d);
  assertEquals(139, Double.parseDouble(driver.findElement(By.id("long-value")).getText()), 0.9d);
  driver.executeCdpCommand("Emulation.setGeolocationOverride",
      ImmutableMap.of("latitude", 51.500611d, "longitude", -0.124611d, "accuracy", 100));
  driver.findElement(By.cssSelector("div#content > div > button")).click();
  assertEquals(51.500611d, Double.parseDouble(driver.findElement(By.id("lat-value")).getText()));
  assertEquals(-0.124611d, Double.parseDouble(driver.findElement(By.id("long-value")).getText()));
}
```

### `getDevTools`

`getDevTools`メソッドは`ChromeDriver`から`DevTools`クラスのインスタンスを取得するメソッドです。後述のイベントリスナーを使う場合には`DevTools`クラスが必要になります。

```java
var devTools = driver.getDevTools();
```

## `DevTools` クラスのメソッド

ここからはCDP連携のために新規追加された`DevTools`クラスについて解説します。

### `createSession`, `createSessionIfThereIsNotOne`, `close`

セッションの新規作成および終了をおこなうメソッド群です。なお、セッション管理まわりは現在も調整が入っているようで、APIにも変更がある可能性があります。

### `send`

`send`はCDPのコマンドを実行するメソッドです。`executeCdpCommand`がコマンドを文字列の形式で指定したのに対して、こちらはドメイン毎に定義済みのクラスとメソッドを使います。
以下のコードは[Network.setExtraHTTPHeaders](https://chromedevtools.github.io/devtools-protocol/tot/Network#method-setExtraHTTPHeaders)メソッドを使ってBASIC認証の情報をヘッダーに付加する例です。`enable`および`disable`はネットワークのトラッキングなどの機能を有効・無効にするメソッドです。

```java
@Test
void testBasicAuth() {
  var devTools = driver.getDevTools();
  devTools.createSessionIfThereIsNotOne();
  var userPass = Base64.getMimeEncoder().encodeToString("admin:admin".getBytes());
  devTools.send(Network.setExtraHTTPHeaders(ImmutableMap.of("Authorization", "Basic " + userPass)));
  devTools.send(Network.enable(Optional.empty(), Optional.empty(), Optional.empty()));
  driver.get("https://the-internet.herokuapp.com/basic_auth");
  assertEquals("Basic Auth", driver.findElement(By.tagName("h3")).getText());
  devTools.send(Network.disable());
}
```

### `addListener`

`addListener`はCDPのうち「イベント」に分類されるコマンドを使うためのメソッドです。あるイベントの発生を契機に実行されるメソッドを登録します。
以下のコードは[Network.responseReceived](https://chromedevtools.github.io/devtools-protocol/tot/Network#event-responseReceived)イベントを使ってHTTPのレスポンスが発生した時にステータスコードを出力する例です。

```java
@Test
void testHttpStatusCode() {
  var devTools = driver.getDevTools();
  devTools.createSessionIfThereIsNotOne();
  devTools.addListener(Network.responseReceived(), (responseReceived -> {
    var status = responseReceived.getResponse().getStatus();
    var url = responseReceived.getResponse().getUrl();
    System.out.println("HTTP status: " + url + ": " + status);
  }));
  devTools.send(Network.enable(Optional.empty(), Optional.empty(), Optional.empty()));
  driver.get("https://selenium.dev/");
  devTools.send(Network.disable());
}
```

## まとめ

Selenium 4の新機能であるChrome DevTools Protocol連携について解説しました。私見ですが、まだalpha版ということもあり、各メソッドのインターフェイスも定まってない感が強いです。
なお、この記事のコードを試す際に使ったalpha-3ではCDPのコマンドに対応する各クラス・メソッド（この記事での`Network`クラスなど）は普通にJavaのクラスとして実装されていますが、コマンドの数が多いことやバージョンアップに追従する頻度[^3]が増えることなどを理由にmasterブランチではJSON形式の仕様定義ファイルからコードを自動生成する方針で開発が進められています。
このような事情があるためbetaから正式リリースまでの間にインターフェイスが大きく変わる可能性もあります。その辺りはまたbetaが出るくらいのタイミングで記事を書こうかなを思っています。

以上です。では、春節（何年かはわからない）のリリースを楽しみに待ちましょう！

---

[^1]: 空いていたので後から書いています。
[^2]: ちなみに設定した座標はロンドンの時計塔です。
[^3]: CDPの仕様ははstableが長らく更新されておらず"latest"バージョンを使用することになるため。
