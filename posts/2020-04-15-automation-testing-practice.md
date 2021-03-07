+++ 
draft = false
date = 2020-04-15T09:00:00+09:00
title = "テスト自動化学習向けのデモサイトを作成しました"
description = ""
slug = "automation-testing-practice" 
tags = ["Selenium", "testing", "automation", "WebdriverIO", "beginners"]
categories = []
externalLink = ""
series = []
isCJKLanguage = true
images = ["/images/2020/04/15/01_hotel-planisphere.png"]
aliases = ["/2020/04/15/automation-testing-practice.html"]
+++

テスト自動化[^1]を学習する人向けに、テスト対象として使えるデモサイトを作成しました！　サンプルコードも作成しているのであわせてお使いください！

<!--more-->

## サイト

<https://hotel.testplanisphere.dev/>

先日作成中のものを公開したときとはURLが変わっているのでご注意ください。

## サンプルコード

<https://github.com/testplanisphere/hotel-example-selenium3-java>
<https://github.com/testplanisphere/hotel-example-webdriverio>

## サイトのイメージ

![サイトの画面イメージ](/images/2020/04/15/01_hotel-planisphere.png)

## 解説

サイトは以前のテスト自動化カンファレンスハンズオンで使われたホテルの予約サイト（STAR HOTEL）をベースにしています。
このサイトは『Selenium実践入門』など書籍のコード例でも使われていました。

<http://example.selenium.jp/reserveApp_Renewal/>
<https://github.com/SoftwareTestAutomationResearch/STARHOTEL-Teaching-Materials>

変更点としては

* サイトの外観をアップデート、モバイルにも対応
* 会員登録、ログインなどのデータが永続する機能を追加（Local Storageでクライアント側に保存）
* HTML5で追加されたinput要素（date, rangeなど）をはじめ各種のinputが一通り出現するように
* Ajax、iframe、新規ウィンドウ、ダイアログなどツールの使い方を覚えるときに必要になる要素の追加

と行った感じです。詳しくは[こちらのページ](https://hotel.testplanisphere.dev/about.html)をご覧ください。

サンプルコードはJavaとJavaScriptの二つを作成しました。Javaの方は素のSeleniumしか使っていないのでやや冗長な書き方になっています。
また、Github Actionsでテスト実行できるようにしています。（Javaは2ブラウザ3OSの6環境、JavaScriptは1環境）

まとめると以下のリソースが現在揃っている状態です。

* テスト対象のウェブサイト
* サンプルコード（2言語）
* CI環境（Github Actions）

リポジトリはOpenになっています。サイトやサンプルコードを見ていて気になったところはissuesで報告をいただけるとありがたいです。

以上となります。このプロジェクトが自動化をはじめる最初の一歩の助けになれば幸いです。

---

[^1]: ここではブラウザを通したE2Eテストを指します
