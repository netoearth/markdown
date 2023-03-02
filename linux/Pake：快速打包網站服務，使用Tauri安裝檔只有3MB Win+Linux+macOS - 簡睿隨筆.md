Pake是跨平台的打包專用的命令行應用，有Windows、Linux與macOS版，開源於GitHub。

Pake類似常見的Electron應用，但是使用的是Rust語言的Tauri框架，與Electron相比，安裝檔大小大幅降低了40倍。只要安裝了nodejs與rust的開發環境，就能很容易的建立需要的網站服務。

本次介紹Windows的安裝與產生需要網路服務的執行步驟。

## 1\. 安裝

由說明區提供的網址下載兩個安裝檔：

### 1.1. 安裝底層框架

1.  [nodejs](https://nodejs.org/en/download/current/)下載與安裝
2.  [Rust開發環境](https://www.rust-lang.org/zh-TW/tools/install)下載rust-init.exe並安裝，執行時會自動安裝Visual Studio Build Tools

安裝後確認PATH環境變數有指到nodejs安裝資料夾與rust的執行檔資料夾(`C:\Users\emisj\.cargo\bin\`)。

以命令行指令驗證安裝狀態，用`rustup update`更新Rust開發環境：

```
node -v
npm -v
rustup update
```

### 1.2. 安裝pake

用`npm install`安裝pake工具。

```
npm install -g pake-cli
```

安裝後驗證版本：

```
pake -V
```

pake完整說明：

```
Usage: cli [options] [url]

A cli application can package a web page to desktop application.

Arguments:
  url                the web url you want to package

Options:
  -V, --version      output the version number
  --name <string>    application name
  --icon <string>    application icon (default: "")
  --height <number>  window height (default: 780)
  --width <number>   window width (default: 1200)
  --no-resizable     whether the window can be resizable
  --fullscreen       makes the packaged app start in full screen (default: false)
  --transparent      transparent title bar (default: false)
  --debug            debug (default: false)
  -h, --help         display help for command
```

### 1.3. 設定路徑

```
set PATH=%PATH:C:\Program Files\nodejs=%
set PATH=%PATH:C:\Users\emisj\.cargo\bin\=%
set PATH=%PATH:C:\Users\emisj\AppData\Roaming\npm\=%
set PATH=C:\Program Files\nodejs;C:\Users\emisj\.cargo\bin\;C:\Users\emisj\AppData\Roaming\npm\;%PATH%

```

## 2\. 打包執行步驟

> \[!REF\] 命令格式  
> pake 網址 –name 產生的MSI安裝檔名 \[–icon 圖示檔名\]

範例：

```
pake https://app.diagrams.net --name DrawIO --icon drawio-logo.ico
pake https://keep.google.com/u/0/ --name Keep --transparent
```

## 3\. 相關鏈接

## 4\. 教學影片

<iframe src="https://www.youtube.com/embed/G1a5ie9sGBk" width="650" height="315" frameborder="0" allowfullscreen="allowfullscreen"></iframe>

＃＃

#### 您可能也會有興趣的類似文章

-   [fselect: 擺脫複雜的命令選項，用SQL語法搜尋檔案](https://jdev.tw/blog/6469/fselect-find-files-by-sql "fselect: 擺脫複雜的命令選項，用SQL語法搜尋檔案") (0則留言, 2020/11/15)
-   [在命令提示字元取得目前資料夾路徑的方法—使用set /p｜初學者的命令行＃10](https://jdev.tw/blog/6636/command-line-get-folder-path-into-clipboard "在命令提示字元取得目前資料夾路徑的方法—使用set /p｜初學者的命令行＃10") (0則留言, 2021/04/22)
-   [\[Obs#13\] 快速開啟筆記的方法：快速切換對話窗與obsidian:// URI 命令行](https://jdev.tw/blog/6445/obsidian-quick-switcher-and-uri-command-line "[Obs#13] 快速開啟筆記的方法：快速切換對話窗與obsidian:// URI 命令行") (0則留言, 2020/10/18)
-   [國人自製Android App：懶人外掛：裝LINE必備，聊天泡泡方便無比](https://jdev.tw/blog/4972/%e6%87%b6%e4%ba%ba%e5%a4%96%e6%8e%9b%ef%bc%9a%e8%a3%9dline%e5%bf%85%e5%82%99 "國人自製Android App：懶人外掛：裝LINE必備，聊天泡泡方便無比") (0則留言, 2017/03/22)
-   [\[Vista\] 符號連結(Symbolic/Soft Link)、永久連結(Hard Link)與連接點(Junction Point)](https://jdev.tw/blog/729/vista-soft-and-hard-link "[Vista] 符號連結(Symbolic/Soft Link)、永久連結(Hard Link)與連接點(Junction Point)") (2則留言, 2008/04/04)
-   [\[Obs＃61\] 提升操作效率的外掛：Pane Relief、Recent Files、Show Current File Path](https://jdev.tw/blog/6941/obsidian-plugins-for-convinient-operations "[Obs＃61] 提升操作效率的外掛：Pane Relief、Recent Files、Show Current File Path") (0則留言, 2021/11/21)
-   [用FinchSync同步Thunderbird與HTC Touch HD的聯絡人資料](https://jdev.tw/blog/1271/finchsync-syncs-thunderbird-htc-touch-hd-contacts "用FinchSync同步Thunderbird與HTC Touch HD的聯絡人資料") (3則留言, 2009/02/01)
-   [\[Obs＃85\] 分享使用中與外觀有關的10個外掛](https://jdev.tw/blog/7095/obsidian-ui-related-10-plugins "[Obs＃85] 分享使用中與外觀有關的10個外掛") (0則留言, 2022/05/01)
-   [\[轉貼\] 美國人最想要的耶誕禮物：華碩Eee PC](https://jdev.tw/blog/666/%e8%bd%89%e8%b2%bc-%e7%be%8e%e5%9c%8b%e4%ba%ba%e6%9c%80%e6%83%b3%e8%a6%81%e7%9a%84%e8%80%b6%e8%aa%95%e7%a6%ae%e7%89%a9%ef%bc%9a%e8%8f%af%e7%a2%a9eee-pc "[轉貼] 美國人最想要的耶誕禮物：華碩Eee PC") (0則留言, 2007/11/15)
-   [時鐘動畫何處尋](https://jdev.tw/blog/38/%e6%99%82%e9%90%98%e5%8b%95%e7%95%ab%e4%bd%95%e8%99%95%e5%b0%8b "時鐘動畫何處尋") (0則留言, 2005/01/13)
-   [\[Android Studio #2\] 操作資源XML檔](https://jdev.tw/blog/3525/android-studio-resources-settings "[Android Studio #2] 操作資源XML檔") (0則留言, 2013/10/03)
-   [建立測試環境以git rebase -i變更Commit歷史](https://jdev.tw/blog/4271/git-rebase-sample "建立測試環境以git rebase -i變更Commit歷史") (0則留言, 2014/10/08)
-   [將Windows 10 Modern App釘選到桌面與快速執行的步驟](https://jdev.tw/blog/4563/windows-10-modern-app-create-shortcut "將Windows 10 Modern App釘選到桌面與快速執行的步驟") (0則留言, 2015/08/12)
-   [\[Tools\] NirCmd: 免費控制Windows的命令列指令](https://jdev.tw/blog/287/tools-nircmd "[Tools] NirCmd: 免費控制Windows的命令列指令") (1則留言, 2005/08/14)
-   [使用croc在不同電腦間複製檔案；跨平台命令行、點對點、續傳](https://jdev.tw/blog/6431/croc-encrypted-file-transfer "使用croc在不同電腦間複製檔案；跨平台命令行、點對點、續傳") (4則留言, 2020/10/01)