Gitea系統有內建自動取得Let's Encrypt SSL憑證的API程式，但我測試時卻一直無法成功…

## 1\. 自動取得憑證的設定

修改 custom/conf/app.ini如下：

```
[server]
PROTOCOL=https
DOMAIN=git.example.com
ENABLE_ACME=true
ACME_ACCEPTTOS=true
ACME_DIRECTORY=https
;; Email can be omitted here and pro
ACME_EMAIL=email@example.com 
```

另有一組LETSENCRYPT\_ACCEPTTOS、LETSENCRYPT\_DIRECTORY等的設定在1.18版後將會被停用，因此用ACME\_開頭是比較正確的設定。  
設定好重啟gitea.exe時取得憑證失敗。

## 2\. 用Certbot取得憑證

後來還是用先前介紹過的[Certbot](https://jdev.tw/blog/7335/certbot-settings)產生CERT\_FILE與KEY\_FILE，用下列設定成功將Gitea Server加上https通訊協定。

```
[server]
...
PROTOCOL=https
CERT_FILE=C:\Certbot\live\git.example.com\cert.pem
KEY_FILE=C:\Certbot\live\git.example.com\privkey.pem
```

## 3\. Client第一次連線

Git client在第一次和Gitea Server連線時出現錯誤：

```
SSL certificate problem: unable to get local issuer certificate ...
```

client可執行下列命令：

```
git config --global http.sslVerify false
```

再次連線時會出現Git Credential Manager對話窗，輸入登入的帳號、密碼後即能正常存取到儲存庫。

## 4\. 參考

-   [啟用Gitea Server的SSH服務，可大幅增加連線速度 – 簡睿隨筆](https://jdev.tw/blog/5246/gitea-server-ssh-settings)

＃＃

#### 您可能也會有興趣的類似文章

-   [\[Git＃9\] Gitea 安裝與設定：輕量級程式碼託管解決方案](https://jdev.tw/blog/7125/gitea-install-and-setup "[Git＃9] Gitea 安裝與設定：輕量級程式碼託管解決方案") (0則留言, 2022/05/22)
-   [啟用Gitea Server的SSH服務，可大幅增加連線速度](https://jdev.tw/blog/5246/gitea-server-ssh-settings "啟用Gitea Server的SSH服務，可大幅增加連線速度") (0則留言, 2018/02/15)
-   [Gitea網頁添加自訂選項以開啟說明文件](https://jdev.tw/blog/7655/gitea-customize-pages "Gitea網頁添加自訂選項以開啟說明文件") (0則留言, 2022/09/01)
-   [\[Windows\] 用Gitea架設自用的Git Server](https://jdev.tw/blog/5089/windows-git-server-gitea "[Windows] 用Gitea架設自用的Git Server") (2則留言, 2017/07/21)
-   [使用SmartGit整合Subversion中央版本庫與Git本地端操作](https://jdev.tw/blog/4984/using-smartgit-integrates-subversion-and-git-operations "使用SmartGit整合Subversion中央版本庫與Git本地端操作") (0則留言, 2017/05/05)
-   [\[Git #2\] 產生SSH金鑰，讓SmartGit與GitHub連線](https://jdev.tw/blog/6049/smartgit-ssh-keygen "[Git #2] 產生SSH金鑰，讓SmartGit與GitHub連線") (0則留言, 2019/12/29)
-   [Subversion版本庫匯入Git的步驟與SVN整合步驟](https://jdev.tw/blog/4256/svn-git-integration "Subversion版本庫匯入Git的步驟與SVN整合步驟") (0則留言, 2014/10/03)
-   [\[Git#8\] 用pre-commit檢查提交時的分支是否正確](https://jdev.tw/blog/6230/git-pre-commit "[Git#8] 用pre-commit檢查提交時的分支是否正確") (0則留言, 2020/04/20)
-   [建立測試環境以git rebase -i變更Commit歷史](https://jdev.tw/blog/4271/git-rebase-sample "建立測試環境以git rebase -i變更Commit歷史") (0則留言, 2014/10/08)
-   [\[Git#1\] SmartGit的安裝與設定](https://jdev.tw/blog/6046/smartgit-install-and-setup "[Git#1] SmartGit的安裝與設定") (0則留言, 2019/12/29)
-   [\[分享\] 濱野純訪談：為什麼 Google 接管開發 Git 2.0 了？](https://jdev.tw/blog/5359/hamano-interview-git-2-0 "[分享] 濱野純訪談：為什麼 Google 接管開發 Git 2.0 了？") (0則留言, 2018/05/25)
-   [Git的Staging Area的中文翻譯探討](https://jdev.tw/blog/4203/git-staging-area-chinese "Git的Staging Area的中文翻譯探討") (2則留言, 2014/09/12)
-   [將Git分支名稱加到提示字元（Prompt）裡](https://jdev.tw/blog/4194/embed-branch-into-prompt "將Git分支名稱加到提示字元（Prompt）裡") (0則留言, 2014/09/02)
-   [\[Batch#3 Git#3\] 如何在Windows批次檔裡將提示字元變更為Git分支名稱？ (初學者的命令行 #7)](https://jdev.tw/blog/6054/git-branch-prompt-with-for "[Batch#3 Git#3] 如何在Windows批次檔裡將提示字元變更為Git分支名稱？ (初學者的命令行 #7)") (0則留言, 2020/01/05)
-   [\[Git#5-2\] 補充說明：產生Commit的檔案清單壓縮檔](https://jdev.tw/blog/6232/smartgit-export-changes-fixed "[Git#5-2] 補充說明：產生Commit的檔案清單壓縮檔") (0則留言, 2020/04/22)