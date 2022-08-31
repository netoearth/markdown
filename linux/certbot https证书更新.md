
Certbot是自動安裝[Let’s Encrypt](https://letsencrypt.org/zh-tw/docs/) SSL憑證的本地端工具，測試性質或不對外的網站可以透過它來安裝SSL憑證，讓網站能使用https通訊協定。

> 下載：https://dl.eff.org/certbot-beta-installer-win32.exe

## 1\. 安裝

執行下載的certbot-beta-installer-win32.exe，會安裝在`C:\Program Files (x86)\certbot`，執行bin/certbot.exe會寫出檔案到`C:\Certbot\`  
安裝成功後，在正確的安裝後，Windows排程裡會增加自動renew的排程，每三個月必須重新取得新的SSL憑證。

網站上找得到的renew步驟都語焉不詳或使用後無效，因此自己重新測試並完成本篇操作步驟。

## 2\. 產生SSL憑證

用下列命令產生SSL憑證檔到`C:\Certbot`資料夾。

> \[!TIP\] 命令 certbot certonly –manual –manual-auth-hook 認證命令  
> –manual-cleanup-hook 善後命令 –preferred-challenges http -m 你的Email -d  
> 要使用的網址(沒有https://)  
> ![✅](https://s.w.org/images/core/emoji/14.0.0/svg/2705.svg) 範例 certbot certonly –manual  
> –manual-auth-hook c:\\temp\\auth.bat –manual-cleanup-hook c:\\temp\\cleanup.bat  
> –preferred-challenges http -m yourname@yourmail.com -d  
> yourdomain.com

說明如下：

1.  必須通知網域認證才能取得憑證，此處以`--preferred-challenges http`指定認證方法是http，因此網站的http通訊必須能瀏覽
2.  第一次建立時必須指定認證的文字檔，文字檔內是Certbot指定的內容。以auth.bat來接收參數並產生認證檔（假設文件根目錄是c:\\Apache\\htdocs\\）：
    
    ```
    echo %CERTBOT_VALIDATION% > c:\Apache\htdocs\.well-known\acme-challenge\%CERTBOT_TOKEN%
    ```
    

3.  善後處理的批次檔 cleanup.bat 再把認證文字檔刪除掉

```
del c:\Apache\htdocs\.well-known\acme-challenge\%CERTBOT_TOKEN% /y
```

4.  執行成功後在`c:\Certbot\域名\`資料夾即會產生SSL憑證檔

## 3\. Renew

[Let’s Encrypt](https://letsencrypt.org/zh-tw/docs/) SSL憑證每三個用需要更新一久，上述步驟執行成功後，在排程裡會自動產生每日定時執行的命令：

```
PowerShell.exe -NoProfile -WindowStyle Hidden -Command "certbot renew"
```

![](https://raw.githubusercontent.com/emisjerry/upgit/master/2022/07/upgit-20220726_1658806090.png)

