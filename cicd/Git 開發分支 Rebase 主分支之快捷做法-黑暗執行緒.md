<table><tbody><tr><td><img src="https://blog.darkthread.net/img/calendar.svg"></td><td><span title="Published at 2022-10-07 08:56 PM"><time datetime="2022-10-07T12:56:20" itemprop="datePublished">2022-10-07 08:56 PM</time></span></td><td><img src="https://blog.darkthread.net/img/comment.svg"></td><td><span title="2 comments">2</span></td><td><img src="https://blog.darkthread.net/img/eye.svg"></td><td><span data-ajax-url="/blog/pageviewcount/rebase-master-efficently" title="1,278 pageviews">1,278</span></td><td><a href="https://blog.darkthread.net/Blog/rebase-master-efficently/"><img src="data:image/gif;base64,R0lGODlhAQABAIAAAP///wAAACH5BAEAAAAALAAAAAABAAEAAAICRAEAOw==" id="nllbtn"></a></td></tr></tbody></table>

使用 Git 協同開發時，我常遇到以下情境。

從主分支 master 新開了 jeffrey-work 分支寫新功能，於此同時團隊其他成員(假設叫 Eric 好了)也從 master 開了分支改程式，比我早開發好已併入 master 並 push 到版控主機。得知消息後，我做了 fetch 取得遠端最新狀態準備 Rebase 新版 master，此時 Git 樹會像這樣：

![](https://blog.darkthread.net/Posts/files/Fig1_638007442154784709.png)

origin/master 是伺服器上的 master 分支，比我本機的 master 多了 Eric 修改的五個 Commit，而我的 jeffrey-work 分支需更新成以 origin/master 為基準，故要進行以下操作再繼續開發：

1.  將本機 master 更新到 origin/master 的位置
2.  jeffrey-work 對 master 做 Rebase

最後要調成這樣：

![](https://blog.darkthread.net/Posts/files/Fig2_638007442156958771.png)

如果不要多想，直覺會這樣操作：

```
git checkout master
git pull 
git checkout jeffrey-work
git rebase master
```

這個做法絕對可行，但卻未必有效率，若 Eric 跟我的分支援都包含大量檔案修改時，過程檔案會反覆還原更新。想像一個極端狀況，Eric 的五個 Commit 改了 100 個檔案，jeffrey-work 的五個 Commit 改了 200 個檔案，在 rebase 動作外會發生什麼事？

git checkout master -> 還原 200 個檔案  
git pull -> 套用 100 個檔案異動  
git checkout jeffrey-work -> 還原 100 個檔案，套用 200 個檔案異動

來來回回總有 600 次檔案異動。

我想到一個改良做法：

```
git rebase origin/master
git checkout master
git pull
```

乍看可以減少到 400 次：

git checkout master -> 還原 200 + 100 個檔案異動  
git pull -> 套用 100 個檔案異動

但想想不對，我還得 git checkout jeffrey-work 才能繼續開發，這下要再套用 200 個檔案異動，完全沒省到!

最後，我學到一個新參數 git branch --force，再改良如下：

```
git rebase origin/master
git branch --force master origin/master
```

直接把 master 標籤撕下來貼在 origin/master 所在的 Commit 上，省去 git checkout 跟 git pull，分支從頭到尾停在 jeffrey-work 上，Rebase 以外的檔案異動次數為 - 0 次，很讚吧!

為了測試驗證我寫了以下腳本，反覆測試蠻方便的，一併筆記備忘。

```
rmdir /s /q demo
rmdir /s /q remote
mkdir remote
cd remote
git init --bare
cd ..
git clone ./remote ./demo
cd demo
touch init.txt
git add init.txt
git commit -m "建立專案"
ping -n 1 localhost > nul
git commit --allow-empty -m "PoC 完成"
ping -n 1 localhost > nul
git commit --allow-empty -m "第一次部署上線"
git tag v1.0
git push
git branch release-1
ping -n 1 localhost > nul
git checkout -b jeffrey-work
git commit --allow-empty -m "Jeffrey 修改1"
ping -n 1 localhost > nul
git commit --allow-empty -m "Jeffrey 修改2"
ping -n 2 localhost > nul
git checkout master 
git commit --allow-empty -m "Eric 修改1"
ping -n 1 localhost > nul
git commit --allow-empty -m "Eric 修改2"
ping -n 1 localhost > nul
git commit --allow-empty -m "Eric 修改3"
ping -n 1 localhost > nul
git commit --allow-empty -m "Eric 修改4"
ping -n 1 localhost > nul
touch eric-changes.txt
git add .
git commit --allow-empty -m "Eric 部署上線"
git tag v1.1
ping -n 2 localhost > nul
git push
git branch release-2
git reset --hard release-1
git checkout jeffrey-work
git commit --allow-empty -m "Jeffrey 修改3"
ping -n 1 localhost > nul
git commit --allow-empty -m "Jeffrey 修改4"
ping -n 1 localhost > nul
touch jeffrey-chages.txt
git add .
git commit --allow-empty -m "Jeffrey 修改(未完待續)"
git branch -d release-1
git branch -D release-2
cd ..
code .\demo
```

Tips of using git branch --force master origin/master to rebase in-process dev branch to master.