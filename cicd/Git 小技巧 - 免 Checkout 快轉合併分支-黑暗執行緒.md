<table><tbody><tr><td><img src="https://blog.darkthread.net/img/calendar.svg"></td><td><span title="Published at 2022-10-13 08:54 PM"><time datetime="2022-10-13T12:54:24" itemprop="datePublished">2022-10-13 08:54 PM</time></span></td><td><img src="https://blog.darkthread.net/img/comment.svg"></td><td><span title="2 comments">2</span></td><td><img src="https://blog.darkthread.net/img/eye.svg"></td><td><span data-ajax-url="/blog/pageviewcount/git-merge-wo-checkout" title="476 pageviews">476</span></td><td><a href="https://blog.darkthread.net/Blog/git-merge-wo-checkout/"><img src="data:image/gif;base64,R0lGODlhAQABAIAAAP///wAAACH5BAEAAAAALAAAAAABAAEAAAICRAEAOw==" id="nllbtn"></a></td></tr></tbody></table>

我習慣開發分支合併到主分支前先做 Rebase 再合併 (延伸閱讀：[直接合併 vs 先 Rebase 再合併](https://blog.darkthread.net/blog/conflict-resolving-in-git-rebase/))。舉例來說，假設我從 master 開了 featureX 分支開發，現在要合併回 master：

![](https://blog.darkthread.net/Posts/files/Fig1_638012626407976336.png)

我會先從 featureX 分支 `git rebase master` 將 featureX 分支接到 master 後面：

![](https://blog.darkthread.net/Posts/files/Fig2_638012626409861357.png)

一般來說，合併的標準做法是先 checkout 到要併入的目標分支，再下指令 merge。因此，我會切回 master，再下指令 git merge featureX：

```
git checkout master
git merge featureX
```

![](https://blog.darkthread.net/Posts/files/Fig4_638012626412030560.png)

如上圖所示，切換 master 再合併 featureX 雖屬快轉合併(Fast-Forward，分支移到新的 Commit 位置即可，不產生合併線)，但檔案狀態會先從 featureX 還原到 master 狀態(在本例是移除 featureX 新增的 c1.txt 及 c2.txt 兩個檔案)，合併時再把 c1.txt 及 c2.txt 加回去。

有沒有可能省掉這段無意義的「刪掉檔案再加回去」程序？

由前幾天分享 [開發分支快速 Rebase 主分支技巧](https://blog.darkthread.net/blog/rebase-master-efficently/)，從讀者 Peter Dave Hello 及馬楊陞的留言，我學到新技巧(特此感謝)。原來，fetch 指令除了取回遠端內容，還有更新分支的功能，其語法為：(<src>:<dst> 是所謂 [refspec](https://git-scm.com/docs/git-fetch#Documentation/git-fetch.txt-ltrefspecgt)，想像成將 repository 的 src 更新到本地的 dst)

`git fetch <repository> <src>:<dst>`

例如：`git fetch orign master:master` 即是取回遠端 origin/master 更新到本機 master，可省下 pull 或 merge 動作。而這組指令，也能實現我們想到的「本機不切換分支直接合併」。以上述案例，用 . 當成 repository，使用指令 `git fetch . featureX:master` 便能在不 Checkout 切換到 master 的狀況下抓取 featureX 的內容更新到 master，達到 master 合併 featureX 的相同結果，但省掉切換 master 刪除檔案，merge 時再還原的過程，快速又方便。

![](https://blog.darkthread.net/Posts/files/Fig5_638012626415871474.gif)

不過，這招只適用快轉(Fast-Forward)模式的合併，如果不是，會被狠狠拒絕。(因此，上回提過的 [git branch --force](https://blog.darkthread.net/blog/rebase-master-efficently/) 雖然也能實現相同效果，但操作失誤風險高，用 fetch 較讓人放心)

![](https://blog.darkthread.net/Posts/files/Fig3_638012626417805167.png)

學會這招，未來應該靠它省下不少無謂的 checkout + merge 檔案異動，讚!!

Tips of how to merge a branch without checkout it first.