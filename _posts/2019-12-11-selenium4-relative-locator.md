---
layout: post
title: "Selenium 4新機能: Relative Locator"
date: 2019-12-11 09:00:00 +0900
tags: Selenium テスト テスト自動化
---

この記事は[Selenium/Appium Advent Calendar 2019](https://qiita.com/advent-calendar/2019/selenium_and_appium)の11日目の記事です。

[3日目の記事]({% post_url 2019-12-03-selenium4-new-features-alpha %})ではSelenium 4の新機能をざっと紹介しましたが、この記事ではその中のひとつであるRelative Locatorについてより詳しく解説してみたいと思います。

**この記事の内容はalpha版にもとづいています。正式リリースまでに内容が変更となる可能性がありますのでご注意ください。**

## Relative Locatorとは

alpha-1の発表時には「Friendly Locator」と呼ばれていました。alpha-3でリリースされ、同時により役割がわかりやすいように改名したとのことです。
idやclassを中心とした従来のロケータに代わり、ある要素を基準として、「上」「下」「左」「右」「近く」といった位置による指定で要素を探すことができます。
この機能が実装された背景としては、フロントエンドでのJavaScriptフレームワークが普及し、自動生成されるid属性やDOMツリーがより複雑になり、これらを活用したロケータの使用が困難になりつつあることが挙げられています。

## Relative Locatorのメソッド

ここからはRelative Locatorの各メソッドを解説していきます。なお、現在リリースされているものがバージョン4.0.0-alpha-3のJavaバインディングのみなのでここではJavaを使って解説します。

![empty]({% link /assets/img/2019/12/11/01_empty.png %}){:width="166px"}

また、説明をやり易くするために上記の画像の形をしたtable要素を例示に使います。各セル（td要素）にテキストと同じ数字（の英単語[^1]）がid属性として付与してあるものとします。

### `withTagName`

`RelativeLocator.withTagName(String tagName)`はRelative Locatorの起点となるStaticメソッドです。**取得したい要素のHTMLタグ名**をここで指定します。このメソッドの戻り値は`RelativeBy`型となっており、ここから後ほど解説する位置指定のメソッドを呼び出します。
取得したい要素が`td`の場合は次のようなコードになります。

```java
driver.findElements(withTagName("td").above(...));
```

### `above`

![above]({% link /assets/img/2019/12/11/02_above.png %}){:width="530px"}

`above(By locator)`, `above(WebElement element)`は指定した要素よりも上の位置を検索対象とするメソッドです。`By`クラスのロケータを引数とするものと取得済みの`WebElement`を引数とするものの二つがあります（他の位置指定メソッドも同様です）。
例示のテーブルで「8」のセルから上にある要素を取得したい場合は次のようなコードになります。

```java
var elements = driver.findElements(withTagName("td").above(By.id("eight")));
assertEquals(6, elements.size());
```

### `below`

![below]({% link /assets/img/2019/12/11/03_below.png %}){:width="528px"}

`below(By locator)`, `below(WebElement element)`は指定した要素よりも下の位置を検索対象とするメソッドです。
例示のテーブルで「2」のセルから下にある要素を取得したい場合は次のようなコードになります。

```java
var elements = driver.findElements(withTagName("td").below(By.id("two")));
assertEquals(6, elements.size());
```

### `toLeftOf`

![leftOf]({% link /assets/img/2019/12/11/04_leftOf.png %}){:width="532px"}

`toLeftOf(By locator)`, `toLeftOf(WebElement element)`は指定した要素よりも左の位置を検索対象とするメソッドです。
例示のテーブルで「6」のセルから左にある要素を取得したい場合は次のようなコードになります。

```java
var elements = driver.findElements(withTagName("td").toLeftOf(By.id("six")));
assertEquals(6, elements.size());
```

### `toRightOf`

![rightOf]({% link /assets/img/2019/12/11/05_rightOf.png %}){:width="528px"}

`toRightOf(By locator)`, `toRightOf(WebElement element)`は指定した要素よりも右の位置を検索対象とするメソッドです。
例示のテーブルで「4」のセルから右にある要素を取得したい場合は次のようなコードになります。

```java
var elements = driver.findElements(withTagName("td").toRightOf(By.id("four")));
assertEquals(6, elements.size());
```

### `near`

![near]({% link /assets/img/2019/12/11/06_near.png %}){:width="165px"}

`near(By locator, int atMostDistanceInPixels)`, `near(WebElement element, int atMostDistanceInPixels)`, `near(By locator)`, `near(WebElement element)`は指定した要素の近くにある要素を検索対象とするメソッドです。 「近く」の距離は二つ目の引数で指定するピクセル数で決まります。指定がない場合のデフォルト値は50ピクセルです。
例示のテーブルで「6」のセルの近く(50px)にある要素を取得したい場合は次のようなコードになります。

```java
var elements = driver.findElements(withTagName("td").near(By.id("six"), 50));
assertEquals(5, elements.size());
```

なおこの`near`メソッドですが、特定の場合に隣接しているはずの要素が取得できない不具合がみつかっています。[^2] そのためこの後の使い方の例では使用を控えることにします。

## Relative Locatorの使い方

### 単数と複数

他のロケータと同様に`findElements`を使用した場合は条件にマッチしたもの全てが`List<WebElement>`型で取得できます。`findElement`を使用した場合はマッチした要素リストの先頭の要素が`WebElement`型で取得できます。

### 条件の組み合わせ

検索対象はページ全体に及ぶため、一つの位置条件だけでは検索対象が広くなり過ぎます。検索対象をより限定するために位置条件のメソッドを複数組み合わせて使うことができます。
例として右上の「3」のセルを指定したい場合、`above`と`toRightOf`を組み合わせて以下のようなコードを書くことができます。

```java
var elements = driver.findElements(withTagName("td").above(By.id("five")).toRightOf(By.id("two")));
assertEquals(1, elements.size());
assertEquals("3", elements.get(0).getText());
```

### 実践例

ここまでの解説を踏まえた上で、[日本Seleniumユーザーコミュニティのテスト用サイト](http://example.selenium.jp/reserveApp_Renewal/)を使って実践的な例を書いてみたいと思います。

#### 例1

予約フォームの最下部に「利用規約に同意して次へ」のボタンと「同意しない」のボタンが並んでいます。このサイトでは両方にid属性が付与してありますが、一部のボタンにidがないページはよく見受けられます。仮に「同意しない」のボタンにidがなかった場合は以下のようなコードを書けばボタンの取得が可能です。

```java
var disagreeButton = driver.findElement(withTagName("button").toRightOf(By.id("agree_and_goto_next")));
assertEquals("同意しない", disagreeButton.getText());
disagreeButton.click();
```

#### 例2

このページには`h3`でマークアップされた見出しが複数あります。idやclassは付与されていないため、リスト内の順番でしか特定ができません。このような要素も位置関係から特定することができます。

```java
var guestNameCaption = driver.findElement(withTagName("h3").below(By.id("plan_a")).above(By.id("guestname")));
assertEquals("お名前", guestNameCaption.getText());
```

## まとめ

Selenium 4の新機能であるRelative Locatorについて解説しました。
現時点ではalpha版であり、不具合が潜在している可能性が高いこと、仕様に不明確な点（重なっている要素の扱いなど）があることなどから実務で使うことはオススメできませんが、将来的にはロケータの新たな選択肢として有用だと考えています。

それでは、春節（何年かはわからない）のリリースを楽しみに待つことにしましょう。

## 参考記事

* [SELENIUM 4 RELATIVE LOCATORS](https://angiejones.tech/selenium-4-relative-locators/)

---

[^1]: id属性は数字はじまりにできないため
[^2]: https://github.com/SeleniumHQ/selenium/issues/7851