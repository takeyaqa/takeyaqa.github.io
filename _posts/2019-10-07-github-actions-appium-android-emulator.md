---
title: "Github Actions+AppiumでAndroid Emulatorを動かす"
date: 2019-10-07 09:00:00 +0900
tags: Selenium Android テスト テスト自動化 appium GitHubActions
---

[前回の記事]({% post_url 2019-09-25-github-actions-appium-iphone-simulator %})でGithub Actionsの環境でiOS Simulatorを動かす方法を書きました。今回はAndroid Emulatorを動かしてみたいと思います。

## ワークフロー定義

```yaml
name: Android Emulator Test

on: [push, pull_request]

jobs:
  build:
    name: Mobile Chrome Test
    runs-on: macOS-10.14
    steps:
      - name: Checkout source code
        uses: actions/checkout@v1
      - name: Set up Node.js
        uses: actions/setup-node@v1
        with:
          node-version: '10.16.3'
      - name: Run Android Emulator
        run: |
          echo "y" | $ANDROID_HOME/tools/bin/sdkmanager --install 'system-images;android-27;google_apis;x86'
          echo "no" | $ANDROID_HOME/tools/bin/avdmanager create avd -n xamarin_android_emulator -k 'system-images;android-27;google_apis;x86' --force
          echo $ANDROID_HOME/emulator/emulator -list-avds
          echo "Starting emulator"
          nohup $ANDROID_HOME/emulator/emulator -avd xamarin_android_emulator -no-snapshot > /dev/null 2>&1 &
          $ANDROID_HOME/platform-tools/adb wait-for-device shell 'while [[ -z $(getprop sys.boot_completed | tr -d '\r') ]]; do sleep 1; done; input keyevent 82'
          $ANDROID_HOME/platform-tools/adb devices
          echo "Emulator started"
      - name: Set up Appium
        run: npm install appium@1.15.0
      - name: Run Appium Server
        run: ./node_modules/.bin/appium --log-timestamp --log-no-colors --allow-insecure chromedriver_autodownload > appium.log &
      - name: Build with Gradle
        run: gradle cleanTest test --tests "com.example.chrome.MobileChromeTest"
        continue-on-error: true
      - name: Upload logs
        uses: actions/upload-artifact@v1
        with:
          name: appium.log
          path: appium.log
      - name: Upload screenshots
        uses: actions/upload-artifact@v1
        with:
          name: screenshots
          path: screenshots
```

順に解説していきます。

### コードのチェックアウトとNode.jsのセットアップ

```yaml
- name: Checkout source code
  uses: actions/checkout@v1
- name: Set up Node.js
  uses: actions/setup-node@v1
  with:
    node-version: '10.16.3'
```

表題の通りです。どちらもactionsを活用しています。

### Android Emulatorのインストールと起動

```yaml

- name: Run Android Emulator
  run: |
    echo "y" | $ANDROID_HOME/tools/bin/sdkmanager --install 'system-images;android-27;google_apis;x86'
    echo "no" | $ANDROID_HOME/tools/bin/avdmanager create avd -n xamarin_android_emulator -k 'system-images;android-27;google_apis;x86' --force
    echo $ANDROID_HOME/emulator/emulator -list-avds
    echo "Starting emulator"
    nohup $ANDROID_HOME/emulator/emulator -avd xamarin_android_emulator -no-snapshot > /dev/null 2>&1 &
    $ANDROID_HOME/platform-tools/adb wait-for-device shell 'while [[ -z $(getprop sys.boot_completed | tr -d '\r') ]]; do sleep 1; done; input keyevent 82'
    $ANDROID_HOME/platform-tools/adb devices
    echo "Emulator started"
```

ここのコードは [Azure DevOpsのヘルプドキュメント](https://docs.microsoft.com/en-us/azure/devops/pipelines/ecosystems/android?view=azure-devops#test-on-the-android-emulator)から拝借しています。

内容としてはsystem-imagesをダウンロードするところから始まり、AVDの作成、起動となっています。毎回ダウンロードが走る事に関してはビルドキャッシュが導入されるまでは我慢するしかないですね……

### Appiumのインストールと起動

```yaml
- name: Set up Appium
  run: npm install appium@1.15.0
- name: Run Appium Server
  run: ./node_modules/.bin/appium --log-timestamp --log-no-colors --allow-insecure chromedriver_autodownload > appium.log &
```

NPMでAppiumをインストールした後に起動しています。起動時にログが見やすくなるようにオプションを設定しているほか、ChromeDriverの自動ダウンロードを有効にしています。これはEmulatorのChromeのバージョンとAppiumにバンドルされているChromeDriverのバージョンが一致していないため必要になります。

### テストの実行

```yaml
- name: Build with Gradle
  run: gradle cleanTest test --tests "com.example.chrome.MobileChromeTest"
  continue-on-error: true
```

GradleでJavaのテストを実行します。`continue-on-error` を有効にしているのは、このステップで失敗するとワークフローの実行が中断され、この後のAppiumログのアップロードが実行されないからです。

Javaのテストコードは以下の通りです。

```java
class MobileChromeTest {
  private AndroidDriver<WebElement> driver;

  @BeforeEach
  void before() {
    DesiredCapabilities caps = new DesiredCapabilities();
    caps.setBrowserName(BrowserType.CHROME);
    caps.setCapability(MobileCapabilityType.AUTOMATION_NAME, AutomationName.ANDROID_UIAUTOMATOR2);
    caps.setCapability(MobileCapabilityType.DEVICE_NAME, "Android Emulator");
    caps.setCapability(MobileCapabilityType.LANGUAGE, "ja");
    caps.setCapability(MobileCapabilityType.LOCALE, "JP");
    caps.setCapability(AndroidMobileCapabilityType.NATIVE_WEB_SCREENSHOT, true);
    caps.setCapability("adbExecTimeout", 60000);
    caps.setCapability("uiautomator2ServerInstallTimeout", 60000);
    try {
      driver = new AndroidDriver<>(new URL("http://0.0.0.0:4723/wd/hub"), caps);
    } catch (MalformedURLException ign) {
      // ignore
    }
  }

  @AfterEach
  void after() {
    if (driver != null) {
      driver.quit();
    }
  }

  @Test
  void testGet() {
    driver.get("https://qiita.com/");
    assertEquals("Qiita", driver.getTitle());
    screenshot();

    driver.get("https://qiita.com/search");
    driver.findElement(By.id("q")).sendKeys("Selenium");
    screenshot();
    driver.findElement(By.className("searchResultContainer_searchButton")).click();
    (new WebDriverWait(driver, 10)).until(ExpectedConditions.titleContains("Selenium"));
    assertEquals("「Selenium」の検索結果 - Qiita", driver.getTitle());
    screenshot();
  }
}
```

Emulatorの動作が遅いのでタイムアウト系の設定を長めに変更しています。テスト本体部分はQiitaの検索ページで検索をしているだけですね。

### ログとスクリーンショットのアップロード

```yaml
- name: Upload logs
  uses: actions/upload-artifact@v1
  with:
    name: appium.log
    path: appium.log
- name: Upload screenshots
  uses: actions/upload-artifact@v1
  with:
    name: screenshots
    path: screenshots
```

actionsを使ってログファイルとスクリーンショットのファイルをアップロードしています。

## まとめ

動作した結果がこちらです。 <https://github.com/takeya0x86/spike-github-actions/runs/249721165>

キャッシュがうまく効くようになれば運用しやすくなるかなと思います。このあたりはGA待って再度試すのが良さそうですね。

Android Emulatorに関してはアプリのテストも実行可能です。iOS SimulatorはSimulator向けビルドなら動作するという感じですね。

Selenium、AppiumもGithub Actionsでも動作させる事はできるので、今後はこれを活用したワークフローをどう作っていくかを考えていきたいと思います。