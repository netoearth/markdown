[Home](https://www.ltsplus.com/)

[Linux](https://www.ltsplus.com/linux)

RHEL / CentOS 安裝 ImageMagick 及 PHP 模組

ImageMagick 是一套開源的圖像處理工具, 可以在指令模式下建立, 編輯, 轉檔圖像檔案, 支援超過 200 種圖像格式, 例如JEPG, GIF, PNG, TIFF 等.

如果在 PHP 要使用 ImageMagick, 除了在 PHP 執行 ImageMagick 外, 最好的方法也是安裝 PHP 的 ImageMagick 模式 — Imagick. Imagick 是 PHP 的模組, 可以透過 ImageMagick API 建立及編輯圖片, 而 ImageMagick 及 Imagick 也是 WordPress 建議安裝的套件.

首先安裝 php-pear, php-devel 及 gcc:

\# yum install php-pear php-devel gcc

安裝好以上套件外, 便安裝 ImageMagick:

\# yum install ImageMagick ImageMagick-devel ImageMagick-perl

如果在 RHEL 8 及 CentOS 8, 需要改為安裝 GraphicsMagick 套件取代, GraphicsMagick 是 ImageMagick 的分支, 用以下指令安裝:

\# dnf install GraphicsMagick GraphicsMagick-devel GraphicsMagick-perl

安裝好 ImageMagick 或 GraphicsMagick 後, 可以安裝 Imagick PHP 擴展, 執行以下指令安裝:

\# pecl install imagick

系統會出現以下提問:

**Please provide the prefix of Imagemagick installation \[autodetect\] :**

直接按 Enter 鍵便可以了.

完成安裝後, 需要在 php.ini 加入 imagick.so 擴展, 執行以下指令:

\# echo extension=imagick.so >> /etc/php.ini

取決於 PHP 安裝的方法, 重新啟動 HTTPD 或 PHP-FPM:

\# systemctl httpd restart

或

\# systemctl php-fpm restart

執行以下指令可以確認 imagick 安裝完成:

\# php -m | grep imagick

如果回傳 “imagick”, 便表示安裝成功了。