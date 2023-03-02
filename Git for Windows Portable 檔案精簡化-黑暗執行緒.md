<table><tbody><tr><td><img src="https://blog.darkthread.net/img/calendar.svg"></td><td><span title="Published at 2022-01-17 09:29 PM"><time datetime="2022-01-17T13:29:03" itemprop="datePublished">2022-01-17 09:29 PM</time></span></td><td><img src="https://blog.darkthread.net/img/comment.svg"></td><td><span title="0 comments">0</span></td><td><img src="https://blog.darkthread.net/img/eye.svg"></td><td><span data-ajax-url="/blog/pageviewcount/compact-git-for-windows" title="2,252 pageviews">2,252</span></td><td><a href="https://blog.darkthread.net/Blog/compact-git-for-windows"><img src="data:image/gif;base64,R0lGODlhAQABAIAAAP///wAAACH5BAEAAAAALAAAAAABAAEAAAICRAEAOw==" id="nllbtn"></a></td></tr></tbody></table>

在 [web.config PowerShell 更新函式庫](https://blog.darkthread.net/blog/ps-web-conf-lib/)中，我借用 git diff 比對 web.config 修改前後變化。由於並非所有主機都會安裝 Git for Windows，我想到讓工具自帶[可攜版 Git for Windows Portable](https://git-scm.com/download/win) 的解法，但有點美中不足，Git for Windows Portable 容量高遠 271MB：

![](https://blog.darkthread.net/Posts/files/Fig1_637780235028757100.png)

為了簡單的異動對照功能，包進 6052 個檔案佔用近 300MB 超不划算，但 git diff 已用得順手，實在不想花時間找替代方案，於是我把腦筋動到「整理一份能跑 git diff 的最精簡版本」。

方法不難，只需要夠強的動機跟一點耐心。用 [Process Monitor](https://blog.darkthread.net/blog/977/) 觀察 Git for Windows 執行 git diff 過程讀到哪些檔案：(小技巧：Filter 條件可設定檔案路徑 (Path) 以 GitPortal 起始 + Result 等於 SUCCESS，聚焦成功讀取的檔案)

![](https://blog.darkthread.net/Posts/files/Fig2_637780235030994259.png)

git diff 過程動用了 git.exe 跟 less.exe 兩個執行檔、一些 dll 程式庫及設定檔。Process Monitor 可將存取記錄匯出成 PML( Process Monitor 專屬格式)、XML 或 CSV，大家知道我要做什麼了吧？

![](https://blog.darkthread.net/Posts/files/Fig3_637780235031487153.png)

沒錯，那就寫一段 PowerShell 將用到的檔案自動打包吧!

```
$dupCheck = @{}
Get-Content D:\GitFileLog.CSV | ConvertFrom-Csv | 
ForEach-Object {
    $srcPath = $_.Path
    if (Test-Path $srcPath -PathType Leaf) {
        $dstPath = $srcPath.Replace('X:\Github\PSLab\IisConfig\GitPortable\', 'D:\GitCompact\')
        if (!$dupCheck.ContainsKey($dstPath)) {
            $dupCheck.Add($dstPath, $true)
            [System.IO.Directory]::CreateDirectory([System.IO.Path]::GetDirectoryName($dstPath)) | Out-Null
            Copy-Item $srcPath $dstPath
        }
    }
}
```

精簡版共 16 個檔案合計 9.5MB，體積縮小到原來的 3.5%，實測也能順利跑完 git diff，大成功!

![](https://blog.darkthread.net/img/loading.svg)

I want to use git diff but Git for Windows Portable take more than 270MB, so I use Process Monitor to pick required file to create a minimal version for git diff.