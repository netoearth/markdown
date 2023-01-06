Ubuntu Linux 的 Xwindow (Gnome、Cinnamon...) 等等的 Desktop 環境設定參數，現在都使用 dconf 的資料庫來儲存，所以要備份還原，只要將這個檔案直接備份再放回去即可。

但是若不像全部備份，想要只備份、還原某部分的設定資料，要怎麼做呢？

dconf 平常放的路徑：/home/user/.config/dconf/user，只要將此檔案備份，然後新的機器再放回去即可。

Ubuntu 官方文件：[Ubuntu Manpage: dconf - Simple tool for manipulating a dconf database](https://manpages.ubuntu.com/manpages/bionic/man1/dconf.1.html)

### dconf dump(備份)、reset(重新設定)、restore(還原)

-   dconf 備份、還原全部
    -   dconf dump / > dconf-desktop # backup
    -   dconf reset -f /
    -   dconf load / < dconf-desktop # restore
-   dconf 備份、還原部分設定(Gnome、cinnamon)
    -   dconf dump /org/gnome/ > my\_gnome\_settings
    -   dconf reset -f /org/gnome/
    -   dconf load /org/gnome/ < my\_gnome\_settings
    -   dconf dump /org/cinnamon/ > cinnamon\_settings
    -   dconf reset -f /org/cinnamon/
    -   dconf load /org/cinnamon/ < cinnamon\_settings
-   查看設定
    -   gsettings list-recursively org.freedesktop.ibus.general
    -   gsettings list-recursively org.gnome.shell

#### 相關網頁

-   [Backup And Restore Linux Desktop System Settings With Dconf](https://ostechnix.com/backup-and-restore-linux-desktop-system-settings-with-dconf/)
-   [dconf - DConf 使用筆記](https://foreachsam.github.io/book-util-dconf/book/content/command/dconf/)

## 文章導覽