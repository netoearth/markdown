更新：下面使用注册表安装语言包后重启会失效，蓝点网测试独立语言包可以。

下载地址：[https://dl.lancdn.com/landian/hotfix/atlasos\_cn](https://dl.lancdn.com/landian/hotfix/atlasos_cn)

下载后把语言包放到桌面，打开管理员模式的powershell，然后进入安装包路径后使用add命令添加即可，添加后需要重启，然后重新设置语言，设置界面还是中文的不知道啥情况。

```
cd desktopadd-appxpackage 19041.cn.pack.Appx#按回车执行
```

![](https://www.landiannews.com/wp-content/uploads/2022/11/image-1-2022-11-26-12-23-08-39.png)

[昨天蓝点网提到有开源项目组基于 Windows 10 20H2 推出AtlasOS替代系统](https://www.landiannews.com/download/96187.html)，该系统删减大量冗余的组件。

项目组的核心目的就是游戏性能，所以凡是可能占用硬件资源的非必要组件都被删除以免影响影响运行性能。

没想到昨天蓝点网转发这个系统后引起相当多的网友关注，不少网友安装体验后疯狂夸赞系统确实非常清爽。

美中不足的是该系统暂时没有简体中文版，内置语言包下载功能被禁用，下面这篇内容就是解决语言包问题。

**为照顾新手用户下面的图文教程超级详细，专业用户嫌长的话可以直接拉到本文结尾查看如何修改注册表项。**

**以下是操作方法：**

**▼转到设置、时间和语言**

[![[技巧] 基于Windows 10 20H2的超精简系统AtlasOS安装简体中文语言包](https://img.lancdn.com/landian/2022/11/96172-1.png)](https://img.lancdn.com/landian/2022/11/96172-1.png)

**▼在语言里点击添加语言包**

[![[技巧] 基于Windows 10 20H2的超精简系统AtlasOS安装简体中文语言包](https://img.lancdn.com/landian/2022/11/96172-2.png)](https://img.lancdn.com/landian/2022/11/96172-2.png)

**▼选择中文后点击安装，此时这里可能会报错**

[![[技巧] 基于Windows 10 20H2的超精简系统AtlasOS安装简体中文语言包](https://img.lancdn.com/landian/2022/11/96172-3.png)](https://img.lancdn.com/landian/2022/11/96172-3.png)

**▼点击语言包旁边的Options**

[![[技巧] 基于Windows 10 20H2的超精简系统AtlasOS安装简体中文语言包](https://img.lancdn.com/landian/2022/11/96172-5.png)](https://img.lancdn.com/landian/2022/11/96172-5.png)

**▼点击Download**

[![[技巧] 基于Windows 10 20H2的超精简系统AtlasOS安装简体中文语言包](https://img.lancdn.com/landian/2022/11/96172-4.png)](https://img.lancdn.com/landian/2022/11/96172-4.png)

**▼出现错误代码**

[![[技巧] 基于Windows 10 20H2的超精简系统AtlasOS安装简体中文语言包](https://img.lancdn.com/landian/2022/11/96172-6.png)](https://img.lancdn.com/landian/2022/11/96172-6.png)

**▼点击开始菜单旁边的搜索按钮，输入 regedit 打开注册表**

```
#转到如下路径HKEY_LOCAL_MACHINE\SOFTWARE\Policies\Microsoft\Windows\WindowsUpdate#双击这个项DoNotConnectToWindowsUpdateInternetLocations#将其键值修改为0保存即可
```

懒得动手？蓝点网做好了注册表，双击打开点击是即可：[https://dl.lancdn.com/landian/Regedit/atlasos](https://dl.lancdn.com/landian/Regedit/atlasos)

[![[技巧] 基于Windows 10 20H2的超精简系统AtlasOS安装简体中文语言包](https://img.lancdn.com/landian/2022/11/96172-7.png)](https://img.lancdn.com/landian/2022/11/96172-7.png)

**▼重新返回语言包界面点击下载即可正常下载**

[![[技巧] 基于Windows 10 20H2的超精简系统AtlasOS安装简体中文语言包](https://img.lancdn.com/landian/2022/11/96172-8.png)](https://img.lancdn.com/landian/2022/11/96172-8.png)

**▼在显示语言里切换为中文，按提示注销或重启**

[![[技巧] 基于Windows 10 20H2的超精简系统AtlasOS安装简体中文语言包](https://img.lancdn.com/landian/2022/11/96172-9.png)](https://img.lancdn.com/landian/2022/11/96172-9.png)

**▼注销或重启后进入时间和语言，调整市区为UTC+8**

[![[技巧] 基于Windows 10 20H2的超精简系统AtlasOS安装简体中文语言包](https://img.lancdn.com/landian/2022/11/96172-13.png)](https://img.lancdn.com/landian/2022/11/96172-12.png)

**▼点击语言、中文旁边的选项**

[![[技巧] 基于Windows 10 20H2的超精简系统AtlasOS安装简体中文语言包](https://img.lancdn.com/landian/2022/11/96172-14.png)](https://img.lancdn.com/landian/2022/11/96172-14.png)

**▼点击区域格式的设置**

[![[技巧] 基于Windows 10 20H2的超精简系统AtlasOS安装简体中文语言包](https://img.lancdn.com/landian/2022/11/96172-15.png)](https://img.lancdn.com/landian/2022/11/96172-15.png)

**▼点击右上角的其他日期、时间和区域设置**

[![[技巧] 基于Windows 10 20H2的超精简系统AtlasOS安装简体中文语言包](https://img.lancdn.com/landian/2022/11/96172-16.png)](https://img.lancdn.com/landian/2022/11/96172-16.png)

**▼点击区域、管理、复制设置**

[![[技巧] 基于Windows 10 20H2的超精简系统AtlasOS安装简体中文语言包](https://img.lancdn.com/landian/2022/11/96172-17.png)](https://img.lancdn.com/landian/2022/11/96172-17.png)

**▼勾选底部的两项确定并提示重启即可**

[![[技巧] 基于Windows 10 20H2的超精简系统AtlasOS安装简体中文语言包](https://img.lancdn.com/landian/2022/11/96172-18.png)](https://img.lancdn.com/landian/2022/11/96172-18.png)

[![[技巧] 基于Windows 10 20H2的超精简系统AtlasOS安装简体中文语言包](https://img.lancdn.com/landian/2022/11/96172-19.png)](https://img.lancdn.com/landian/2022/11/96172-19.png)

**修改后还有些是英文怎么办？**

微软的语言包没法彻底将系统中文化，除非本身系统就是对应的语言，所以即便切换后还有些地方还是英文。

这个问题基本不影响日常使用，不知道项目组后续有没有计划推出简体中文版，如果有的话那本地化会好些。

哪些地方还是英文的：开始菜单的用户按钮里包括注销、文档、图片之类的文件夹名称等可能还是英文显示。

文件选择对话框可能还是英文显示的，其他地方蓝点网没挨个测试，不过按照经验应该有不少地方还是英文。

**专业用户直接看这里修改注册表：**

```
#转到如下路径HKEY_LOCAL_MACHINE\SOFTWARE\Policies\Microsoft\Windows\WindowsUpdate#双击这个项DoNotConnectToWindowsUpdateInternetLocations#将其键值修改为0保存即可
```

懒得动手？蓝点网做好了注册表，双击打开点击是即可：[https://dl.lancdn.com/landian/Regedit/atlasos](https://dl.lancdn.com/landian/Regedit/atlasos)

### 分享海报

下载海报

![[技巧] 基于Windows 10 20H2的超精简系统AtlasOS安装简体中文语言包](https://www.landiannews.com/wp-content/uploads/2022/11/image-1-2022-11-24-14-50-34-60.png)

252022/11

## \[技巧\] 基于Windows 10 20H2的超精简系统AtlasOS安装简体中文语言包

更新：下面使用注册表安装语言包后重启会失效，蓝点网测试独立语言包可以。 下载地址：https://dl.lancdn.com/landian/hotfix/atlasos\_cn 下载后把语言包放到桌面...

![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8CAYAAAA6/NlyAAAAAXNSR0IArs4c6QAAA/5JREFUaEPtWtlynDAQhP//aFK7ZVFD08eg2KnUyn5JzK5AI/U1Mvu2bcfW+DmOY9v3/fbN1/XxMz5n1+pA/Hz8nu6Pz3n9rualSnpVcNQJ4BdrEez/46Gvf13B+Az23XoPNmG1uOpe9R7jO2fBanVVEW5X1G6yCbAFqztXn6M2Rm3EeF5FwW/BuDsdGCvOuh1Ju1V3fkA9jXE6MwXpVDxysE5AcRi5qqjiNODHIP1RBUsZ/7IihAsTk9eC4K4yMWRwVdfUvJIdKrd5i1bDhmUhrEC3GMjHsUjqerLFKR9eqmCbOsxKpJWvHugCRYcqbLwLS24Ddyy4QkylJxUgamBwnOwUidzvJDU1pl5fr2AmWsngXThhXmwhBg0JCl6XOl2VP3e4bnvKyWgJnQVSdqXgzWylk6vHOAnvweGlCkahqqv0xODZqqbYh/RQgthRZSW4Fe5vSKuCO3xkyEgL5pKWU1pGNZXXMdCcc1q+YBf1aphQBwPJf5moKNo4L1bqzdB1qckd8XQeeOHHV/PAbChl5idpqsNVpQU3DquJMZvA+Dgm/cTWnEi5uSR/Zqh5zw85/PEFM0g/6WFV6mKwTqcZTyk0E07sDqtI2LEOFycrX93Rj4KlO2RIdPotWIlIJxGlNKRSlxvngg2KpApKl/YQOex41LEbtCmm5Lio3fMx5uHKFqUNLlcws6UkOPg5CoXKvEkEGbSfdHHOMc45zfqwi4jM1tyxToW94qxLV2w8Pu+c77IFMxXuBBCm6qmHdj6OIqcg3Tn9oGOHaKnuR5m/SlIIvaQHqm9lFGBzTFl8qPgYe/tzqTuvUjuOHEp+7GzJ8flbCmbn0h0RUZ6oFBzjZJo8hojqt+wz5/cXFA3Rmg0IClIK8jhZlaVdxsbi2bMUtc4svUzBLGmljsOdJSeeo4ikzic1+grq8QBAqWmSfwYdFx5YwQ6SnRDj6DModNbhdhiFJlnM7eZwxqWEyqGis+CJjrWO23taKsJ1rUZ1KW7x/mnBqXlIqcmp7tNcnPisrNC1lzjGvqdVIfw3TXryzQ7UcTEU7xPt1it45pUHeZog/taroIi74XaxQ4+OftAszbyv+l2n22H+mBQXhc0JaPJ7pQfrFfwE0mpVcec6rSaD+UynljI30ubRi2mdSTJeOkV1Rz8Klp3rah5rFtzx2M53MCcroVNqTI9kxLueKk6qZuNEJx7xYNhIjbpSUjYh1Qi4sNDplth4xe2pN+LrAz66YOXPuJsdH1cUSa2lQwMKIIvG0zucuMKaik5YqE7Q7dAQcRh66lzWLVhBRYWICuMODJPfose7Yyan8qlVnPLh2IIVK+lA1CkxK9wtnprb5YgnFcA40bWS/63gPx8GtisXzmxlAAAAAElFTkSuQmCC)

感谢您的阅读，本文为蓝点网原创内容，转载时请标注来源于蓝点网和[本文链接](https://www.landiannews.com/archives/96172.html)