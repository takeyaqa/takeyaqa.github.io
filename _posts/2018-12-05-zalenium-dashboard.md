---
layout: post
title: "Zaleniumのダッシュボードを便利に使う"
date: 2018-12-05 09:00:00 +0900
tags: Selenium Zalenium テスト テスト自動化
---

この記事は [Selenium/Appium Advent Calendar 2018](https://qiita.com/advent-calendar/2018/selenium_and_appium) の5日目の記事です。

この記事ではSelenium Gridの拡張であるZaleniumのダッシュボードについて、ちょっと便利な使い方を解説します。

Zaleniumについての解説はここでは行いません。[本家のサイト](https://opensource.zalando.com/zalenium/)か、CodeZineの記事[DockerでSelenium Gridを構築して複数マシンのブラウザ自動テストを行う](https://codezine.jp/article/detail/10471)をご覧ください。

## Zaleniumのダッシュボード

![zalenium-dashboard]({% link /assets/img/2018/12/05/01.png %})

ダッシュボードは上のようなレイアウトになっています。左側に実行したテストがリスト形式で表示され、それを選択すると右側のエリアに実行時間や環境などの詳細情報と録画した実行中画面が表示されます。

この時、リストに表示されているテストの名称は未設定の場合「63684425-a7e9...」の様なランダムな文字列になります。このままではテストを探すときに不便です。検索しやすいようにテスト名を設定しましょう。

## テスト名を指定する

WebDriverを初期する際のcapabilityに`zal:name`という名前で値を設定すると、この値がダッシュボードにテスト名として表示されます。
Javaでのコード例は以下のとおりです。

```java
ChromeOptions options = new ChromeOptions();
options.setCapability("zal:name", "STAR-HOTEL-E2ETest");
```

## ビルド名を指定する

同様に`zal:build`でビルド名が設定できます。

```java
ChromeOptions options = new ChromeOptions();
options.setCapability("zal:build", "#1");
```

まとめると以下のようなコードになります。

```java
ChromeOptions options = new ChromeOptions();
options.setCapability("zal:name", "STAR-HOTEL-E2ETest");
options.setCapability("zal:build", "#1");
WebDriver driver = new RemoteWebDriver(new URL("http://localhost:4444/wd/hub"), options);
```

## 何を設定するのが良い？

テストをJenkinsで実行している場合は、Jenkinsのジョブ名とビルド番号を渡すと関連付けができて扱いやすくなると思います。
一つのプロジェクトの中に複数のテストがある場合は、ビルド名を統一してテスト名をつけると後述の絞り込みがやりやすくなります。

## 絞り込み

ダッシュボードの「Search」欄へのキーワード入力でリストをフィルタできます。キーワードはスペースで区切って複数入力することでAND検索として扱われます。

また、`http://localhost:4444/dashboard/?q=STAR-HOTEL-E2ETest`のようにパラメータ`q`の後にキーワードを指定するとリンクでフィルタを使うことができます。Jenkinsのジョブ画面にリンクを貼ったりするときに便利です。こちらのキーワードも`+`で結合することで複数指定できます[^1]。ビルドのエラー通知などに載せる場合はすぐにたどれるようにピンポイントでフィルタする方が良いでしょう。

## 最後に

ダッシュボードは基本的な機能ですが、Zaleniumは日本語の記事がほとんど無いため書いてみました。参考になりましたでしょうか？

次回は……きっと誰かが書いてくれるはず……

---

[^1]: アルファベットと一部の記号以外はURLエンコードが必要です。
