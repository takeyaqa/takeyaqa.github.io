+++ 
draft = false
date = 2020-08-02T23:00:00+09:00
title = "テスト自動化練習サイトの英語版をリリースしました"
description = ""
slug = "automation-testing-practice-en-us" 
tags = ["Selenium", "testing", "automation", "WebdriverIO", "Selenide", "Capybara", "beginners"]
categories = []
externalLink = ""
series = []
isCJKLanguage = true
images = ["/images/2020/08/02/01_hotel-planisphere-en.png"]
+++

[以前の記事](/2020/04/15/automation-testing-practice/)で紹介したテスト自動化[^1]の練習用サイト「[HOTEL PLANISPHERE](https://hotel.testplanisphere.dev/)」ですが、このたび英語版をリリースしました。

この記事では英語版のリリースに伴う変更点とサイトの紹介を改めて行いたいと思います。

<!--more-->

## 日本語版サイトの変更点について

アドレスが変更になりました。

<https://hotel.testplanisphere.dev/ja/>

これまでの `/` 直下のページについて現在は利用可能ですが、8月末の更新で完全に移行する予定です。お手数ですがブックマーク等の変更をお願いいたします。
また、アップデートとして以下の点が変更になっています。

* サイト名が「テスト自動化デモサイト」から「テスト自動化練習サイト」に変更になりました。
* 宿泊予約ページの「予約内容を確認する」ボタンをプラン情報の読み込みが完了するまで非活性状態にするように変更しました。
* プラン一覧、宿泊予約ページのAjax通信がFetch APIからXMLHttpRequestに変更になりました。
* 宿泊予約ページのDOM構造が変更になりました。form要素の位置がdivの外側になっています。

サイト名の変更はこのサイトの目的が明確になるように、ボタンの非活性化はAjax通信完了のwaitをやりやすくするための変更です。
残りの二つはChrome以外のブラウザへの対応のために変更しました。メインの環境としてはChromeで動作確認をしていますが、今回の更新で以下のブラウザでも動作するようになりました。

* Google Chrome
* Microsoft Edge (Chromium based)
* Mozilla Firefox
* Apple Safari
* Internet Explorer 11

ただし、一部の機能がブラウザの実装に依存しているため動作が異なる箇所があります。把握できている箇所は以下の通りです。

* HTML5で追加されたinput要素（date, colorなど）の非対応（Safari、IE11）
* 通貨フォーマットの実装による表示の差異（IE11）

また、Seleniumを使用した自動テストの動作確認はChromeで行っています。他のブラウザはdriverの不安定さにより今のところ正式に「絶対に動作します」とは言えない状態です。あしからず。[^2]

## 英語（アメリカ）版サイトについて

![英語版サイトの画面イメージ](/images/2020/08/02/01_hotel-planisphere-en.png)

ご覧の通り日本語版からUIの言語を英語へ変更し、日付および通貨をアメリカ合衆国のフォーマットに修正したサイトです。したがってURLは `/en-US/` となります。

Twitterにも書きましたが、トップページの注意文などの翻訳についてはテスト自動化研究会の[@ito_nozomi](https://twitter.com/ito_nozomi)さん、[@Kazu_cocoa](https://twitter.com/Kazu_cocoa)さんにサポートいただきました。ありがとうございました。

予約機能などのサイト本体の表現は実際のホテル・ホテル予約サイトなどを参考にしました。あと[DeepL](https://www.deepl.com/home)大先生。翻訳のクオリティについてはまだまだ不十分だと思うので、誤訳などに気付いた方は[GitHubのissues](https://github.com/testplanisphere/hotel-example-site/issues)まで報告いただくか私のTwitterに声をかけていただけると助かります。

## サイトについて（改めて）

このサイトは以下のような方のために作成した**テスト対象サイト**です。

* 自動テストの学習をしている人
* 社内、セミナーなど自動テストの研修を企画する人
* 自動テストについてのブログ記事や書籍を書く人
* 新しいテスト用ツールを試したい人

ログイン情報、会員登録の情報はブラウザのCookieおよびSession Storage、Local Storageに保存されます。そのため、自分専用のサーバを立ち上げたりする必要はなく、他のユーザの利用を気にしたりせずに使うことができます。

### サンプルコードについて

学習の参考になるようにサンプルコードを用意しています。言語と自動化フレームワークが異なるプロジェクトが現在4種類あります。

* [言語/Java、フレームワーク/Selenide](https://github.com/testplanisphere/hotel-example-selenide-ja)
* [言語/JavaScript、フレームワーク/WebdriverIO](https://github.com/testplanisphere/hotel-example-webdriverio-ja)
* [言語/Ruby、フレームワーク/Capybara, SitePrism](https://github.com/testplanisphere/hotel-example-capybara-ja)
* [言語/Java、フレームワーク/Selenium3](https://github.com/testplanisphere/hotel-example-selenium3-java-ja)

いずれも（ほぼ）同じテストシナリオを別言語・フレームワークで実装したものです。また、Github Actionsで動作するCIの設定ファイルも含まれています。

### サイト名について

![星座早見盤](/images/2020/08/02/02_planisphere.png)

サイト名の "Planisphere" （プラニスフィア）は「星座早見盤」という意味です。元となったサイトが「[STAR HOTEL](https://github.com/SoftwareTestAutomationResearch/STARHOTEL-Teaching-Materials)」なので天体関連の用語を、なおかつあんまり使われてなさそうで響きの良さそうな感じの単語をということで選択しました。なにより「（たぶん）ビギナーが多く使う」「広い夜空（＝ソフトウェアテスト）での道しるべ」という点がサイトのコンセプトに合っているというのが決定打でした。

## さいごに

最近は[ツールの比較](https://www.slideshare.net/MaiKaneko4/selenium-webdrivercypresstestcafe)にも使っていただけたようで、作者としてはありがたい限りです。[^3] このサイトはWebのいろいろな要素をできるだけ網羅するように配置しています。これは学習はもちろんですが、このように複数のツールを同じ条件で比較できるようにするためでもあります。私自身も業務でツールの比較検証をすることが多く、こういうサイトがあったら便利だよなと考えていたのが反映されたものです。

現在までに作成したリソースはすべてOSSとして以下のプロジェクト "Test Planisphere" で公開しています。

<https://github.com/testplanisphere>

今のところメンバーを増やす予定はありませんが（私が管理とかできないので）、Pull Requestやissueの報告はいつでも受け付けています。サイトやサンプルコードを見ていて気になったことがあれば遠慮なく指摘いただければと思います。

以上です。改めて、このプロジェクトがみなさんの「星座」を探す助けになれば幸いです。

## おまけ

![プラニスフィアロボ](/images/2020/08/02/03_planisphere_robot.png)

その名はプラニスフィアロボ。ソフトウェアテストの道を歩む者をやさしく見守る正体不明の謎のロボットである。ペイントとの1時間に及ぶ格闘の末に誕生した。透過の仕方がわからないので背景はいつも白い。

---

[^1]: ここではブラウザを通したE2Eテストを指します。
[^2]: 案の定IEDriverが不安定すぎて早々に断念しました。他はもうちょっとがんばりたい。
[^3]: 別件があり発表は直接聞けませんでした。かなしい。
