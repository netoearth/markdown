<table><tbody><tr><td><img src="https://blog.darkthread.net/img/calendar.svg"></td><td><span title="Published at 2022-06-29 08:32 AM"><time datetime="2022-06-29T00:32:10" itemprop="datePublished">2022-06-29 08:32 AM</time></span></td><td><img src="https://blog.darkthread.net/img/comment.svg"></td><td><span title="0 comments">0</span></td><td><img src="https://blog.darkthread.net/img/eye.svg"></td><td><span data-ajax-url="/blog/pageviewcount/setup-openvpn-on-ddwrt" title="334 pageviews">334</span></td><td><a href="https://blog.darkthread.net/Blog/setup-openvpn-on-ddwrt/"><img src="data:image/gif;base64,R0lGODlhAQABAIAAAP///wAAACH5BAEAAAAALAAAAAABAAEAAAICRAEAOw==" id="nllbtn"></a></td></tr></tbody></table>

最近重整家中網路配置，我有台刷了 [DD-WRT 韌體](https://zh.wikipedia.org/zh-tw/DD-WRT)的古老網路分享器，心血來潮想試試它的 OpenVPN 功能。網路上中英文教學多如牛毛，原以為是個簡單任務，不料頻頻卡關，結論是在 Windows 安裝使用 OpenVPN 需要相關知識背景，就像 Linux 一樣，需要敲一堆指令，頗有在當駭客的錯覺。總之，這篇會設定過程的眉角整理成筆記，等待某天有緣人參考。

我的主要參考文件：

-   [OpenVPN 文件：Easy Windows Guide](https://community.openvpn.net/openvpn/wiki/Easy_Windows_Guide)
-   [How To Install and Configure OpenVPN On Your DD-WRT Router](https://www.howtogeek.com/64433/how-to-install-and-configure-openvpn-on-your-dd-wrt-router/)

安裝記錄如下：

1.  下載安裝 OpenVPN Client，下載網址在 [https://openvpn.net/community-downloads/](https://openvpn.net/community-downloads/)  
    安裝程序很簡單，下一步下一步而已，預設程式會安裝在 C:\\Program Files\\OpenVPN\\
2.  幾乎所有教學都說下一步要執行 C:\\Program Files\\OpenVPN\\easy-rsa 下的程式建立憑證，但你會發現根本沒有 easy-rsa 這個目錄  
    依據[論壇討論](https://forums.openvpn.net/viewtopic.php?t=13059)，OpenVPN 2.3+ 起已拿掉 easy-rsa，需自行[從 Github 下載](https://github.com/OpenVPN/easy-rsa-old)。git clone 或下載 ZIP 解壓後，將 easy-rsa\\Windows 下的檔案複製到 C:\\Program Files\\OpenVPN\\easy-rsa 資料夾。
3.  將 vars.bat.sample 更名為 vars.bat 並開啟修改，裡面有 KEY\_COUNTRY、KEY\_PROVINCE、KEY\_CITY、KEY\_ORG、KEY\_EMAIL、KEY\_CN、KEY\_NAME、KEY\_OU 等欄位，會作為稍後建立憑證輸入資訊的預設值，但依我實測在 Windows 無效，openssl 還會用原本的預設值，可隨便填。至於 PKCS11\_MODULE\_PATH、PKCS11\_PIN，有用到 Dongle 等硬體加密裝置才需要 [參考](https://forum.archive.openwrt.org/viewtopic.php?id=46946)，一般可忽略。  
    easy-rsa 依賴 OpenSSL 工具產生金鑰與憑證，Windows 未內建 OpenSSL，建議可安裝 [Cmder](https://cmder.net/) 或 [Git for Windows](https://git-scm.com/download/win) (順便[學一下 Git](https://blog.darkthread.net/blog/my-git-cheatsheet/) 讓你人生變彩色)，用 where openssl 找到 openssl.exe，設定檔在隔壁資料夾 ..\\ssl\\openssl.cnf，將甚複製到 easy-rsa，再修改 KEY\_CONFIG=openssl.cnf。  
    ![](https://blog.darkthread.net/Posts/files/Fig1_637920595670328156.png)
4.  執行 vars.bat 設好環境參數，執行 build-ca.bat 建立 CA 憑證，填寫組織資料的預設值依教學文件應帶出 vars.bat 中的 KEY\_\* 變數，但實測則是以 openssl.cnf 的設定為準，所以手動重填吧。  
    ![](https://blog.darkthread.net/Posts/files/Fig2_637920595671015029.png)
5.  下一步是用 build-key.bat 建立客戶端憑證。剛才 build.ca.bat 產生的 CA 憑證與金鑰在 C:\\Program Files\\OpenVPN\\easy-rsa\\keys\\ca.crt 及 ca.key。但 build-key.bat 在產生憑證時並未指定 ca.crt 及 ca.key 位置，真懷疑教學文件到底是怎麼成功的？  
    我採用的方法是修改 C:\\Program Files\\OpenVPN\\easy-rsa\\openssl.cnf 使其正確對映 keys\\ca.crt 及 ca.key，另外，預設新憑證是寫入 keys\\newcerts，openssl.exe 不會自動建立資料夾，要 mkdir 一下，另外有兩個設定 rand\_serial = no、email\_in\_dn = no 是用來防止 build-key-server.bat 出錯：  
    ![](https://blog.darkthread.net/Posts/files/Fig3_637920595671818241.png)  
    注意以下範例中 build-key t470p 的 t470p 是客戶端名稱，要於 CN 一致：  
    ![](https://blog.darkthread.net/img/loading.svg)
6.  接著是伺服器端憑證 build-key-server dd-wrt，此還會踩到一個 "Error Loading extension section server" 錯誤，請在 openssl.cnf 加入以下這段： [參考](http://wordpress.joontech.nl/?p=83)
    
    ```
    [ server ]
    # http://wordpress.joontech.nl/?p=83
    nsCertType = server
    ```
    
    伺服器憑證終於也順利產生了：  
    ![](https://blog.darkthread.net/img/loading.svg)
7.  用 build-dh.bat 產生加密用質數 keys\\dh2048.pem，這個步驟很單純不會出錯。一路摔坑下來，忽然發現「什麼都不用改就成功」是多麼幸福的事，easy-rsa 一點都不 Easy 啊啊啊啊~

最麻煩的部分搞定，餘下照[教學](https://www.howtogeek.com/64433/how-to-install-and-configure-openvpn-on-your-dd-wrt-router/)修改 client.ovpn、設定 DD-WRT Daemon 填入剛才產生的 ca.crt、server.crt、server.key、dh2048.pem，加入 iptables 路由，應該就能成功了。

後記：後來也試了在 Asus 無線基地台設定 OpenVPN，操作簡單許多，啟用 OpenVPN、下載 client.ovpn、設定帳號密碼三個步驟就做完了，簡單十倍。[參考](https://www.asus.com/tw/support/FAQ/1008713/)