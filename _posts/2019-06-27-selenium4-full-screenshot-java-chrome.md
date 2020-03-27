---
layout: post
title: "Selenium 4でページ全体のスクリーンショットを取得する（Java/Google Chrome）"
date: 2019-06-27 09:00:00 +0900
tags: Selenium テスト テスト自動化
---

## Selenium WebDriverでのスクリーンショット

すでに多くの記事があるように、Seleniumでスクリーンショットとして取得できるのはウェブページのうちブラウザのウィンドウの範囲のみとなっています。[^1]

![Qiitaトップページのスクリーンショット（一部分のみ）]({% link /assets/img/2019/06/27/01.png %})

これはSeleniumを使用して取得したスクリーンショットです。ページ下部が含まれていないのがわかると思います。
これまでページ全体のスクリーンショットを得るためには、スクロールしながらスクリーンショットを取得しつなぎ合わせる、Xvfbなどの仮想フレームバッファで縦長のスクリーンを作成するなどの手段がありました。前者はスクロールに追従するメニューバーなどが何度も写ってしまう、後者は使える環境が限られるなどの問題がありました。

この記事ではこれらの問題の解決を目指し、現在アルファ版がリリースされているSelenium 4を使ってページ全体のスクリーンショット取得を試みたいと思います。

## Selenium 4

2019年4月19日にSelenium 4.0.0-alpha-1がリリースされました。[^2] Selenium 4ではいくつかの新機能が予告されていますが、[^3] 今回のリリースではChrome DevTools Protocol (CDP)の呼び出し機能が追加されています。CDPとはChromeの開発者向け機能を呼び出すためのAPIです。SeleniumのChromedriverやGoogleの自動化ツールである[Puppeteer](https://pptr.dev/)も裏側でこのAPIを使用しています。それが今回直接呼び出せるようになったわけです。

さて、前述のPuppeteerにはスクリーンショットをページ全体にするオプションがあります。[^4] したがってPuppeteerと同じ処理を実装すればSeleniumでもページ全体のスクリーンショットが取得できるのではという気がしてきます。これを検証してみましょう。

## コード

```java
@SuppressWarnings("unchecked")
public <X> X getFullPageScreenshotAs(OutputType<X> outputType) {

  // get current layout
  Map<String, Object> layoutMetrics = driver
      .executeCdpCommand("Page.getLayoutMetrics", Collections.emptyMap());

  // override device metrics
  Map<String, Long> contentSize = (Map<String, Long>) layoutMetrics.get("contentSize");
  long width = contentSize.get("width");
  long height = contentSize.get("height");
  driver.executeCdpCommand("Emulation.setDeviceMetricsOverride",
      ImmutableMap.of("mobile", true, "width", width, "height", height, "deviceScaleFactor", 1));

  // capture screenshot
  Map<String, Object> clip = ImmutableMap
      .of("x", 0, "y", 0, "width", width, "height", height, "scale", 1);
  Map<String, Object> result = driver.executeCdpCommand("Page.captureScreenshot", ImmutableMap.of("clip", clip));

  // restore device metrics
  Map<String, Long> visualViewport = (Map<String, Long>) layoutMetrics.get("layoutViewport");
  long clientWidth = visualViewport.get("clientWidth");
  long clientHeight = visualViewport.get("clientHeight");
  driver.executeCdpCommand("Emulation.setDeviceMetricsOverride",
      ImmutableMap.of("mobile", true, "width", clientWidth, "height", clientHeight, "deviceScaleFactor", 1));

  String base64 = (String) result.get("data");

  return outputType.convertFromBase64Png(base64);
}
```
## 実行結果

![qiitaのトップページ]({% link /assets/img/2019/06/27/02.png %})

## 解説

ページ全体のスクリーンショットが取得できていますね🎉

<https://github.com/GoogleChrome/puppeteer/blob/v1.18.0/lib/Page.js#L801-L889>
Puppeteerでスクリーンショットを取得しているコードです。上記のメソッドはこのコードを簡略化してJavaに置き換えたものです。なおインターフェイスは標準の `getScreenshotAs` メソッドにあわせてあります。
`executeCdpCommand` が新しく追加されたメソッドです。一つ目の引数にCDPのコマンドを、二つ目の引数にパラメータを指定します。
実際の処理内容としては`DeviceMertics`の縦横の数値をコンテンツの縦横の数値で一時的に上書きすることで実現しているようです。

## 課題

**実行環境とウェブページの組み合わせによってはうまく動作しないようです。**
ここに貼った画像はmacOSで実行したものです。同じページを対象にWindowsでも実行してみましたが、コンテンツの高さの倍くらいの高さを持った画像になりました。また、Windowsでは他のサイトでも真っ白の画像になる、ページの端が切れてしまうなどの問題が発生しました。macOSでもウェブページによっては期待した通りの結果にならない可能性があります。

インフィニットスクロールのページでは取得タイミングによってスクリーンショットに含まれる範囲が変わります。

また実装ですが、上記のコードでは `DeviceMertics` を書き換える時と戻す時にいくつかの引数を指定しています。このうち `mobile` や `deviceScaleFactor` の値は現在値がAPIで取得できない（と思う）ので、現在の設定を変数などで保持しておく必要があります。
具体的には [mobileEmulation](http://chromedriver.chromium.org/mobile-emulation) などを使っている場合が該当します。今回は `true` と `1` で固定していますが、本来はエミュレーション設定にあわせて指定しなければいけません。

## 最後に

Selenium 4の機能を使うことでページ全体のスクリーンショットが取得できました。
前述のようにPuppeteerでは以前から可能だったことなので、はじめからPuppeteerを使えばいいのでは？という意見もあると思いますし、それはその通りだと思います。しかし、ツールは多様な使用者や環境に対応できるよう複数の選択肢が用意されるべきだと思います。その選択肢が増えたという点で今回のアップデートは歓迎すべきものだと思います。ちょっと余談でした。

以上です。この情報が皆さんのスクリーンショットライフ（？）の一助となれば幸いです。

---

[^1]: これはW3CのWebDriver仕様にも記載されています。<https://w3c.github.io/webdriver/#screen-capture>
[^2]: この日はSeleniumConf Tokyoの開催日でした。<https://conf.selenium.jp/>
[^3]: プロジェクトリードのSimon Stewart氏の解説動画があります。<https://www.youtube.com/watch?v=6iHdvOYdJk8>
[^4]: <https://pptr.dev/#?product=Puppeteer&version=v1.17.0&show=api-pagescreenshotoptions>