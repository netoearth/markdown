## 目錄

1.  [1.安裝系統時的眉角](https://ivonblog.com/posts/install-archlinux-on-laptop/#1%E5%AE%89%E8%A3%9D%E7%B3%BB%E7%B5%B1%E6%99%82%E7%9A%84%E7%9C%89%E8%A7%92)
2.  [2\. 筆電Intel內顯與Nvidia獨顯的切換](https://ivonblog.com/posts/install-archlinux-on-laptop/#2-%E7%AD%86%E9%9B%BBintel%E5%85%A7%E9%A1%AF%E8%88%87nvidia%E7%8D%A8%E9%A1%AF%E7%9A%84%E5%88%87%E6%8F%9B)
3.  [3\. 補齊驅動程式](https://ivonblog.com/posts/install-archlinux-on-laptop/#3-%E8%A3%9C%E9%BD%8A%E9%A9%85%E5%8B%95%E7%A8%8B%E5%BC%8F)
4.  [4\. MSI的風扇控制](https://ivonblog.com/posts/install-archlinux-on-laptop/#4-msi%E7%9A%84%E9%A2%A8%E6%89%87%E6%8E%A7%E5%88%B6)
5.  [4\. 結束](https://ivonblog.com/posts/install-archlinux-on-laptop/#4-%E7%B5%90%E6%9D%9F)

跟桌電比起來，筆電含有更多內建的硬體設備，所以要額外注意。過去困擾的就是內顯獨顯切換的問題，Arch Linux也不例外。 ![](https://i.imgur.com/bq2p3Vu.jpg)

工作用的筆電還敢裝Arch Linux啊？很勇嘛。對我來說，就算開機只有終端機畫面也大概知道怎麼修。但若是開會前掛掉就…所以要妥善安排更新時程XDD

Arch Linux提供的社群資源頗豐，缺韌體到AUR抓就好啦。

我的筆電是微星MSI Modern 15 A10RBS。CPU為Intel i5-10210u，GPU為Nvidia MX350。

以下的安裝過程大致上跟桌機差不多：[在主力電腦安裝Arch Linux](https://ivonblog.com/posts/install-archlinux/)

但有幾點不同，我特地列出來。

## [](https://ivonblog.com/posts/install-archlinux-on-laptop/#1%E5%AE%89%E8%A3%9D%E7%B3%BB%E7%B5%B1%E6%99%82%E7%9A%84%E7%9C%89%E8%A7%92)[1.安裝系統時的眉角](https://ivonblog.com/posts/install-archlinux-on-laptop/#contents:1%E5%AE%89%E8%A3%9D%E7%B3%BB%E7%B5%B1%E6%99%82%E7%9A%84%E7%9C%89%E8%A7%92)

首先，BIOS的Secure Boot是一定要關的。

在用fdisk切割硬碟的時候，筆電的NVME裝置代號不會是`/dev/sda`，而是`/dev/nvme0n1`。

因為Wifi網卡是Intel的，安裝系統前用Arch Linux開機隨身碟內附的`iwctl`工具連線就可以了。

在pacstrap的階段，參照[Arch Linux Wiki](https://wiki.archlinux.org/title/MSI_Modern_15_A11M)類似機型的建議，安裝`intel-ucode` 和`intel-media-driver`套件。`linux-firmware`套件也涵蓋了這臺筆電部份的驅動程式。

## [](https://ivonblog.com/posts/install-archlinux-on-laptop/#2-%E7%AD%86%E9%9B%BBintel%E5%85%A7%E9%A1%AF%E8%88%87nvidia%E7%8D%A8%E9%A1%AF%E7%9A%84%E5%88%87%E6%8F%9B)[2\. 筆電Intel內顯與Nvidia獨顯的切換](https://ivonblog.com/posts/install-archlinux-on-laptop/#contents:2-%E7%AD%86%E9%9B%BBintel%E5%85%A7%E9%A1%AF%E8%88%87nvidia%E7%8D%A8%E9%A1%AF%E7%9A%84%E5%88%87%E6%8F%9B)

剛裝好系統的時候顯示卡只有Intel的內顯啟動，參考[Wiki](https://wiki.archlinux.org/title/NVIDIA_Optimus)選一個切換Nvidia顯示卡的套件，例如安裝[optimus-manager](https://github.com/Askannz/optimus-manager)可以在內顯和獨顯間切換，以確保HDMI孔能輸出訊號。

1.  從AUR安裝`optimius-manager`：

<table><tbody><tr><td><pre tabindex="0"><code><span>1
</span></code></pre></td><td><pre tabindex="0"><code data-lang="bash"><span><span>yay -S optimius-manager
</span></span></code></pre></td></tr></tbody></table>

2.  如果之前是用`nvidia`套件安裝Nvidia驅動的話，要將其改成DKMS的版本，optimius-manager才會正常運作：

<table><tbody><tr><td><pre tabindex="0"><code><span>1
</span><span>2
</span></code></pre></td><td><pre tabindex="0"><code data-lang="bash"><span><span>sudo pacman -S nvidia-dkms
</span></span><span><span>sudo pacman -R nvidia
</span></span></code></pre></td></tr></tbody></table>

3.  安裝圖形界面`optimius-manager-qt`，會在系統匣顯示切換顯示卡的按鈕：

<table><tbody><tr><td><pre tabindex="0"><code><span>1
</span></code></pre></td><td><pre tabindex="0"><code data-lang="bash"><span><span>yay -S optimius-manager-qt
</span></span></code></pre></td></tr></tbody></table>

4.  設定開機啟動Optimius Manager服務：

<table><tbody><tr><td><pre tabindex="0"><code><span>1
</span></code></pre></td><td><pre tabindex="0"><code data-lang="bash"><span><span>sudo systemctl <span>enable</span> optimius-manager
</span></span></code></pre></td></tr></tbody></table>

## [](https://ivonblog.com/posts/install-archlinux-on-laptop/#3-%E8%A3%9C%E9%BD%8A%E9%A9%85%E5%8B%95%E7%A8%8B%E5%BC%8F)[3\. 補齊驅動程式](https://ivonblog.com/posts/install-archlinux-on-laptop/#contents:3-%E8%A3%9C%E9%BD%8A%E9%A9%85%E5%8B%95%E7%A8%8B%E5%BC%8F)

開機後，重裝kernel：

<table><tbody><tr><td><pre tabindex="0"><code><span>1
</span></code></pre></td><td><pre tabindex="0"><code data-lang="bash"><span><span>sudo pacman -S linux linux-headers
</span></span></code></pre></td></tr></tbody></table>

在那之後會顯示`possible missing firmware`字樣，把那些驅動名稱拿去Google，通常AUR都會有收。

## [](https://ivonblog.com/posts/install-archlinux-on-laptop/#4-msi%E7%9A%84%E9%A2%A8%E6%89%87%E6%8E%A7%E5%88%B6)[4\. MSI的風扇控制](https://ivonblog.com/posts/install-archlinux-on-laptop/#contents:4-msi%E7%9A%84%E9%A2%A8%E6%89%87%E6%8E%A7%E5%88%B6)

1.  安裝防止Intel CPU過熱的thermald，啟用開機服務：

<table><tbody><tr><td><pre tabindex="0"><code><span>1
</span><span>2
</span></code></pre></td><td><pre tabindex="0"><code data-lang="bash"><span><span>sudo pacman -S thermald
</span></span><span><span>sudo systemctl <span>enable</span> thermald
</span></span></code></pre></td></tr></tbody></table>

2.  安裝`cpupower`監看CPU頻率，還有`power-profiles-daemon`，這樣系統匣就會出現省電與效能模式的開關：

<table><tbody><tr><td><pre tabindex="0"><code><span>1
</span><span>2
</span></code></pre></td><td><pre tabindex="0"><code data-lang="bash"><span><span>sudo pacman -S cpupower power-profiles-daemon
</span></span><span><span>sudo systemctl <span>enable</span> cpupower
</span></span></code></pre></td></tr></tbody></table>

3.  最後是比較麻煩的按照溫度控制風扇轉速，我採用的是[Ice-Sealed Wyvern (ISW-Modern)](https://github.com/FaridZelli/ISW-Modern)，按照作者指示安裝。
    
4.  編輯`/etc/isw.conf`，參考[類似機型設定](https://wiki.archlinux.org/title/MSI_Modern_15_A11M)，在底部填入風扇轉速規則：
    

<table><tbody><tr><td><pre tabindex="0"><code><span> 1
</span><span> 2
</span><span> 3
</span><span> 4
</span><span> 5
</span><span> 6
</span><span> 7
</span><span> 8
</span><span> 9
</span><span>10
</span><span>11
</span><span>12
</span><span>13
</span><span>14
</span><span>15
</span><span>16
</span><span>17
</span><span>18
</span><span>19
</span><span>20
</span><span>21
</span><span>22
</span><span>23
</span><span>24
</span><span>25
</span><span>26
</span><span>27
</span><span>28
</span><span>29
</span><span>30
</span><span>31
</span><span>32
</span><span>33
</span></code></pre></td><td><pre tabindex="0"><code data-lang="bash"><span><span><span>[</span>15A10RBS<span>]</span>
</span></span><span><span><span># Modern 15 A10RBS-462TW</span>
</span></span><span><span><span>address_profile</span> <span>=</span> MSI_ADDRESS_DEFAULT
</span></span><span><span><span>fan_mode</span> <span>=</span> <span>140</span>
</span></span><span><span><span>battery_charging_threshold</span> <span>=</span> <span>100</span>
</span></span><span><span><span># CPU</span>
</span></span><span><span><span>cpu_temp_0</span> <span>=</span> <span>45</span>
</span></span><span><span><span>cpu_temp_1</span> <span>=</span> <span>55</span>
</span></span><span><span><span>cpu_temp_2</span> <span>=</span> <span>65</span>
</span></span><span><span><span>cpu_temp_3</span> <span>=</span> <span>75</span>
</span></span><span><span><span>cpu_temp_4</span> <span>=</span> <span>90</span>
</span></span><span><span><span>cpu_temp_5</span> <span>=</span> <span>95</span>
</span></span><span><span><span>cpu_fan_speed_0</span> <span>=</span> <span>50</span>
</span></span><span><span><span>cpu_fan_speed_1</span> <span>=</span> <span>58</span>
</span></span><span><span><span>cpu_fan_speed_2</span> <span>=</span> <span>65</span>
</span></span><span><span><span>cpu_fan_speed_3</span> <span>=</span> <span>72</span>
</span></span><span><span><span>cpu_fan_speed_4</span> <span>=</span> <span>80</span>
</span></span><span><span><span>cpu_fan_speed_5</span> <span>=</span> <span>85</span>
</span></span><span><span><span>cpu_fan_speed_6</span> <span>=</span> <span>100</span>
</span></span><span><span><span># GPU</span>
</span></span><span><span><span>gpu_temp_0</span> <span>=</span> <span>45</span>
</span></span><span><span><span>gpu_temp_1</span> <span>=</span> <span>60</span>
</span></span><span><span><span>gpu_temp_2</span> <span>=</span> <span>70</span>
</span></span><span><span><span>gpu_temp_3</span> <span>=</span> <span>82</span>
</span></span><span><span><span>gpu_temp_4</span> <span>=</span> <span>90</span>
</span></span><span><span><span>gpu_temp_5</span> <span>=</span> <span>93</span>
</span></span><span><span><span>gpu_fan_speed_0</span> <span>=</span> <span>50</span>
</span></span><span><span><span>gpu_fan_speed_1</span> <span>=</span> <span>55</span>
</span></span><span><span><span>gpu_fan_speed_2</span> <span>=</span> <span>65</span>
</span></span><span><span><span>gpu_fan_speed_3</span> <span>=</span> <span>72</span>
</span></span><span><span><span>gpu_fan_speed_4</span> <span>=</span> <span>80</span>
</span></span><span><span><span>gpu_fan_speed_5</span> <span>=</span> <span>85</span>
</span></span><span><span><span>gpu_fan_speed_6</span> <span>=</span> <span>100</span>
</span></span></code></pre></td></tr></tbody></table>

5.  寫入EC，啟動ISW服務，並設定開機自動啟動ISW服務。不確定是否有啟動的話用`sudo systemctl status isw@15A10RBS.service`確認。

<table><tbody><tr><td><pre tabindex="0"><code><span>1
</span><span>2
</span><span>3
</span></code></pre></td><td><pre tabindex="0"><code data-lang="bash"><span><span>sudo isw -w 15A10RBS
</span></span><span><span>sudo systemctl start isw@15A10RBS.service
</span></span><span><span>sudo systemctl <span>enable</span> isw@15A10RBS.service
</span></span></code></pre></td></tr></tbody></table>

## [](https://ivonblog.com/posts/install-archlinux-on-laptop/#4-%E7%B5%90%E6%9D%9F)[4\. 結束](https://ivonblog.com/posts/install-archlinux-on-laptop/#contents:4-%E7%B5%90%E6%9D%9F)

筆電闔上螢幕進入睡眠通常會睡死，桌面環境不論GNOME或KDE皆然。這個要查看dmesg然後排除問題…通常我是在系統設定將它關掉。

這樣就完成了，麥克風、相機都正常運作。

[Linux](https://ivonblog.com/tags/linux/) [Arch Linux](https://ivonblog.com/tags/arch-linux/)