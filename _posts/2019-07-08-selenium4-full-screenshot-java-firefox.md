---
layout: post
title: "Selenium 4でページ全体のスクリーンショットを取得する（Java/Firefox）"
date: 2019-07-08 09:00:00 +0900
tags: Selenium テスト テスト自動化
---

前回 [Selenium 4でページ全体のスクリーンショットを取得する（Java/Google Chrome）]({% post_url 2019-06-27-selenium4-full-screenshot-java-chrome %}) という記事を書きました。

今回は同じくページ全体のスクリーンショットをFirefoxでやってみたいと思います。と言ってもChromeの時より簡単です。

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

![qiitaトップページ]({% link /assets/img/2019/07/08/01_qiita-toppage.png %}){:width="1280px"}

Firefoxでもページ全体のスクリーンショットが取得できていますね🎉

## 課題

横スクロールのページは取得できません。[^1]

また、Chrome同様インフィニットスクロールのページでは取得タイミングによってスクリーンショットに含まれる範囲が変わります。他にスクロールイベントをきっかけに画像が変わるようなページもきれいに取得できない可能性があります。


以上です。この情報が皆さんのスクリーンショットライフ（？）の一助となれば幸いです。

---

[^1]: バグかどうかは確認中 <https://github.com/mozilla/geckodriver/issues/1580>