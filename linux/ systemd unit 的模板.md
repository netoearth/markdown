

如果需要设置成系统服务的话，这里提供一个 systemd unit 的模板作为参考：

```
[Unit]
Description=kiwix-serve Service
Wants=network-online.target
After=network.target

[Service]
Type=simple
Restart=on-failure
RestartSec=3
WorkingDirectory=/var/lib/kiwix/
ExecStart=/usr/local/bin/kiwix-serve --library /var/lib/kiwix/library.xml --port 9000
StandardOutput=journal
StandardError=null

# Hardening
DynamicUser=true
MemoryDenyWriteExecute=true
NoNewPrivileges=true

[Install]
WantedBy=multi-user.target




