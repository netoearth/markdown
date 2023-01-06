在 Linux 想要刪除檔案最前面、最後面，或者中間幾行的內容，可以怎麼做呢？

## Linux 使用 SED 對檔案 最前、最後一行資料

sed 要刪除檔案第幾行內容

-   sed -i '1d' filename # 刪除第一行
-   sed -i '2d' filename # 刪除第二行

sed 要刪除區間幾行的內容

-   sed -i '1,2d' filename # 刪除第1~2行
-   sed -i '2,4d' filename # 刪除第2~4行

sed 要刪除檔案最後一行的內容

-   sed -i '$d' filename # 最後一行
-   sed -i '$d;$d' filename # 刪除最後兩行的內容

### 相關網頁

-   [sed - How to insert text before the first line of a file?](https://unix.stackexchange.com/questions/99350/how-to-insert-text-before-the-first-line-of-a-file)

### _相關_

#### [Git 初學筆記 - 指令操作教學](https://blog.longwin.com.tw/2009/05/git-learn-initial-command-2009/?relatedposts_hit=1&relatedposts_origin=14086&relatedposts_position=1 "Git 初學筆記 - 指令操作教學")

Git 是分散式的版本控制系統, 從架設、簡易操作、設定, 此篇主要是整理 基本操作、遠端操作 等.…

2009 年 05 月 19 日

在「My\_Note-Unix」中

#### [SVN 基本指令教學](https://blog.longwin.com.tw/2007/07/svn_tutorial_2007/?relatedposts_hit=1&relatedposts_origin=14086&relatedposts_position=2 "SVN 基本指令教學")

SVN 的基本指令介紹, 主要參考自下述: SubTrain - Open Source Train…

2007 年 07 月 10 日

在「My\_Note-Programming」中

![](https://secure.gravatar.com/avatar/b636a9bf35d30f7781db075f241e6fb2?s=42&d=mm&r=g)

## 文章導覽