<table><tbody><tr><td><img src="https://blog.darkthread.net/img/calendar.svg"></td><td><span title="Published at 2022-09-25 08:03 AM"><time datetime="2022-09-25T00:03:02" itemprop="datePublished">2022-09-25 08:03 AM</time></span></td><td><img src="https://blog.darkthread.net/img/comment.svg"></td><td><span title="0 comments">0</span></td><td><img src="https://blog.darkthread.net/img/eye.svg"></td><td><span data-ajax-url="/blog/pageviewcount/ms-learn-vuejs-training" title="3,749 pageviews">3,749</span></td><td><a href="https://blog.darkthread.net/Blog/ms-learn-vuejs-training/"><img src="data:image/gif;base64,R0lGODlhAQABAIAAAP///wAAACH5BAEAAAAALAAAAAABAAEAAAICRAEAOw==" id="nllbtn"></a></td></tr></tbody></table>

隨著微軟擁抱開源，微軟官網出現 .NET/C# 以外的語言教學已不是新鮮事(之前就出過 [Python 教學影片](https://blog.darkthread.net/blog/python-for-beginners/))，這回輪到 Vue.js 了!

如果你還不知道 Vue.js 是什麼，這裡簡單科普一下。

Vue.js 是當今(2022 年)的前端三大框架 - VAR (Vue.js、Angular、React) 之一。Vue.js 雖然沒有原廠撐腰(Angular 有 Google、React 有 Facebook)，但 Github Vue.js 專案獲得的星星數曾榮登總排第三 (2022 九月排名則為第七，React 排第九、Angular 第 44，[參考](https://gitstar-ranking.com/repositories))。我心中 Vue.js 與其他二者最大的差別在於它是「漸進式框架」，可以只在 HTML <script src="https://unpkg.com/vue@next"> 引用它繼續用原本的方式開發網頁。不需為了框架多學一整套開發工具(node.js、npm、webpack...)

Vue.js 尤其適合「覺得 jQuery 已經很 OK，只想再補上 MVVM 改善生活品質」的全端工程師(延伸閱讀：[從「鄙視 jQuery」聊起 -技術鄙視從何而來？](https://blog.darkthread.net/blog/jquery-dispise/))，Vue.js 是一把超容易上手的輕巧利器；但若以前端開發為職志，想效法 Gmail、Facebook 將網頁技術發揮到淋漓盡致，漸進式框架的 Vue.js 也有複雜且功能強大的玩法，變身需要技巧方能駕御但威力驚人的重兵器。

依據 Stackoverflow 2022 年的開發者調查，Vue.js 目前在 VAR 中排第三，僅落後 Angular 1.5 個百分點，但我個人看好 Vue.js 會超越 Angular。(延伸閱讀：[StackOverflow 2022 年度開發者調查 - 前端框架誰稱王？你愛你的程式語言嗎？](https://blog.darkthread.net/blog/stackoverflow-survey-2022/))

微軟 Learn 網站(註：前陣子[大改版](https://techcommunity.microsoft.com/t5/microsoft-learn-blog/build-skills-that-open-doors-with-microsoft-learn/ba-p/3614011?WT.mc_id=DOP-MVP-37580)，把官方文件、訓練、認證、QA、程式範例、研討會資料整併成單一入口)，有個「[使用 Vue.js 邁出第一步](https://learn.microsoft.com/zh-tw/training/paths/vue-first-steps/?WT.mc_id=DOP-MVP-37580)」教學，透過實際動手做教你開始用 Vue.js 寫網頁。

![](https://blog.darkthread.net/Posts/files/Fig1_637996611710347170.png)

這個教學適合已具備 HTML/CSS 跟基本 JavaScript 基礎的人，開發工具用的自然是 VSCode，會用到 [Git](https://blog.darkthread.net/blog/my-git-cheatsheet/) 下載程式範例，後期也會用到 npm、webpack 等正統前端工具。教學有翻成中文，看起來不是機器翻譯，品質還不錯。如果你會寫一點網頁，想認識 Vue.js，可以花 110 分鐘實際玩一下。

教學共有四個模組，雖然我已離開新手村一陣子，還是全程走一次，溫故知新兼檢查遺漏知識，筆記如下：

1.  開始使用 Vue
    
    -   介紹用 `<script src="https://unpkg.com/vue@next"></script>` 跟 `const App = Vue.createApp({ ... });` 入水的[輕前端](https://blog.darkthread.net/blog/vue-3-release/)做法
    -   用 git clone 下載範例專案，使用 VSCode 編輯修改程式
    -   介紹好用的 Live Server 擴充功能，感受過程式一存檔在瀏覽器馬上看到結果，你就回不去了 (沒體驗過終生遺憾呀)
    -   學習 v-bind:attribute、:attribute 繫結、v-bind:class="{ cssName: true/false }" 用法、v-bind:style="styleObject" / styleObject = { 'background-color': 'red' } 用法
    -   模組結尾都有個小測驗
2.  使用 Vue.js 的動態頁面顯示
    
    -   學習使用 v-for 跑迴圈將陣列轉成數量不定的 DOM 元素
    -   v-for="(value, index) in arrayData" 同時取陣列元素及序號
    -   從 Github 下載產品陣列資料產生清單 (學到 [Number.toLocaleString()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Number/toLocaleString)，數字轉當地格式)
    -   學習 v-show、v-if、v-else-if、v-else
3.  在 Vue.js 中使用資料與事件
    
    -   學習 v-model 雙向繫結到 input 值
    -   v-model 在 checkbox 時預設對映成 true/false，但也可用 true-value/false-value 定義對映值
    -   v-model 在 select 時對映選取 option 的 value 值
    -   從 Github 下載範例，一個簡單的下拉及 textarea 輸入
    -   學習事件指示詢 v-on:click 或 @click，事件方法寫在 methods 的函式，用 this.propertyName 可存取屬性
    -   學習在 computed 加入函式製作計算屬性，動態產生屬性值
    -   到此為止學到的已可拿來用來寫簡單的輕前端程式，再來 node.js、npm 就要出場了
4.  開始使用 Vue.js 中的 Vue CLI 和單一檔案元件
    
    -   安裝 Node.js，用 npm 下載 Vue CLI (命令列工具)
    -   用 `vue create projName` 建立新專案
    -   了解檔案結構：package.json、public/index.html、src/main.js、src/App.vue、src/components
    -   網頁執行前需要經過編譯，重組出真正要執行的 .html、.js、.css
    -   專業的前端寫法會將網頁拆解成一個個元件，每個元件用的 HTML、JavaScript、CSS 抽取出來包成一個 .vue 檔 (稱為 Single File Componet，SFC)  
        延伸閱讀：[輕前端使用 Vue SFC (.vue 元件)](https://blog.darkthread.net/blog/vue3-sfc-loader/)
    -   SFC 的 JavaScript 及 CSS 若較複雜，也可再拆成 .js 及 .css 供 SFC 載入
    -   VSCode 擴充套件 Vetur 可支援在 VSCode 編輯 SFC (.vue)、另外可安裝 Sarah Drasner 的 Vue VSCode 程式碼片段快速產生 .vue 主結構
    -   元件以 props 定義屬性(如: props: \['prop1','prop2'\])，可透過 `<comp-name prop1="..." prop2="..."></comp-name>` 指定屬性值
    -   元件屬性定義可限制型別、也可傳入複雜型別當成屬性值
    -   要引用元件時，先 `import 元件名 from './元件名.vue'`，再以 components: { 元件名 } 註冊
    -   元件要傳資料到父元件，可寫為 `<button @click="$emit('userUpdated', true)">Save user</button>`  
        父元件寫法：
        
        ```
        <template>
        <user-dialog @user-updated="handleUserUpdated"></user-dialog>
        </template>
        <script>
        import UserDialog from './UserDialog.vue';
        export default {
            methods: {
                handleUserUpdated(success) {
                    if (success) {
                        alert('It worked!!');
                    } else {
                        alert('Something went wrong');
                    }
                }
            },
            components: {
                UserDialog
            }
        }    
        ```
        
    -   元件自訂事件
        
        ```
        props: {
            cabins: Array,
        },
        emits: ['bookingCreated'],
        data() {
            return {
                cabinIndex: -1
            }
        },
        methods: {
            bookCabin() {
                if(this.cabinIndex < 0) return;
                this.$emit('bookingCreated', this.cabinIndex);
                this.cabinIndex = -1;
            },
        }    
        ```
        
        父元件寫法：
        
        ```
        <booking-form @booking-created="addBooking" :cabins="cruise.cabins"></booking-form>
        ```
        

### 看完心得

這個 Vue.js 入門教學還蠻不錯，從最基本 HTML 載入 vue.js 寫 MVVM JavaScript 的輕前端做法，到需要動用 npm、切元件寫 SFC (.vue) 的正統做法都有講到，採實際動手體驗方式，跟著做完大概就知道 Vue.js 是怎麼一回事了。雖然離能用 Vue 寫完專案或賺錢維生還有一大段距離，也算不錯的開始，有興趣的話不妨花兩小時玩看看，就算不喜歡，長見識也好，呵。

Introduce to the Vue.js traning in Microsoft Learn website.