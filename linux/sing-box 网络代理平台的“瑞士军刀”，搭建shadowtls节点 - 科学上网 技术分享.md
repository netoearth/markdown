说起网络代理工具大家第一时间想到的应该是v2ray和clash，最近又有另一个非常火的网络代理工具出现了，sing-box，和v2ray属于同一种类型的工具，但是sing-box对协议支持的丰富程度可以说无人能及，本期教程就来带大家简单上手sing-box，使用sing-box搭建一个非常骚的shadowtls节点

## 视频教程

<iframe src="https://www.youtube.com/embed/o-IFEu4GENE" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen=""></iframe>

___

## 安装sing-box

官方文档：[https://sing-box.sagernet.org/](https://bulianglin.com/g/aHR0cHM6Ly9zaW5nLWJveC5zYWdlcm5ldC5vcmcv)  
下载sing-box：`wget https://github.com/SagerNet/sing-box/releases/download/v1.0.4/sing-box_1.0.4_linux_amd64.deb`

> 最新版请在项目官方地址获取：[https://github.com/SagerNet/sing-box/releases/latest](https://bulianglin.com/g/aHR0cHM6Ly9naXRodWIuY29tL1NhZ2VyTmV0L3NpbmctYm94L3JlbGVhc2VzL2xhdGVzdA)

安装sing-box：`dpkg -i sing-box_1.0.4_linux_amd64.deb`

查看安装包内容：`dpkg -c sing-box_1.0.4_linux_amd64.deb`

修改配置文件：`vim /etc/sing-box/config.json`

查看sing-box状态：`systemctl status sing-box.service`

启动sing-box：`systemctl start sing-box.service`

## sing-box服务端shadowtls配置

```
{
  "inbounds": [
    {
      "type": "shadowtls",
      "listen_port": 443,
      "handshake": {
        "server": "www.bing.com",
        "server_port": 443 
      },
      "detour": "shadowsocks-in"
    },
    {
      "type": "shadowsocks",
      "tag": "shadowsocks-in",
      "listen": "127.0.0.1",
      "method": "2022-blake3-aes-128-gcm",
      "password": "8JCsPssfgS8tiRwiMlhARg=="
    }
  ]
}
```

## sing-box客户端shadowtls配置

### socks/http代理模式

```
{
  "inbounds": [
    {
      "type": "mixed",
      "listen_port": 1080,
      "sniff": true,
      "set_system_proxy": true
    }
  ],
  "outbounds": [
    {
      "type": "shadowsocks",
      "method": "2022-blake3-aes-128-gcm",
      "password": "8JCsPssfgS8tiRwiMlhARg==",
      "detour": "shadowtls-out",
      "multiplex": {
        "enabled": true,
        "max_connections": 4,
        "min_streams": 4
      }
    },
    {
      "type": "shadowtls",
      "tag": "shadowtls-out",
      "server": "填写服务器ip地址",
      "server_port": 443,
      "tls": {
        "enabled": true,
        "server_name": "www.bing.com"
      }
    },
    {
      "type": "direct",
      "tag": "direct"
    },
    {
      "type": "block",
      "tag": "block"
    }
  ],
  "route": {
    "rules": [
      {
        "geosite": "category-ads-all",
        "outbound": "block"
      },
      {
        "geosite": "cn",
        "geoip": "cn",
        "outbound": "direct"
      }
    ]
  }
}
```

### TUN模式

```
{
  "dns": {
    "servers": [
      {
        "tag": "google",
        "address": "tls://8.8.8.8"
      },
      {
        "tag": "local",
        "address": "223.5.5.5",
        "detour": "direct"
      }
    ],
    "rules": [
      {
        "geosite": "cn",
        "server": "local"
      }
    ],
    "strategy": "ipv4_only"
  },
  "inbounds": [
    {
      "type": "tun",
      "inet4_address": "172.19.0.1/30",
      "auto_route": true,
      "sniff": true
    }
  ],
  "outbounds": [
    {
      "type": "shadowsocks",
      "method": "2022-blake3-aes-128-gcm",
      "password": "8JCsPssfgS8tiRwiMlhARg==",
      "detour": "shadowtls-out",
      "multiplex": {
        "enabled": true,
        "max_connections": 4,
        "min_streams": 4
      }
    },
    {
      "type": "shadowtls",
      "tag": "shadowtls-out",
      "server": "填写服务器ip地址",
      "server_port": 443,
      "tls": {
        "enabled": true,
        "server_name": "www.bing.com"
      }
    },
    {
      "type": "direct",
      "tag": "direct"
    },
    {
      "type": "block",
      "tag": "block"
    },
    {
      "type": "dns",
      "tag": "dns-out"
    }
  ],
  "route": {
    "rules": [
      {
        "protocol": "dns",
        "outbound": "dns-out"
      },
      {
        "geosite": "category-ads-all",
        "outbound": "block"
      },
      {
        "geosite": "cn",
        "geoip": "cn",
        "outbound": "direct"
      }
    ],
    "auto_detect_interface": true
  }
}
```