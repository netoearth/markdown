Linux 啟動、結束程式或程式自動重啟等等，都可以使用 systemd 來處理，但是遇到下述的問題要怎麼解決呢？

-   11月 21 12:04:09 w1 systemd\[1\]: monitor\_error.service: Start request repeated too quickly.
-   11月 21 12:04:09 w1 systemd\[1\]: monitor\_error.service: Failed with result 'start-limit-hit'.
-   "start-limit-hit"
    -   A start limit was defined for the unit and it was hit, causing the unit to fail to start.
    -   See systemd.unit(5)'s StartLimitIntervalSec= and StartLimitBurst= for details.

問題發生主要是自己寫的一隻 script，然後想要自動執行，執行後若有問題會需要自動重新啟動，參考此篇：[Linux systemd 寫 可自動啟動的 Daemon Service](https://blog.longwin.com.tw/2018/02/linux-systemd-auto-start-daemon-service-2018/)。

遇到 Start request repeated too quickly 和 start-limit-hit 的問題，查過下述一些文件與設定：

-   [systemd.exec](https://www.freedesktop.org/software/systemd/man/systemd.exec.html#%24SERVICE_RESULT)
-   man 5 systemd.unit
-   man 5 systemd.service
    -   \[Unit\]
    -   Description=foobar-test
    -   StartLimitBurst=3
    -   StartLimitIntervalSec=120

最後解決的方式是把 shell script 程式裡面有 & (丟入背景執行)，這會等同程式結束，然後程式結束後，又自動啟動，於是造成快速的重複啟動，於是遇到快速重啟而且很多次(超過 start-limit-hit 限制 )

-   解法就是把「**shell script 裡面的 & 拿掉**」就可以了 (**程式不要丟背景執行，就不會結束，程式要不要結束由 systemd 來控制**)

### 相關網頁

-   [start request repeated too quickly](https://stackoverflow.com/questions/35452591/start-request-repeated-too-quickly)
-   [How to fail service so that SERVICE\_RESULT contains "start-limit-hit"](https://unix.stackexchange.com/questions/721448/how-to-fail-service-so-that-service-result-contains-start-limit-hit)

## 文章導覽