[![](http://jdev.tw/blog/wp-content/uploads/2012/12/jerry-20130318.jpg)](http://jdev.tw/blog)

  

3C設備

作者的敗家記錄，包含iPad、Nexus 7、Galaxy S3等iOS與Android相關電腦設備，也有中華電信ADSL網路與NAS設備等之應用技巧。

Windows學習誌

聚焦於Windows作業系統各個版本的使用經驗與操作技巧，有Windows XP、Windows Vista、Windows 7、Windows 8與Windows Server等不同版本。

檔案、資料夾管理

日常使用電腦時有很大的比例是在操作檔案與資料夾，在此分類裡作者介紹了增進作業效率的各式技巧與心得分享。

生產力工具

與工作效提升有關的各式工具、網站服務，例如Toodledo、Evernote、Google各式服務等之經驗分享。

閱讀筆記

作者於網站衝浪之際，特別轉貼有所感的文章，內容不限電腦科技，尚有不少人文社經之作。

[首頁](http://jdev.tw/blog/) » [HTML/JavaScript/CSS網頁設定](http://jdev.tw/blog/category/software-development/javascript)

作者: 日期: 2022/02/20 – 21:53:26[尚無留言](http://jdev.tw/blog/6996/picsum-ai-generated-faces#respond) | 瀏覽數: 1 / 320

**文章目錄：**

-   [1\. 隨機圖片(假圖)API](http://jdev.tw/blog/6996/picsum-ai-generated-faces#header-0 "Jump to 1. 隨機圖片(假圖)API")
-   [2\. AI臉孔(假臉)API](http://jdev.tw/blog/6996/picsum-ai-generated-faces#header-1 "Jump to 2. AI臉孔(假臉)API")
-   [3\. 免費AI臉孔網站與API](http://jdev.tw/blog/6996/picsum-ai-generated-faces#header-2 "Jump to 3. 免費AI臉孔網站與API")
-   [4\. 免費臉孔網站與API](http://jdev.tw/blog/6996/picsum-ai-generated-faces#header-3 "Jump to 4. 免費臉孔網站與API")
-   [5\. SVG圖示](http://jdev.tw/blog/6996/picsum-ai-generated-faces#header-4 "Jump to 5. SVG圖示")
-   [6\. 中英文假字、文章產生器](http://jdev.tw/blog/6996/picsum-ai-generated-faces#header-5 "Jump to 6. 中英文假字、文章產生器")
-   [7\. 教學影片](http://jdev.tw/blog/6996/picsum-ai-generated-faces#header-6 "Jump to 7. 教學影片")

![01|900](https://raw.githubusercontent.com/emisjerry/upgit/master/2022/02/upgit-20220218_1645178907.png)

> 網址：https://picsum.photos/

1.  正方形圖片：`https://picsum.photos/寬`
2.  矩形圖片：`https://picsum.photos/寬/高`
3.  確保取到不同的圖：附加 `?random=數字`
4.  取灰階圖：附加 `?grayscale`
5.  模糊化：附加 `?blur`

> 網址：https://generated.photos/  
> ![❗](https://s.w.org/images/core/emoji/11.2.0/svg/2757.svg)需要申請API Key，免費版每月只能調用50次

#### 2.1. 網址選項

-   說明：https://generated.photos/account#apikey

| 選項 | 能使用的值 |
| --- | --- |
| emotion (情緒) | joy, neutral, surprise |
| gender (性別) | male, female |
| age (年齡) | infant, child, young-adult, adult, elderly |
| ethnicity (種族) | white, latino, asian, black |
| eye\_color (朣孔顏色) | brown, blue, gray, green |
| hair\_color (髮色) | brown, blond, black, gray, red |
| hair\_length (頭髮長度) | short, medium, long |
| order\_by (排序) | latest, oldest, random, Default: latest. |
| page (頁數) | 取回的頁數，預設是1 |
| per\_page (每頁個數) | 最大100，預設10 |

#### 2.2. API使用範例

```
<script>
  async function getFace() {
    // gender=female
    let response = await fetch("https://api.generated.photos/api/v1/faces?api_key=申請的Key&per_page=3&order_by=random", {
      method: "GET"
    });
    let result = await response.json();
    document.getElementById("img1").src = result.faces[0].urls[4]["512"];
    document.getElementById("img2").src = result.faces[1].urls[4]["512"];
    document.getElementById("img3").src = result.faces[2].urls[4]["512"];
  }

  getFace();
</script>

```

> 網站：https://100k-faces.glitch.me/  
> 說明：[ozgrozer/100k-faces: API for 100,000 AI generated faces](https://github.com/ozgrozer/100k-faces)  
> 圖片來源：https://generated.photos

#### 3.1. 使用方法

1.  直接存取隨機圖片 /random-image

```
<img src="https://100k-faces.glitch.me/random-image" />

```

1.  取回帶有圖片網址的JSON回應: https://100k-faces.glitch.me/random-image-url

```
{"url":"https://ozgrozer.github.io/100k-faces/0/1/001343.jpg"}
```

> 網址: https://randomuser.me/  
> 說明: https://randomuser.me/documentation#howto

#### 4.1. 網址選項

| 選項 | 能使用的值 |
| --- | --- |
| results | 圖片個數 |
| gender (性別) | male, female |
| format | json(預設), PrettyJSON或pretty, csv, yaml, xml |

```
<script>
  async function getFace() {
    // gender=female
    let response = await fetch("https://randomuser.me/api/?results=3", {
      method: "GET"
    });
    let result = await response.json();
    document.getElementById("img1").src = result.results[0].picture.medium;
    document.getElementById("img2").src = result.results[1].picture.medium;
    document.getElementById("img3").src = result.results[2].picture.medium;
  }

  getFace();
</script>
```

> 網址: https://heroicons.dev

免費的SVG圖片，方便製作按鈕的圖示。  
![01|700](https://raw.githubusercontent.com/emisjerry/upgit/master/2022/02/upgit-20220218_1645178654.png)

> 網址: http://www.richyli.com/tool/loremipsum/

-   [MoreText.js: 一用就愛上的中文假文產生器](https://more.handlino.com/api)
-   [中文假文產生器](https://textgen.cqd.tw/)
-   [中文假字生成器 | Chinese Lorem ipsum Generator](https://pinkylam.me/generator/chinese-lorem-ipsum/)
-   [唬爛產生器](https://howtobullshit.me/)

<iframe width="650" height="315" src="https://www.youtube.com/embed/2WEL_RS6k5c" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen=""></iframe>

＃＃

#### 您可能也會有興趣的類似文章

-   [把網路空間當成本機資料夾：Gladinet](http://jdev.tw/blog/1238/gladinet-cloud-desktop-local-directory "2009/01/04") (2則留言, 2009/01/04)
-   [初試AJAX：中文的傳遞與接收](http://jdev.tw/blog/508/ajax-passing-chinese "2007/03/16") (0則留言, 2007/03/16)
-   [IE和FireFox存取同名物件有不同的作法](http://jdev.tw/blog/524/firefox-ie-dom-naming "2007/04/08") (0則留言, 2007/04/08)
-   [\[Obs＃27\] Timelines：時間軸外掛](http://jdev.tw/blog/6584/obsidian-plugin-timelines "2021/03/12") (0則留言, 2021/03/12)
-   [將IE網頁轉換成FireFox也能執行](http://jdev.tw/blog/501/%e5%b0%87ie%e7%b6%b2%e9%a0%81%e8%bd%89%e6%8f%9b%e6%88%90firefox%e4%b9%9f%e8%83%bd%e5%9f%b7%e8%a1%8c "2007/02/28") (0則留言, 2007/02/28)
-   [撰寫API規格文件的利器：API Blueprint與aglio](http://jdev.tw/blog/4924/api-blueprint-with-aglio "2016/12/14") (2則留言, 2016/12/14)
-   [\[Blog\] 在文章底部隨機顯示相關文章的功能說明](http://jdev.tw/blog/540/blog-%e5%9c%a8%e6%96%87%e7%ab%a0%e5%ba%95%e9%83%a8%e9%9a%a8%e6%a9%9f%e9%a1%af%e7%a4%ba%e7%9b%b8%e9%97%9c%e6%96%87%e7%ab%a0%e7%9a%84%e5%8a%9f%e8%83%bd%e8%aa%aa%e6%98%8e "2007/04/26") (6則留言, 2007/04/26)
-   [幫Trac加上TiddlyWiki的雙擊快速編輯功能](http://jdev.tw/blog/593/trac-tiddlywiki-double-click "2007/08/10") (0則留言, 2007/08/10)
-   [事件處理程式與物件的順序對FireFox很重要](http://jdev.tw/blog/502/%e4%ba%8b%e4%bb%b6%e8%99%95%e7%90%86%e7%a8%8b%e5%bc%8f%e8%88%87%e7%89%a9%e4%bb%b6%e7%9a%84%e9%a0%86%e5%ba%8f%e5%b0%8dfirefox%e5%be%88%e9%87%8d%e8%a6%81 "2007/03/02") (0則留言, 2007/03/02)
-   [\[Database\] Aqua Data Studio 4.0.2推出](http://jdev.tw/blog/99/database-aqua-data-studio-402%e6%8e%a8%e5%87%ba "2005/02/23") (0則留言, 2005/02/23)
-   [網頁程式碼美化程式 Google Code Prettify](http://jdev.tw/blog/558/%e7%b6%b2%e9%a0%81%e7%a8%8b%e5%bc%8f%e7%a2%bc%e7%be%8e%e5%8c%96%e7%a8%8b%e5%bc%8f-google-code-prettify "2007/05/27") (0則留言, 2007/05/27)
-   [自動填Google搜尋的輸入字串：Xuite自動書籤按鈕 V1.4](http://jdev.tw/blog/507/%e8%87%aa%e5%8b%95%e5%a1%abgoogle%e6%90%9c%e5%b0%8b%e7%9a%84%e8%bc%b8%e5%85%a5%e5%ad%97%e4%b8%b2%ef%bc%9axuite%e8%87%aa%e5%8b%95%e6%9b%b8%e7%b1%a4%e6%8c%89%e9%88%95-v14 "2007/03/13") (9則留言, 2007/03/13)
-   [Google Reader快速訂閱的FireFox擴充](http://jdev.tw/blog/568/google-reader%e5%bf%ab%e9%80%9f%e8%a8%82%e9%96%b1%e7%9a%84firefox%e6%93%b4%e5%85%85 "2007/06/16") (2則留言, 2007/06/16)
-   [\[Obs＃51\] QuickAdd全攻略(2)：腳本撰寫與巨集使用要點](http://jdev.tw/blog/6821/obsidian-quickadd-scripts-and-macros "2021/09/18") (0則留言, 2021/09/18)
-   [使用Inno Setup包裝安裝程式的備忘](http://jdev.tw/blog/3872/inno-setup-install-packages "2014/03/21") (2則留言, 2014/03/21)

標籤: [Web Services](http://jdev.tw/blog/tag/web-services)