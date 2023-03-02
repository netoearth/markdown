Linux 寫一隻 Daemon，想要讓此程式死掉時，會自動啟動，systemd 有內建的方法可以做。

寫在 service 裡面，此程式用 systemctl start 後，此 Process 再怎麼 kill 都會自動啟動，直到 systemctl stop 才會停止。

在此寫的是 systemd service 一個簡單的範例，只要是 Restart=always 這個參數，就可以達到自動啟動的功效。

systemd service 範例與執行

1.  $ sudo vim /etc/systemd/system/example.service
    
    ```
    [Unit]
    Description=example daemon
    
    [Service]
    Type=simple
    WorkingDirectory=/usr/local/
    #ExecStart=/usr/local/go/bin/go run src/test1/main.go
    ExecStart=/usr/local/bin/example
    Restart=always
    
    [Install]
    WantedBy=multi-user.target
    ```
    
2.  若 /etc/systemd/system/example.service 有修改，需要執行下述才可以繼續 start / stop  
    -   $ sudo systemctl daemon-reload
3.  sudo systemctl start example.service # 手動啟動
4.  sudo systemctl stop example.service # 手動關閉
5.  sudo systemctl enable example.service # 開機自動啟動
6.  sudo systemctl disable example.service # 取消開機自動啟動
7.  sudo systemctl daemon-reload && systemctl enable example.service && systemctl start example.service # 連在一起~

## 文章導覽