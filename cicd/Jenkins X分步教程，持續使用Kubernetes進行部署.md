在TestProject中，我們努力使用最新的最佳實踐，並且作為基礎結構中某些組件和工作流程升級的一部分，我們將部分遷移到JenkinsX。Jenkins X無伺服器設置上缺少有用的資料，因此，出於我們對共享和回饋社區的信念的一部分，我們決定創建一個功能完善的分步教程！

Jenkins X無伺服器和Kubernetes的持續集成解決了以下問題：

1.  為CI / CD中涉及的各個組件維護專用伺服器，例如：構建伺服器，測試環境，編排伺服器等。
2.  自動部署並發按需構建和測試環境，允許測試人員和開發人員並行發布和測試專用功能。
3.  與微服務開發和發布流程自然集成。

在本分步指南中，我們將使用Jenkins X為基於微服務的現代雲應用程式構建CI / CD解決方案。

Jenkins X是用於Kubernetes上現代雲應用程式的CI / CD解決方案。它可以幫助我們為項目創建良好的渠道，並實施完整的CI和CD。

持續集成（CI）是一種軟體開發方法，用於自動將代碼更改從多個貢獻者集成到單個軟體項目中。連續部署（CD）是一種軟體開發方法，該方法可以自動，快速且安全地將軟體部署到生產環境中。

Jenkins X通過GitOps自動化環境管理以及在環境之間促進應用程式新版本的升級。

GitOps是用於Kubernetes集群管理和應用程式交付的軟體開發方法。當遵循這種方法時，Git是基礎架構即代碼和應用程序代碼的唯一真實來源。對所需狀態的所有更改都是Git提交。

Jenkins X為我們的拉取請求自動啟動預覽環境，因此我們可以在將更改合併到主文件之前獲得快速反饋。

## 讓我們開始吧！

就本教程而言，我們創建了一個帶有微服務設計的簡單演示項目，該項目將部署在Kubernetes上。我們的演示項目包括3個主要服務：一個前端應用程式和2個後端服務。

我們的演示項目顯示了後端服務的當前版本及其本身。這將有助於說明和測試我們稍後構建的CI / CD管道。

我們的演示項目服務使用Python編寫，並打包到基於Alpine映像的Docker容器中。下圖說明了我們的演示項目：

![Jenkins X分步教程，持續使用Kubernetes進行部署](https://images.twgreatdaily.com/images/image/VV6/VV6lDnEBrZ4kL1Vig_zv.jpg)

在本教程結束時，我們將擁有一個有效的CI / CD解決方案，使我們能夠以快速，可靠的方式開發，測試和交付我們的應用程式。

## Jenkins X無伺服器架構

在開始本教程的動手部分之前，我們需要介紹Jenkins X無伺服器基礎架構的主要組件。下圖說明了這些組件：

![Jenkins X分步教程，持續使用Kubernetes進行部署](https://images.twgreatdaily.com/images/image/Vl6/Vl6lDnEBrZ4kL1Vihfw-.jpg)

讓我們再深入一點：

-   **Kubernetes** –雲應用程式開發的核心，我們在其中構建，測試和部署應用程式。
-   **Docker映像註冊表** –構建它們後，我們將在其中存儲Docker映像。需要注意的一點是：默認情況下，Jenkins X使用您的雲提供商的Docker註冊表，但是可以將其配置為使用其他私有註冊表。
-   **Git** –保存代碼的地方–應用程式原始碼，系統配置和系統狀態。
-   **航海圖博物館** –我們存儲舵圖的位置。
-   **測試框架** –我們在測試應用程式更改的地方使用 TestProject（免費）。
-   **Kaniko** –構建我們的Docker容器。
-   **Skaffold** –在Kubernetes上部署我們的Docker容器。
-   **草稿** –創建用於部署應用程式的Helm圖表。
-   **Tekton** –這就是運行我們的管道的原因。
-   **Jenkins X** –協調CI / CD流程。

## 安裝和設置Jenkins X Serverless

現在，我們將安裝Jenkins X，並使用Google Cloud作為我們的雲提供商，但是安裝過程與其他雲非常相似。

### **先決條件**

1.  Google雲帳戶（如果您還沒有，請在此處創建一個）。
2.  Github帳戶（如果您還沒有，則可以在此處創建一個）。

**在Google Kubernetes Engine（GKE）上安裝Jenkins X**

1.  登錄到您的Google雲帳戶。
2.  轉到控制台（右上角的控制台按鈕）。
3.  轉到您的雲外殼（位於上方藍色欄右側的\[> \_\]按鈕）。
4.  運行以下命令來安裝jx：

![Jenkins X分步教程，持續使用Kubernetes進行部署](https://images.twgreatdaily.com/images/image/WF6/WF6lDnEBrZ4kL1ViiPy1.jpg)

5.運行以下命令以：

-   部署為Jenkins X配置的新Kubernetes集群。
-   部署無伺服器的Jenkins X組件：

![Jenkins X分步教程，持續使用Kubernetes進行部署](https://images.twgreatdaily.com/images/image/Wl6/Wl6lDnEBrZ4kL1ViivwK.jpg)

6.現在，將詢問一系列配置安裝的問題：

1.  與集群關聯的項目。如果您是新的Google雲用戶，請選擇「我的第一個項目」，否則請選擇所需的人。
2.  群集的部署位置（地理區域）。選擇最接近您的一個。
3.  機器類型，如果您沒有經驗，請使用默認值。
4.  要部署的節點數，與上面相同，可以使用默認值。
5.  您想要的Jenkins X部署類型，選擇帶有Tekton Pipelines的無伺服器JenkinsX。
6.  系統將詢問您是否要創建負載平衡器，同意該負載平衡器，否則您將無法工作。
7.  系統將詢問您是否要將nip.io用作魔術DNS。同意這一點。
8.  不要忘記安裝會在您的Github帳戶中創建多個存儲庫並詢問管理員權限，強烈建議您為教程創建一個新帳戶。您將被問到有關您的Github帳戶的各種問題，請相應回答，不要使用電子郵件作為帳戶名，這可能會在以後出現問題，請使用您為該帳戶選擇的暱稱。
9.  安裝將開始，安裝完成後，將為您提供一些有用的連結和Jenkins X相關服務的密碼，請不要忘記將其保存在將來使用的地方。

而已！您已準備好開始。

## 將現有項目導入Jenkins X

在這一部分中，我們將學習如何將現有項目導入JenkinsX。

### **Jenkins X默認創建的不同環境**

在將項目導入到Jenkins X之前，我們需要了解Jenkins X默認創建的不同環境是什麼。默認情況下，Jenkins X創建以下環境：

1.  **jx環境** –在此環境中，我們執行構建和單元測試。
2.  登台**環境** –在此環境中，我們執行集成測試。
3.  **生產環境** –我們的最終生產版本。
4.  **預覽環境** –用於測試的環境。

### **第1步–創建新存儲庫**

您需要做的第一件事是將資源庫與項目的原始碼進行分叉，然後將其克隆到您的開發環境中。

在本教程的這一章中，我們將以Backend1演示項目為例。您應該派生主教程存儲庫並運行以下命令，以將其克隆到您的開發環境中：

![Jenkins X分步教程，持續使用Kubernetes進行部署](https://images.twgreatdaily.com/images/image/W16/W16lDnEBrZ4kL1Vii_zl.jpg)

現在，您可以使用一個新的存儲庫，但是仍然沒有Jenkins X所需的所有文件。

### **第2步–為Jenkins X添加必要的文件**

要為Jenkins X添加必要的文件，您需要做的第一件事就是導入項目。下面的命令將導入項目。在運行此命令之前，請驗證您是否位於我們剛剛克隆到開發環境中的項目的主目錄中：

![Jenkins X分步教程，持續使用Kubernetes進行部署](https://images.twgreatdaily.com/images/image/XV6/XV6lDnEBrZ4kL1Vijfxy.jpg)

作為默認導入過程的一部分，Jenkins X將遍歷代碼，並根據程式語言為項目選擇正確的默認構建包。我們的項目是使用Python開發的，因此它將是一個Python構建包。

您可以通過查看以下內容查看構建包的內容：

![Jenkins X分步教程，持續使用Kubernetes進行部署](https://images.twgreatdaily.com/images/image/X16/X16lDnEBrZ4kL1VikPyu.jpg)

要驗證導入進度，請運行以下命令：

![Jenkins X分步教程，持續使用Kubernetes進行部署](https://images.twgreatdaily.com/images/image/YF6/YF6lDnEBrZ4kL1VikvxK.jpg)

導入成功完成後，您將看到構建包中的所有文件現在都顯示在項目存儲庫中，不同之處在於其中一些以前是模板，而現在是合法文件。

### **等等，這裡發生了什麼？**

讓我們回顧一下導入過程幕後發生的情況：

1.  Jenkins X選擇正確的構建包並將新文件推送到項目的存儲庫。
2.  Jenkins X創建一個Jenkins項目。
3.  Jenkins X創建一個Webhook，以在項目存儲庫中的代碼更改時觸發構建。
4.  Jenkins X使用skaffold.yaml構建容器映像。
5.  Jenkins X使用掌舵圖表來創建構建並將其推送到圖表博物館。
6.  Jenkins X將更改提交到暫存環境存儲庫。
7.  它將新服務版本添加為對環境的依賴。
8.  它為新服務添加了一個入口。
9.  Jenkins X將更改應用於Kubernetes集群，新服務由Kubernetes部署在登台環境中。
10.  Jenkins X為我們提供了新服務的URL。
11.  我們可以通過HTTP與我們的服務進行交互。

令人印象深刻，是嗎？

這個過程的妙處在於，所有更改都在Git中進行了跟蹤，可以通過以下步驟進行查看：

為了驗證服務是否已正確部署到登台環境中，可以在導入過程結束時通過將在控制台中列印的URL查看服務。例如，在我們的演示項目Backend1中，打開提供的URL後，以下JSON應出現在瀏覽器中：

![Jenkins X分步教程，持續使用Kubernetes進行部署](https://images.twgreatdaily.com/images/image/YV6/YV6lDnEBrZ4kL1Vik_yC.jpg)

您可以看到獲得了服務的名稱和版本，但是單元測試又如何呢？默認構建包中不包含此階段。為了解決這個問題，您將需要一個針對Jenkins X的自定義構建包。

## 為Jenkins X創建自定義構建包

在這一部分中，我們將為Jenkins X創建一個自定義構建包，並將單元測試作為其中的一部分。

構建包定義了每次開發人員更改項目原始碼時如何構建，測試和交付項目。Jenkins X提供了基本的構建包，但在現實生活中，您需要為項目創建一個自定義的構建包。

建議將構建包保存在中央存儲庫中，並在公司的開發人員之間共享它們。

為了為教程項目創建自定義構建包，您需要做的第一件事是將官方構建庫與社區構建包合併。然後，使用以下命令將存儲庫克隆到您的開發環境：

![Jenkins X分步教程，持續使用Kubernetes進行部署](https://images.twgreatdaily.com/images/image/Y16/Y16lDnEBrZ4kL1VilPy9.jpg)

建議添加上游存儲庫以接收來自官方存儲庫的將來更新，以保持最新狀態。使用以下命令來這樣做：

![Jenkins X分步教程，持續使用Kubernetes進行部署](https://images.twgreatdaily.com/images/image/ZF6/ZF6lDnEBrZ4kL1VilvxN.jpg)

### **在Jenkins X中選擇構建包的來源**

默認情況下，構建包的源位於以下文件夾中：

![Jenkins X分步教程，持續使用Kubernetes進行部署](https://images.twgreatdaily.com/images/image/a16/a16lDnEBrZ4kL1VimPw8.jpg)

要將源更改為指向新創建的存儲庫，請運行以下命令：

![Jenkins X分步教程，持續使用Kubernetes進行部署](https://images.twgreatdaily.com/images/image/bF6/bF6lDnEBrZ4kL1VimvxB.jpg)

從現在開始，當您導入新項目時，Jenkins X將使用新創建的存儲庫中的構建包。

默認情況下，Jenkins X提供流行開發語言的構建包。由於我們的項目是用Python編寫的，因此，我們將使用Python構建包作為定製構建包的基礎。

將Python build pack文件夾複製到一個名為tutorial-build-pack的文件夾中。為此，請運行以下命令：

![Jenkins X分步教程，持續使用Kubernetes進行部署](https://images.twgreatdaily.com/images/image/bV6/bV6lDnEBrZ4kL1Vim_yv.jpg)

現在，讓我們看一下我們剛剛創建的新構建包文件夾中的文件：

-   **Dockerfile** –包含**docker鏡像**的構建。
-   **圖表** –包含Helm圖表和Kubernetes文件，這些文件描述了如何將項目部署到暫存和生產環境。
-   **pipeline.yaml** –包含構建，測試和交付項目的步驟。
-   **預覽** –包含Helm圖表和Kubernetes文件，這些文件描述了如何將項目部署到預覽環境。
-   **skaffold.yaml** –包含構建和存儲**Docker映像**的步驟。

### **在Jenkins X中將單元測試支持添加為自定義構建包的一部分**

為了在定製構建包中實施單元測試，您需要擴展基本構建包。要擴展構建包，您只需要修改位於構建包主目錄中的pipeline.yaml文件即可。

假設您要添加一個在構建映像之前運行一些單元測試的腳本。為了示例，您將運行echo命令「正在運行單元測試！」。但實際上，您應該運行將運行測試的命令或腳本。

現在，在構建服務之前，將新步驟添加到pipeline.yaml以執行新腳本。使用以下命令來這樣做：

![Jenkins X分步教程，持續使用Kubernetes進行部署](https://images.twgreatdaily.com/images/image/bl6/bl6lDnEBrZ4kL1VinPzS.jpg)

新文件應如下所示：https : //github.com/testproject-io/jenkinsx-tutorial/blob/master/build-pack-pipeline.yaml

為了使更改生效，我們需要將更改提交到構建包。但是，新創建的構建包未與任何項目關聯。

就本教程而言，如果您已經分叉並克隆了主教程存儲庫，則跳過下一步，只需在存儲庫文件夾內輸入Backend2文件夾。

如果尚未克隆主教程存儲庫，請運行以下命令：

![Jenkins X分步教程，持續使用Kubernetes進行部署](https://images.twgreatdaily.com/images/image/b16/b16lDnEBrZ4kL1Vinfzx.jpg)

現在，我們有了一個新的存儲庫。要使用已創建的構建包導入新項目，請運行以下命令：

![Jenkins X分步教程，持續使用Kubernetes進行部署](https://images.twgreatdaily.com/images/image/cF6/cF6lDnEBrZ4kL1Vin_yB.jpg)

為了檢查是否使用我們的新單元測試正確執行了構建，請運行以下命令：

![Jenkins X分步教程，持續使用Kubernetes進行部署](https://images.twgreatdaily.com/images/image/cl6/cl6lDnEBrZ4kL1Viofw2.jpg)

恭喜！您的單元測試正在執行中！

## Jenkins X中的集成測試

在這一部分中，我們將學習如何對通過Jenkins X部署的服務實施集成測試。

為了了解在哪裡實施集成測試，您需要了解它們的目的–驗證我們生產環境中的所有服務將以正確的方式相互通信。

在Jenkins X中，您可以根據需要擁有任意數量的環境，但是現在我們將重點放在登台環境上，因為該環境模擬了將發布到生產環境中的下一個版本。因此，如果服務在登台環境中以正確的方式進行通信，那麼我們可以假定，如果將它們轉移到生產環境中，它們將無縫運行。

如果您從一開始就已按照本教程進行操作，則應該具有一個登台環境，其中包括：兩個工作服務，Backend1和Backend2。這些服務彼此之間沒有交互，因此我們無法執行實際的集成測試來驗證服務是否正確通信。

為了使集成測試示例有意義，我們將添加一個與兩個後端服務交互的前端服務。

就本教程而言，如果您已經分叉並克隆了主教程存儲庫，則跳過下一步，只需在存儲庫文件夾內輸入Frontend文件夾。

如果尚未克隆主教程存儲庫，請運行以下命令：

![Jenkins X分步教程，持續使用Kubernetes進行部署](https://images.twgreatdaily.com/images/image/dF6/dF6lDnEBrZ4kL1Viovy0.jpg)

導入成功完成後，請驗證3個服務是否已在臨時環境中啟動並正在運行。

要驗證這三個服務是否可用，您可以運行以下命令：

![Jenkins X分步教程，持續使用Kubernetes進行部署](https://images.twgreatdaily.com/images/image/dV6/dV6lDnEBrZ4kL1VipPw0.jpg)

## **如何通過API使用TestProject執行集成測試**

TestProject是由社區提供動力的測試自動化平台，完全免費。它建立在Selenium和Appium之上，支持所有主要作業系統，並使軟體團隊的每個成員都可以輕鬆協作和測試Web，Android和iOS應用程式。TestProject非常適合CI / CD管道內的集成測試，因為TestProject公開了RESTful API，該API允許研發團隊安排和觸發自動化，獲取狀態並檢索測試結果。此外，TestProject代理可輕鬆部署在各種平台（本地和遠程）上，包括可在無伺服器的情況下部署的代理的docker化版本，從而可按需進行並行分布式測試。

讓我們開始吧！

1.  在這裡創建一個TestProject帳戶，它是完全免費的！
2.  下載並註冊TestProject代理。
3.  創建您的集成測試。在此示例中，我們將使用記錄的測試，但是您也可以使用TestProject以C＃或Java開發Selenium和Appium編碼的測試。
4.  用TestProject生成您的API密鑰，以從您的CI / CD工具訪問TestProject平台。
5.  使用其API在TestProject中執行測試自動化作業。
6.  例如，您可以使用以下Python腳本從CI / CD工具執行TestProject作業：https : //github.com/testproject-io/jenkinsx-tutorial/blob/master/trigger-job.py

現在，讓我們實施集成測試！您需要做的第一件事是將登台環境存儲庫克隆到您的開發環境。使用以下命令：

![Jenkins X分步教程，持續使用Kubernetes進行部署](https://images.twgreatdaily.com/images/image/dl6/dl6lDnEBrZ4kL1Vipfx9.jpg)

將存儲庫克隆到您的開發環境之後，可以在主目錄文件夾中找到一個名為jenkins-x.yaml的文件。該文件描述了代表將服務部署到登台環境的管道的步驟。

為了在構建後執行集成測試，我們需要向該文件添加以下命令：

![Jenkins X分步教程，持續使用Kubernetes進行部署](https://images.twgreatdaily.com/images/image/eV6/eV6lDnEBrZ4kL1ViqPz_.jpg)

新文件應如下所示：https : //github.com/testproject-io/jenkinsx-tutorial/blob/master/staging-pipeline.yml

現在，我們已經添加了將執行集成測試的命令，我們可以將更改推送到Git。

恭喜，每次將任何部署到暫存環境的集成測試都將執行！

您可以通過查看構建日誌來驗證這一點：

![Jenkins X分步教程，持續使用Kubernetes進行部署](https://images.twgreatdaily.com/images/image/el6/el6lDnEBrZ4kL1ViqvyE.jpg)

感謝你的閱讀

點擊關注私信小編「資源」即可獲得JAVA免費學習資料。