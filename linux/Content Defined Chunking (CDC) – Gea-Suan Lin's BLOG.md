前幾個禮拜在 [Hacker News Daily](https://www.daemonology.net/hn-daily/) 上看到「[CDC File Transfer (github.com/google)](https://news.ycombinator.com/item?id=34303497)」這則，連結是指到 [Google](https://www.google.com/) 的 [GitHub](https://github.com/) 專案上，裡面實做了 [FastCDC](https://www.usenix.org/conference/atc16/technical-sessions/presentation/xia) 演算法，另外說明他們為什麼要解這個問題以及對應的成果：「[google/cdc-file-transfer](https://github.com/google/cdc-file-transfer)」。

Google 的人看起來像是是在 CI/CD 階段遇到頻寬上的問題 (從「The builds are 40-45 GB large.」這邊猜)，用 [scp](https://en.wikipedia.org/wiki/Secure_copy_protocol) 與 [rsync](https://en.wikipedia.org/wiki/Rsync) 看起來都不能解，所以他們自己刻了 FastCDC 演算法來解。

但我對 Content Defined Chunking (CDC) 不熟，所以先查一下 CDC 是什麼東西，就查到 [restic](https://restic.net/) 這篇講得很清楚：「[Foundation - Introducing Content Defined Chunking (CDC)](https://restic.net/blog/2015-09-12/restic-foundation1-cdc/)」。

要計算 delta 很直覺的作法就是要切 chunk，而接著的直覺就是固定大小的 chunk 切開，像是這樣每 16 bytes 切一個 chunk：

```
0123456789abcdef 0123456789abcdef 0123456789abcdef 0123456789abcdef
```

如果其中一個地方有變化，但其他沒變化的話就可以透過 cryptographic hash function (像是 [SHA-256](https://en.wikipedia.org/wiki/SHA-256)) 確認 chunk 內容一樣，進而省下很多傳輸的頻寬：

```
0123456789abcdef 0123456789ABCDEF 0123456789abcdef 0123456789abcdef
```

但可以馬上看出來這個方法的大缺點是只能處理 replacement，很難處理 insert & delete 的部份，舉例來說，如果變更是在開頭的地方加上 ABC，就會造成 chunk 會完全不一樣，而導致全部都要再傳一次：

```
ABC0123456789abc def0123456789abc def0123456789abc def0123456789abc def
```

這邊其實是個經典的演算法問題：想要找出兩個 string 的差異 (把舊的內容當作一個 string，新的內容也當作一個 string)。

這個問題算是 [Edit distance](https://en.wikipedia.org/wiki/Edit_distance) 類型的題目，但你會發現解 Edit distance 的演算法會需要先傳輸完整個 string 才能開始跑演算法，這就本末倒置了。

而另外一個想法是，放棄固定的 chunk 大小，改用其他方式決定 chunk 的邊界要切在哪裡。而 CDC 就是利用一段 sliding window + hash 來找出切割的點。

文章裡面提到的 sliding window 是 64 bytes，這邊就可以算出對應的 `HASH(b0, b1, ..., b63)`，然後往右滑動變成 `HASH(b1, b2, ..., b64)`，再來是 `HASH(b2, b3, ..., b65)`，一直往右滑動計算。

接下來 restic 會看 hash 值，如果最低的 21 bits 都是 0 就切開，所以 chunk 大小的期望值應該是 2MB？(這邊不確定，好像不能直接用 2^21 算，應該用積分之類的方法...)

> For each fingerprint, restic then tests if the lowest 21 bits are zero. If this is the case, restic found a new chunk boundary.

而這個演算法可以適應新增與刪除的操作，不會造成從新增或刪除後的資料都要重傳，只有自己這個 chunk 需要重傳 (可能前或後的 chunk 也會要)。

然後挑一下 hash function 的特性，就可以讓計算的速度也很快。這邊提到了 hash function 可以用 [Rolling hash](https://en.wikipedia.org/wiki/Rolling_hash)，可以很快的從 `HASH(b0, b1, ..., b63)` 算出 `HASH(b1, b2, ..., b64)`，而不需要全部重算。

有了 chunk 後，再用 cryptographic hash function 比較 chunk 的內容是否一樣，這樣就可以大幅降低傳輸所需要的頻寬了。

## Post navigation