## 目錄

1.  [1\. 製作Arch Linux開機隨身碟](https://ivonblog.com/posts/install-archlinux/#1-%E8%A3%BD%E4%BD%9Carch-linux%E9%96%8B%E6%A9%9F%E9%9A%A8%E8%BA%AB%E7%A2%9F)
2.  [2\. 開始安裝Arch Linux](https://ivonblog.com/posts/install-archlinux/#2-%E9%96%8B%E5%A7%8B%E5%AE%89%E8%A3%9Darch-linux)
3.  [2.1. 開機進入Arch Linux安裝媒體](https://ivonblog.com/posts/install-archlinux/#21-%E9%96%8B%E6%A9%9F%E9%80%B2%E5%85%A5arch-linux%E5%AE%89%E8%A3%9D%E5%AA%92%E9%AB%94)
    1.  [2.2. 分割硬碟](https://ivonblog.com/posts/install-archlinux/#22-%E5%88%86%E5%89%B2%E7%A1%AC%E7%A2%9F)
    2.  [2.3. 安裝系統和桌面環境](https://ivonblog.com/posts/install-archlinux/#23-%E5%AE%89%E8%A3%9D%E7%B3%BB%E7%B5%B1%E5%92%8C%E6%A1%8C%E9%9D%A2%E7%92%B0%E5%A2%83)
    3.  [2.4. 設定一般使用者與開機服務](https://ivonblog.com/posts/install-archlinux/#24-%E8%A8%AD%E5%AE%9A%E4%B8%80%E8%88%AC%E4%BD%BF%E7%94%A8%E8%80%85%E8%88%87%E9%96%8B%E6%A9%9F%E6%9C%8D%E5%8B%99)
    4.  [2.5. 安裝GRUB開機選單](https://ivonblog.com/posts/install-archlinux/#25-%E5%AE%89%E8%A3%9Dgrub%E9%96%8B%E6%A9%9F%E9%81%B8%E5%96%AE)
4.  [3\. 系統後續安裝優化](https://ivonblog.com/posts/install-archlinux/#3-%E7%B3%BB%E7%B5%B1%E5%BE%8C%E7%BA%8C%E5%AE%89%E8%A3%9D%E5%84%AA%E5%8C%96)
    1.  [3.1. KDE桌面入門](https://ivonblog.com/posts/install-archlinux/#31-kde%E6%A1%8C%E9%9D%A2%E5%85%A5%E9%96%80)
    2.  [3.2. 安裝Fcitx5注音輸入法](https://ivonblog.com/posts/install-archlinux/#32-%E5%AE%89%E8%A3%9Dfcitx5%E6%B3%A8%E9%9F%B3%E8%BC%B8%E5%85%A5%E6%B3%95)
    3.  [3.3. 安裝AUR套件管理員](https://ivonblog.com/posts/install-archlinux/#33-%E5%AE%89%E8%A3%9Daur%E5%A5%97%E4%BB%B6%E7%AE%A1%E7%90%86%E5%93%A1)
    4.  [3.4. 安裝UFW防火牆](https://ivonblog.com/posts/install-archlinux/#34-%E5%AE%89%E8%A3%9Dufw%E9%98%B2%E7%81%AB%E7%89%86)
    5.  [3.5. 安裝Flatpak](https://ivonblog.com/posts/install-archlinux/#35-%E5%AE%89%E8%A3%9Dflatpak)
5.  [4\. Arch Linux安裝後記](https://ivonblog.com/posts/install-archlinux/#4-arch-linux%E5%AE%89%E8%A3%9D%E5%BE%8C%E8%A8%98)

看上Arch Linux的優點：系統安裝具高自訂性，要簡潔要複雜隨你所願。還有Arch Linux提供的軟體和Linux核心版本非常新，亦有大量第三方軟體庫作為補充。 ![](https://i.imgur.com/kB3rFY5.jpg)

這篇文章教學供想安裝Arch Linux的人當參考，你將會得到一個正體中文，帶有中文輸入法，以KDE Plasma作桌面環境的Arch Linux。

每個步驟會盡量解釋我在做什麼。建議對Linux系統熟悉之後再嘗試，因為Arch Linux是完全純文字畫面打指令安裝的。本文不會使用任何圖形安裝器或archinstall之類的快速安裝指令稿。

如有其他問題，一切參照[Arch Linux Wiki](https://wiki.archlinux.org/title/installation_guide)說明，該頁面有中文版翻譯。

電腦規格：

|  |  |
| --- | --- |
| 主機板 | ASUS K31CD-K |
| CPU | Intel I5-7400 |
| GPU | Nvidia GTX1050Ti |
| RAM | 16GB DDR4 |
| SSD | Micron 500GB |
| Wifi網路卡 | Qualcomm Atheros QCA9377 |

## [](https://ivonblog.com/posts/install-archlinux/#1-%E8%A3%BD%E4%BD%9Carch-linux%E9%96%8B%E6%A9%9F%E9%9A%A8%E8%BA%AB%E7%A2%9F)[1\. 製作Arch Linux開機隨身碟](https://ivonblog.com/posts/install-archlinux/#contents:1-%E8%A3%BD%E4%BD%9Carch-linux%E9%96%8B%E6%A9%9F%E9%9A%A8%E8%BA%AB%E7%A2%9F)

1.  至[Arch Linux官網](https://archlinux.org/download/)，找到臺灣的鏡像站。 ![](https://i.imgur.com/2kQjGSp.png)
    
2.  下載Arch Linux的ISO。 ![](https://i.imgur.com/ZtPwxMm.png)
    
3.  安裝[balenaEtcher](https://www.balena.io/etcher/)，插入隨身碟，按照螢幕指示製作開機隨身碟。隨身碟的資料會被清空。
    

## [](https://ivonblog.com/posts/install-archlinux/#2-%E9%96%8B%E5%A7%8B%E5%AE%89%E8%A3%9Darch-linux)[2\. 開始安裝Arch Linux](https://ivonblog.com/posts/install-archlinux/#contents:2-%E9%96%8B%E5%A7%8B%E5%AE%89%E8%A3%9Darch-linux)

安裝系統請使用有線網路，用手機USB分享網路也可以，免得Wifi還要另外裝驅動。

## [](https://ivonblog.com/posts/install-archlinux/#21-%E9%96%8B%E6%A9%9F%E9%80%B2%E5%85%A5arch-linux%E5%AE%89%E8%A3%9D%E5%AA%92%E9%AB%94)[2.1. 開機進入Arch Linux安裝媒體](https://ivonblog.com/posts/install-archlinux/#contents:21-%E9%96%8B%E6%A9%9F%E9%80%B2%E5%85%A5arch-linux%E5%AE%89%E8%A3%9D%E5%AA%92%E9%AB%94)

1.  隨身碟插著電腦，開機按Delete進入BIOS，關閉Secure Boot。
    
2.  調整BIOS開機順序，以**UEFI模式**隨身碟開機，進入Arch Linux，用鍵盤選第一個選項，按Enter進入安裝媒體。
    
3.  載入Arch Linux系統後會進入終端機(顯示`root@archiso`)，系統應該會自動連上網路。ping到Arch Linux官網看看有無回傳封包:
    

<table><tbody><tr><td><pre tabindex="0"><code><span>1
</span></code></pre></td><td><pre tabindex="0"><code data-lang="bash"><span><span>ping -c <span>3</span> archlinux.org
</span></span></code></pre></td></tr></tbody></table>

### [](https://ivonblog.com/posts/install-archlinux/#22-%E5%88%86%E5%89%B2%E7%A1%AC%E7%A2%9F)[2.2. 分割硬碟](https://ivonblog.com/posts/install-archlinux/#contents:22-%E5%88%86%E5%89%B2%E7%A1%AC%E7%A2%9F)

我將使用`fdisk`指令進行硬碟分割，這裡只有一個硬碟，將原本的分割表刪除，全用於安裝Arch Linux。

1.  首先查看目前的硬碟分區

2.  應會看到`/dev/sda`這樣的裝置代號。SSD或HDD通常是顯示為`/dev/sda`，NVME為`/dev/nvme0n1`。

<table><tbody><tr><td><pre tabindex="0"><code><span>1
</span><span>2
</span></code></pre></td><td><pre tabindex="0"><code data-lang="bash"><span><span>/dev/sda1  EFI System
</span></span><span><span>/dev/sda2  Linux filesystem
</span></span></code></pre></td></tr></tbody></table>

3.  進入該硬碟

4.  輸入`g`刪除全部分區，建立GPT分割表。

5.  新增用於UEFI開機的第一個硬碟分區。輸入`n`，再輸入`1`，First Sector使用預設，Last Sector輸入`+512M`將該分區設為512MB。

<table><tbody><tr><td><pre tabindex="0"><code><span>1
</span><span>2
</span><span>3
</span><span>4
</span><span>5
</span></code></pre></td><td><pre tabindex="0"><code data-lang="bash"><span><span>n
</span></span><span><span><span>1</span>
</span></span><span><span>First Sector: <span>(</span>Enter<span>)</span>
</span></span><span><span>Last Sector: +512M
</span></span><span><span><span># 遇到Do you want to remove the signature?的問題就輸入yes</span>
</span></span></code></pre></td></tr></tbody></table>

6.  輸入`t`再輸入`uefi`，將該分區類型切換為EFI：

7.  硬碟剩下的空間都給Linux的根目錄。新增第二個硬碟分區：

<table><tbody><tr><td><pre tabindex="0"><code><span>1
</span><span>2
</span><span>3
</span><span>4
</span></code></pre></td><td><pre tabindex="0"><code data-lang="bash"><span><span>n
</span></span><span><span><span>2</span>
</span></span><span><span>First Sector: <span>(</span>Enter<span>)</span>
</span></span><span><span>Last Sector: <span>(</span>Enter<span>)</span>
</span></span></code></pre></td></tr></tbody></table>

8.  最後輸入`w`確認將變更寫入硬碟：

9.  用指令`fdisk -l`看目前硬碟分區，應該會是以下情況：

<table><tbody><tr><td><pre tabindex="0"><code><span>1
</span><span>2
</span></code></pre></td><td><pre tabindex="0"><code data-lang="bash"><span><span>/dev/sda1 <span>(</span>EFI<span>)</span>
</span></span><span><span>/dev/sda2 <span>(</span>Linux<span>)</span>
</span></span></code></pre></td></tr></tbody></table>

10.  將硬碟格式化並建立檔案系統，EFI分區是FAT32，根目錄分區採用BTRFS。

<table><tbody><tr><td><pre tabindex="0"><code><span>1
</span><span>2
</span></code></pre></td><td><pre tabindex="0"><code data-lang="bash"><span><span>mkfs.fat -F32 /dev/sda1
</span></span><span><span>mkfs.btrfs /dev/sda2
</span></span></code></pre></td></tr></tbody></table>

11.  將根目錄分區掛載至`/mnt`：

### [](https://ivonblog.com/posts/install-archlinux/#23-%E5%AE%89%E8%A3%9D%E7%B3%BB%E7%B5%B1%E5%92%8C%E6%A1%8C%E9%9D%A2%E7%92%B0%E5%A2%83)[2.3. 安裝系統和桌面環境](https://ivonblog.com/posts/install-archlinux/#contents:23-%E5%AE%89%E8%A3%9D%E7%B3%BB%E7%B5%B1%E5%92%8C%E6%A1%8C%E9%9D%A2%E7%92%B0%E5%A2%83)

1.  用`pacstrap`安裝基本Linux檔案系統`base`、核心`linux`、專有韌體`linux-firmware`:

<table><tbody><tr><td><pre tabindex="0"><code><span>1
</span></code></pre></td><td><pre tabindex="0"><code data-lang="bash"><span><span>pacstrap /mnt base linux linux-firmware
</span></span></code></pre></td></tr></tbody></table>

2.  設定開機後硬碟掛載的規則:

<table><tbody><tr><td><pre tabindex="0"><code><span>1
</span><span>2
</span></code></pre></td><td><pre tabindex="0"><code data-lang="bash"><span><span>mount /dev/sda1 /mnt/boot
</span></span><span><span>genfstab -U /mnt &gt;&gt; /mnt/etc/fstab
</span></span></code></pre></td></tr></tbody></table>

3.  chroot到系統。

4.  接著要來安裝系統軟體和桌面環境。下面是我事先安裝這些軟體的理由：
    
    |  |  |
    | --- | --- |
    | sudo | 管理超級使用者權限 |
    | networkmanager | 網路管理員，會自動處理網路連線。 |
    | xorg, xorg-server | X視窗系統 |
    | pipewire-pulse | 音訊伺服器 |
    | nvidia-dkms, nvidia-settings, nvtop | Nvidia顯卡專有驅動及相關工具 |
    | intel-ucode | Intel CPU微碼修補程式 |
    | intel-media-driver | Intel顯示卡驅動 |
    | sddm | 顯示管理器，也就是開機登入畫面 |
    | plasma-meta, kde-applications | KDE桌面環境和KDE應用程式 |
    | vim | 文字編輯器 |
    | firefox | Firefox網路瀏覽器 |
    | noto-fonts-cjk | 中文字體 |
    | fcitx5 | 中文輸入法框架，含注音輸入法新酷音 |
    | git, openssh, fakeroot, base-devel | 開發工具 |
    
5.  用pacman套件管理員開始安裝KDE桌面環境和常用軟體，全部Enter同意，大約會下載5GB資料。
    

<table><tbody><tr><td><pre tabindex="0"><code><span>1
</span><span>2
</span><span>3
</span><span>4
</span><span>5
</span></code></pre></td><td><pre tabindex="0"><code data-lang="bash"><span><span>pacman -S sudo networkmanager vim firefox noto-fonts-cjk
</span></span><span><span>pacman -S xorg xorg-server pipewire-pulse nvidia nvidia-settings nvtop intel-ucode intel-media-driver
</span></span><span><span>pacman -S sddm plasma-meta kde-applications
</span></span><span><span>pacman -S fcitx5-im fcitx5-chewing fcitx5-qt fcitx5-gtk fcitx5-chinese-addons
</span></span><span><span>pacman -S git openssh fakeroot base-devel
</span></span></code></pre></td></tr></tbody></table>

### [](https://ivonblog.com/posts/install-archlinux/#24-%E8%A8%AD%E5%AE%9A%E4%B8%80%E8%88%AC%E4%BD%BF%E7%94%A8%E8%80%85%E8%88%87%E9%96%8B%E6%A9%9F%E6%9C%8D%E5%8B%99)[2.4. 設定一般使用者與開機服務](https://ivonblog.com/posts/install-archlinux/#contents:24-%E8%A8%AD%E5%AE%9A%E4%B8%80%E8%88%AC%E4%BD%BF%E7%94%A8%E8%80%85%E8%88%87%E9%96%8B%E6%A9%9F%E6%9C%8D%E5%8B%99)

1.  設定時區為台灣台北。

<table><tbody><tr><td><pre tabindex="0"><code><span>1
</span></code></pre></td><td><pre tabindex="0"><code data-lang="bash"><span><span>ln -sf /usr/share/zoneinfo/Asia/Taipei /etc/localtime
</span></span></code></pre></td></tr></tbody></table>

2.  使用VIM編輯語言檔案

3.  把正體中文`zh_TW.UTF-8`取消註解(前面去掉#字號)如下：

<table><tbody><tr><td><pre tabindex="0"><code><span>1
</span><span>2
</span></code></pre></td><td><pre tabindex="0"><code data-lang="bash"><span><span><span># 使用上下左右鍵移動，按i輸入文字，改好後按Esc，再輸入「:wq」退出VIM</span>
</span></span><span><span>zh_TW.UTF-8 UTF-8
</span></span></code></pre></td></tr></tbody></table>

4.  設定系統語言成正體中文

<table><tbody><tr><td><pre tabindex="0"><code><span>1
</span><span>2
</span></code></pre></td><td><pre tabindex="0"><code data-lang="bash"><span><span>locale-gen
</span></span><span><span><span>echo</span> <span>"LANG=zh_TW.UTF-8 UTF-8"</span> &gt;&gt; /etc/locale.conf
</span></span></code></pre></td></tr></tbody></table>

5.  給自己的主機取名為`MyLinux`:

<table><tbody><tr><td><pre tabindex="0"><code><span>1
</span></code></pre></td><td><pre tabindex="0"><code data-lang="bash"><span><span><span>echo</span> <span>"MyLinux"</span> &gt;&gt; /etc/hostname
</span></span></code></pre></td></tr></tbody></table>

6.  建立host檔案。最後一行為主機名稱。

<table><tbody><tr><td><pre tabindex="0"><code><span>1
</span><span>2
</span><span>3
</span><span>4
</span></code></pre></td><td><pre tabindex="0"><code data-lang="bash"><span><span>touch /etc/hosts
</span></span><span><span><span>echo</span> <span>"127.0.0.1 localhost"</span> &gt;&gt; /etc/hosts
</span></span><span><span><span>echo</span> <span>"::1 localhost"</span> &gt;&gt; /etc/hosts
</span></span><span><span><span>echo</span> <span>"127.0.1.1 MyLinux"</span> &gt;&gt; /etc/hosts
</span></span></code></pre></td></tr></tbody></table>

7.  更改root的密碼:

8.  建立一般使用者帳戶"ivon"(名字可自取)，並修改密碼

<table><tbody><tr><td><pre tabindex="0"><code><span>1
</span><span>2
</span></code></pre></td><td><pre tabindex="0"><code data-lang="bash"><span><span>useradd -m -g users -G wheel,audio,video,storage -s /bin/bash ivon
</span></span><span><span>passwd ivon
</span></span></code></pre></td></tr></tbody></table>

9.  用vim編輯`/etc/sudoers`，賦予一般使用者帳戶sudo權限

<table><tbody><tr><td><pre tabindex="0"><code><span>1
</span><span>2
</span><span>3
</span></code></pre></td><td><pre tabindex="0"><code data-lang="bash"><span><span>vim /etc/sudoers
</span></span><span><span><span>#在"root ALL=(ALL) ALL"的下一行加入以下內容:</span>
</span></span><span><span>ivon <span>ALL</span><span>=(</span>ALL<span>)</span> ALL
</span></span></code></pre></td></tr></tbody></table>

10.  設定開機自動啟動SDDM顯示管理器、Network Manager、SSH的服務

<table><tbody><tr><td><pre tabindex="0"><code><span>1
</span><span>2
</span><span>3
</span></code></pre></td><td><pre tabindex="0"><code data-lang="bash"><span><span>systemctl <span>enable</span> sddm.service
</span></span><span><span>systemctl <span>enable</span> NetworkManager.service
</span></span><span><span>systemctl <span>enable</span> sshd.service
</span></span></code></pre></td></tr></tbody></table>

### [](https://ivonblog.com/posts/install-archlinux/#25-%E5%AE%89%E8%A3%9Dgrub%E9%96%8B%E6%A9%9F%E9%81%B8%E5%96%AE)[2.5. 安裝GRUB開機選單](https://ivonblog.com/posts/install-archlinux/#contents:25-%E5%AE%89%E8%A3%9Dgrub%E9%96%8B%E6%A9%9F%E9%81%B8%E5%96%AE)

1.  安裝GRUB和efibootmgr套件

<table><tbody><tr><td><pre tabindex="0"><code><span>1
</span></code></pre></td><td><pre tabindex="0"><code data-lang="bash"><span><span>pacman -S grub efibootmgr
</span></span></code></pre></td></tr></tbody></table>

2.  將EFI分區掛載到`/boot`目錄

3.  Nvidia顯示卡的用戶建議編輯GRUB設定檔：`vim /etc/default/grub`，在`GRUB_CMDLINE_LINUX_DEFAULT`後面加入`nomodeset`的核心參數，防止開機失敗卻無任何訊息輸出，以及啟用DRM框架：

<table><tbody><tr><td><pre tabindex="0"><code><span>1
</span></code></pre></td><td><pre tabindex="0"><code data-lang="bash"><span><span><span>GRUB_CMDLINE_LINUX_DEFAULT</span><span>=</span><span>"quiet splash nomodeset nvidia_drm.modeset=1"</span>
</span></span></code></pre></td></tr></tbody></table>

4.  接著安裝GRUB至EFI分區

<table><tbody><tr><td><pre tabindex="0"><code><span>1
</span><span>2
</span></code></pre></td><td><pre tabindex="0"><code data-lang="bash"><span><span>grub-install --target<span>=</span>x86_64-efi --bootloader-id<span>=</span>GRUB --efi-directory<span>=</span>/boot
</span></span><span><span>grub-mkconfig -o /boot/grub/grub.cfg
</span></span></code></pre></td></tr></tbody></table>

5.  系統安裝完成。退出chroot，取消掛載，關機。

<table><tbody><tr><td><pre tabindex="0"><code><span>1
</span><span>2
</span><span>3
</span><span>4
</span></code></pre></td><td><pre tabindex="0"><code data-lang="bash"><span><span><span>exit</span>
</span></span><span><span>umount /mnt/boot
</span></span><span><span>umount /mnt
</span></span><span><span>shutdown now 
</span></span></code></pre></td></tr></tbody></table>

6.  拔掉開機隨身碟後重開機。

## [](https://ivonblog.com/posts/install-archlinux/#3-%E7%B3%BB%E7%B5%B1%E5%BE%8C%E7%BA%8C%E5%AE%89%E8%A3%9D%E5%84%AA%E5%8C%96)[3\. 系統後續安裝優化](https://ivonblog.com/posts/install-archlinux/#contents:3-%E7%B3%BB%E7%B5%B1%E5%BE%8C%E7%BA%8C%E5%AE%89%E8%A3%9D%E5%84%AA%E5%8C%96)

如果Arch Linux開機後沒畫面，按CTRL+ALT+F2開啟純文字介面，登入root帳號後再除錯。或是用開機隨身碟chroot到系統進行修復。

一切順利的話，應該能登入KDE桌面。下面是一些小優化。

### [](https://ivonblog.com/posts/install-archlinux/#31-kde%E6%A1%8C%E9%9D%A2%E5%85%A5%E9%96%80)[3.1. KDE桌面入門](https://ivonblog.com/posts/install-archlinux/#contents:31-kde%E6%A1%8C%E9%9D%A2%E5%85%A5%E9%96%80)

KDE桌面布局跟Windows 10十分類似。

點一下桌面左下角可找到所有應用程式列表，包含「系統設定」和終端機「Konsole」。

右下角則是開關Wifi/藍芽、輸入法、顯示時間的面板。

### [](https://ivonblog.com/posts/install-archlinux/#32-%E5%AE%89%E8%A3%9Dfcitx5%E6%B3%A8%E9%9F%B3%E8%BC%B8%E5%85%A5%E6%B3%95)[3.2. 安裝Fcitx5注音輸入法](https://ivonblog.com/posts/install-archlinux/#contents:32-%E5%AE%89%E8%A3%9Dfcitx5%E6%B3%A8%E9%9F%B3%E8%BC%B8%E5%85%A5%E6%B3%95)

Fcitx5是Arch Linux最新的中文輸入法框架，支援注音/拼音等輸入方案，對Wayland支援度較好。

1.  確認已安裝Fcitx5套件

<table><tbody><tr><td><pre tabindex="0"><code><span>1
</span></code></pre></td><td><pre tabindex="0"><code data-lang="bash"><span><span>sudo pacman -S fcitx5-im fcitx5-chewing fcitx5-qt fcitx5-gtk fcitx5-chinese-addons
</span></span></code></pre></td></tr></tbody></table>

2.  在KDE左下角的應用程式列表找到「Fcitx5設定程式」，加入「新酷音」注音輸入法。
    
3.  針對Fcitx5，使用指令編輯檔案：`sudo vim /etc/environment`，加入以下環境變數：
    

<table><tbody><tr><td><pre tabindex="0"><code><span>1
</span><span>2
</span><span>3
</span><span>4
</span><span>5
</span></code></pre></td><td><pre tabindex="0"><code data-lang="bash"><span><span><span>GTK_IM_MODULE</span><span>=</span>fcitx
</span></span><span><span><span>QT_IM_MODULE</span><span>=</span>fcitx
</span></span><span><span><span>XMODIFIERS</span><span>=</span>@im<span>=</span>fcitx
</span></span><span><span><span>SDL_IM_MODULE</span><span>=</span>fcitx
</span></span><span><span><span>GLFW_IM_MODULE</span><span>=</span>fcitx
</span></span></code></pre></td></tr></tbody></table>

4.  重新載入環境變數。點選右下角的鍵盤圖示，右鍵重新啟動Fcitx5。

<table><tbody><tr><td><pre tabindex="0"><code><span>1
</span></code></pre></td><td><pre tabindex="0"><code data-lang="bash"><span><span><span>source</span> /etc/environment
</span></span></code></pre></td></tr></tbody></table>

5.  輸入文字時用CTRL+空白鍵來切換應該就會出現新酷音。Fcitx5還有對系統匣圖示按右鍵切換簡體輸出的功能。

### [](https://ivonblog.com/posts/install-archlinux/#33-%E5%AE%89%E8%A3%9Daur%E5%A5%97%E4%BB%B6%E7%AE%A1%E7%90%86%E5%93%A1)[3.3. 安裝AUR套件管理員](https://ivonblog.com/posts/install-archlinux/#contents:33-%E5%AE%89%E8%A3%9Daur%E5%A5%97%E4%BB%B6%E7%AE%A1%E7%90%86%E5%93%A1)

AUR (Arch User Repository) 是Arch Linux官方軟體庫以外的主要套件來源，通常會安裝一個AUR Helper來方便安裝套件，我使用yay。

1.  安裝yay

<table><tbody><tr><td><pre tabindex="0"><code><span>1
</span><span>2
</span><span>3
</span></code></pre></td><td><pre tabindex="0"><code data-lang="bash"><span><span>git clone https://aur.archlinux.org/yay.git
</span></span><span><span><span>cd</span> yay
</span></span><span><span>makepkg -si
</span></span></code></pre></td></tr></tbody></table>

2.  之後yay就能用像pacman的語法來安裝AUR上的套件，例如Arch Linux官方套件庫沒收的`Google Chrome`。yay執行時不需要加sudo。

<table><tbody><tr><td><pre tabindex="0"><code><span>1
</span><span>2
</span></code></pre></td><td><pre tabindex="0"><code data-lang="bash"><span><span>yay -S google-chrome
</span></span><span><span><span># 按下Enter同意安裝，並按照螢幕提示輸入密碼</span>
</span></span></code></pre></td></tr></tbody></table>

3.  參考[PTT文章](https://www.ptt.cc/bbs/Linux/M.1626719840.A.60B.html)，編輯`/etc/makepkg.conf`，加入`MAKEFLAGS="-j$(nproc)"`，讓AUR編譯時使用全部CPU。
    
4.  編輯`/etc/pacman.conf`，取消註解`Color`和`ParallelDownloads`，開啟顏色和平行下載套件。自行新增`ILoveCandy`參數開啟pacman的彩蛋。
    

### [](https://ivonblog.com/posts/install-archlinux/#34-%E5%AE%89%E8%A3%9Dufw%E9%98%B2%E7%81%AB%E7%89%86)[3.4. 安裝UFW防火牆](https://ivonblog.com/posts/install-archlinux/#contents:34-%E5%AE%89%E8%A3%9Dufw%E9%98%B2%E7%81%AB%E7%89%86)

Arch Linux預設的防火牆是iptables，通常我們會裝一個前端方便管理防火牆規則。

1.  安裝UFW，並設定開機自動啟動UFW服務

<table><tbody><tr><td><pre tabindex="0"><code><span>1
</span><span>2
</span><span>3
</span></code></pre></td><td><pre tabindex="0"><code data-lang="bash"><span><span>sudo pacman -S ufw
</span></span><span><span>sudo systemctl <span>enable</span> ufw
</span></span><span><span>sudo systemctl start ufw
</span></span></code></pre></td></tr></tbody></table>

2.  設定防火牆基本規則：允許所有對外連線，阻擋所有連入連線，僅開放SSH的通訊埠。

<table><tbody><tr><td><pre tabindex="0"><code><span>1
</span><span>2
</span><span>3
</span><span>4
</span></code></pre></td><td><pre tabindex="0"><code data-lang="bash"><span><span>sudo ufw default allow outgoing
</span></span><span><span>sudo ufw default deny incoming
</span></span><span><span>sudo ufw allow ssh
</span></span><span><span>sudo ufw reload
</span></span></code></pre></td></tr></tbody></table>

### [](https://ivonblog.com/posts/install-archlinux/#35-%E5%AE%89%E8%A3%9Dflatpak)[3.5. 安裝Flatpak](https://ivonblog.com/posts/install-archlinux/#contents:35-%E5%AE%89%E8%A3%9Dflatpak)

用Flatpak安裝軟體可以在各Linux發行版享有一致的軟體版本，例如LibreOffice、GIMP、Steam，並且安裝軟體不需要root權限。

1.  安裝Flatpak，並加入Flathub軟體庫

<table><tbody><tr><td><pre tabindex="0"><code><span>1
</span><span>2
</span></code></pre></td><td><pre tabindex="0"><code data-lang="bash"><span><span>sudo pacman -S flatpak
</span></span><span><span>flatpak remote-add --user --if-not-exists flathub https://dl.flathub.org/repo/flathub.flatpakrepo
</span></span></code></pre></td></tr></tbody></table>

2.  到[Flathub](https://flathub.org/)查看APP列表，例如安裝Steam：

<table><tbody><tr><td><pre tabindex="0"><code><span>1
</span></code></pre></td><td><pre tabindex="0"><code data-lang="bash"><span><span>flatpak install flathub com.valvesoftware.Steam
</span></span></code></pre></td></tr></tbody></table>

## [](https://ivonblog.com/posts/install-archlinux/#4-arch-linux%E5%AE%89%E8%A3%9D%E5%BE%8C%E8%A8%98)[4\. Arch Linux安裝後記](https://ivonblog.com/posts/install-archlinux/#contents:4-arch-linux%E5%AE%89%E8%A3%9D%E5%BE%8C%E8%A8%98)

這篇文章最初寫於2022年7月，後多次修訂至今。

那時說Distro hopping的終點來到了Arch Linux。

標題之所以說主力，是因為主要用途是用於文書、剪輯、開發、遊戲。

資質駑鈍，安裝Arch Linux是一年下來玩各種Linux(不論是手機還電腦)累積下來的成果，現在體驗到了掌控自己電腦的感覺。

2021年第一次嘗試安裝Arch Linux時，GNOME的APP怎麼開怎麼崩，不會設定中文和輸入法，之後開機便卡在終端機，果斷放棄回去用Ubuntu和Windows 11。幾個月後玩FreeBSD才想到應該是沒裝Nvidia驅動的問題。

有了各種Linux基礎概念後，這次Arch Linux一小時就安裝完成。

玩手機Termux Proot distro、實機操作過PinePhone、在Android上刷postmarketOS、嘗試編譯Android的docker kernel、在各種桌面系統的distro hopping經驗，讓我掌握了設定語言和時間等基本知識。

大致上主流發行版都體驗過了一次： Ubuntu → Manjaro → Debian → Fedora → openSUSE Tumbleweed → GUIX → Arch Linux

KDE介面很細心，GNOME操作邏輯很炫，XFCE是測試桌面的第一穩定選擇，所以到現在我還是選了KDE。

安裝時是有線網路…不過裝完linux-firmware開機後我的高通網卡就自動抓到驅動了 :P 還以為要像Realtek的自己編譯。

原本擔心AUR會不會套件比較難安裝，但其實也沒那麼難用。Fedora、openSUSE某些軟體會不受開發者支援，所以才會去用Flatpak，但AUR似乎是包山包海。

等Arch Linux玩熟後，就可以前進到Gentoo了。

[Linux](https://ivonblog.com/tags/linux/) [Arch Linux](https://ivonblog.com/tags/arch-linux/)