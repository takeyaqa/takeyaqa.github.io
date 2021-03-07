+++ 
draft = false
date = 2019-07-08T09:00:00+09:00
title = "Selenium 4でページ全体のスクリーンショットを取得する（Java/Firefox）"
description = ""
slug = "selenium4-full-screenshot-java-firefox" 
tags = ["Selenium", "Selenium4", "testing", "automation", "Firefox", "screenshot"]
categories = []
externalLink = ""
series = []
isCJKLanguage = true
aliases = ["/2019/07/08/selenium4-full-screenshot-java-firefox.html"]
+++

前回 [Selenium 4でページ全体のスクリーンショットを取得する（Java/Google Chrome）](/2019/06/27/selenium4-full-screenshot-java-chrome/) という記事を書きました。

今回は同じくページ全体のスクリーンショットをFirefoxでやってみたいと思います。と言ってもChromeの時より簡単です。

<!--more-->

## Selenium 4-alpha-2 and GeckoDriver 0.24.0

GeckoDriverには[バージョン0.24.0](https://github.com/mozilla/geckodriver/releases/tag/v0.24.0)からページ全体のスクリーンショットを取得するためのエンドポイントが追加されています。そして、2019年7月1日リリースのSelenium 4.0.0-alpha-2にはこのエンドポイントを呼び出すメソッドである `getFullPageScreenshotAs` が追加されています。したがって今回はこのメソッドを呼ぶだけで目的が達成できます👍

## コード

```java
public void testFullPageScreenshot() throws IOException {
  FirefoxDriver driver = new FirefoxDriver();
  driver.get("https://qiita.com/");
  Path screenshot = driver.getFullPageScreenshotAs(OutputType.FILE).toPath();
  Files.copy(screenshot, Paths.get("screenshot.png"));
}
```
## 実行結果

![qiitaトップページ](/images/2019/07/08/01_qiita-toppage.png)

Firefoxでもページ全体のスクリーンショットが取得できていますね🎉

## 課題

横スクロールのページは取得できません。[^1]

また、Chrome同様インフィニットスクロールのページでは取得タイミングによってスクリーンショットに含まれる範囲が変わります。他にスクロールイベントをきっかけに画像が変わるようなページもきれいに取得できない可能性があります。


以上です。この情報が皆さんのスクリーンショットライフ（？）の一助となれば幸いです。

---

[^1]: バグかどうかは確認中 <https://github.com/mozilla/geckodriver/issues/1580>
