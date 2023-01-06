查看時區的設定為何?  
**\# timedatectl**

設定目前"日期":  
**\# timedatectl set-time <YYYY-MM-DD>**

設定目前"時間":  
**\# timedatectl set-time <HH:MM:SS>**

啟用UTC時間:  
**\# timedatectl set-local-rtc no**

關閉UTC時間，使用本地時間:  
**\# timedatectl set-local-rtc yes**

查看所有時區:  
**\# timedatectl list-timezones**

設定時區區域:  
**\# timedatectl set-timezone Asia/Taipei**

與NTP server同步:  
**\# timedatectl set-ntp yes**

與NTP Server同步後再寫到Hardware Clock:  
**\# ntpdate -s time.stdtime.gov.tw  
\# hwclock --systohc**

引用至: http://www.tianwaihome.com/2014/08/centos-7-set-local.html