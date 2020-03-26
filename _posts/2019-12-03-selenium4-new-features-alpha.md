---
layout: post
title: "Selenium 4新機能まとめ (alpha)"
date: 2019-12-03 09:00:00 +0900
tags: Selenium テスト テスト自動化
---

この記事は[Selenium/Appium Advent Calendar 2019](https://qiita.com/advent-calendar/2019/selenium_and_appium)の3日目の記事です。

次バージョンのSeleniumであるSelenium 4は今年4月のSeleniumConf Tokyoでalpha-1が発表＆リリースされました。その後7月にalpha-2が、Selenium Conf London直前となる9月にalpha-3がリリースされました。現在はこのalpha-3が最新版となっています。

この記事ではSelenium WebDriverのJavaバインディングを中心にSelenium 4での新機能や変更点をまとめたいと思います。

**この記事の内容はalpha版にもとづいています。正式リリースまでに内容が変更となる可能性がありますのでご注意ください**

## W3C WebDriverプロトコルサポート

Selenium 4はW3CのWebDriverプロトコルのみの対応となる予定です。現時点ですでに各ブラウザのdriverはW3Cに対応しており、バックエンドでの通信はW3Cで行うことが可能となっています。また、現在のPublicなAPIが何か変わるわけでなく、そのままアップデートが可能です。

## Relative Locators

alpha-1の発表時には「**Friendly Locators**」と呼ばれていました。alpha-3でリリースされ、同時により実態がわかりやすいように改名したとのことです。
idやclassを中心とした従来のlocatorに代わり、ある要素を基準として、「上」「下」「左」「右」「近く」といった位置による指定で要素を探すことができます。

詳しくはこちらの記事をご覧ください。
[Selenium 4新機能: Relative Locator]({% post_url 2019-12-11-selenium4-relative-locator %})

## Chrome Debugging Protocolサポート

[Chrome Debugging Protocol](https://chromedevtools.github.io/devtools-protocol/)のAPIを直接呼び出すことができます。フルページスクリーンショット、HTTPログの取得などの操作ができるようになります。

詳しくはこちらの記事をご覧ください。
[Selenium 4新機能: Chrome DevTools Protocol]({% post_url 2019-12-28-selenium4-chrome-devtools-protocol %})

## まとめ

Selenium 4の新機能を駆け足で紹介しました。春節（何年かはわからない）のリリースを楽しみに待ちましょう！

### 参考ページ

* [State of the Union - Simon Stewart \| SeleniumConf Tokyo](https://www.youtube.com/watch?v=NtEZ2aBszrc)
* [[Webinar] Selenium 4 with Simon Stewart and BrowserStack](https://www.browserstack.com/blog/webinar-selenium-4-with-simon-stewart/)
* [State of the Union \| Simon Stewart \| #SeConfLondon](https://www.youtube.com/watch?v=RGM4FtDA06M)