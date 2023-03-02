阅读： 97

耳朵问了个问题，怎样批量修改文件名为从1开始的数字，在不下软件的情况下。

至少可以写个rename.bat，内容如下，支持目录树

————————————————————————–  
@rem rename.bat \[target dir\]  
@echo off  
setlocal enabledelayedexpansion  
if “%~1” == “” (  
set target=”.”  
) else (  
set target=%1  
)  
set i=0  
for /r %target% %%x in (\*) do (  
rem echo “%%~nxx”  
if not “%%~nxx” == “rename.bat” (  
set /a i += 1  
rem @echo “%%x” “%%~dpx” “%%~dpx!i!%%~xx”  
rem @echo rename “%%x” “!i!%%~xx”  
if not exist “%%~dpx!i!%%~xx” (  
rename “%%x” “!i!%%~xx”  
)  
)  
)  
————————————————————————–

DOS批处理属于过时的技术，比较复杂，前述实现的部分参数可参”for /?”，其余看不明白的放狗搜吧。只在Win10的cmd中测试，不清楚Windows更早版本是否适用。

上面是保留扩展名的版本，下面是不保留扩展名的版本

————————————————————————–  
@rem rename.bat \[target dir\]  
@echo off  
setlocal enabledelayedexpansion  
if “%~1” == “” (  
set target=”.”  
) else (  
set target=%1  
)  
set i=0  
for /r %target% %%x in (\*) do (  
rem echo “%%~nxx”  
if not “%%~nxx” == “rename.bat” (  
set /a i += 1  
rem @echo “%%x” “%%~dpx” “%%~dpx!i!%%~xx”  
rem @echo rename “%%x” “!i!%%~xx”  
if not exist “%%~dpx!i!” (  
rename “%%x” “!i!”  
)  
)  
)  
————————————————————————–

若非IT人士，较简用法是

————————————————————————–  
Win+R  
输入cmd回车  
执行”cd /d <target dir>”  
执行”rename.bat .”  
————————————————————————–

不会命令行的，将rename.bat放到目标目录，双击执行即可。注意rename.bat属于不可逆操作，谨慎使用。

微博用户uid(1650307040) uid(2005085137)提供了PowerShell实现，学习TA俩的实现后我做了一点笔记。下面是批量重命名之前的一些测试记录

Get-ChildItem | Select \*  
Get-ChildItem -Recurse -Path . -File | Select BaseName,Extension  
Get-ChildItem -Recurse -Path . -File | Where {$\_.Extension -eq ‘.jpg’}  
$i=0;Get-ChildItem -Recurse -Path . -File | %{Write-Output (‘{0} -> {1}\\{2}{3}’ -f $\_.FullName, $\_.DirectoryName, ++$i, $\_.Extension)}  
Get-ChildItem -Recurse -Path . -File | foreach {$i=0} {Write-Output ($\_.FullName + ” -> ” + $\_.DirectoryName + “\\” + (‘{0}’ -f ++$i) + $\_.Extension)}

\-f左侧是fmt，右侧是实际参数，{0}、{1}等对应实参的位置。Get-ChildItem不要用”-Name”，用如下命令检查，此时有PSChildName属性，没有Extension属性。

Get-ChildItem -Name | Select \*

如下命令均可完成批量重命名，支持目录树

Get-ChildItem -Recurse -Path . -File | %{$i=0}{Rename-Item $\_.FullName -NewName (‘{0}\\{1}{2}’ -f $\_.DirectoryName, ++$i, $\_.Extension)}  
Get-ChildItem -Recurse -Path . -File | foreach {$i=0} {Rename-Item $\_.FullName -NewName ($\_.DirectoryName+(‘\\{0}’ -f ++$i)+ $\_.Extension)}

%和foreach都是ForEach-Object的别名

$ alias | findstr ForEach-Object  
$ Get-Alias -Definition ForEach-Object  
Alias % -> ForEach-Object  
Alias foreach -> ForEach-Object

Get-ChildItem之后对文件名排序后倒序输出

Get-ChildItem -Recurse -Path . -File | Sort-Object -Property FullName -Descending | %{$i=0}{Write-Output (‘{0} -> {1}\\{2}{3}’ -f $\_.FullName, $\_.DirectoryName, ++$i, $\_.Extension)}  
Get-ChildItem -Recurse -Path . -File | Sort-Object -Property @{Expression = “FullName”; Descending = $true} | %{$i=0}{Write-Output (‘{0} -> {1}\\{2}{3}’ -f $\_.FullName, $\_.DirectoryName, ++$i, $\_.Extension)}

$ alias | findstr sort  
Alias sort -> Sort-Object

微博用户uid(5688990748)提供了一个有意思的思路，选中所有待改名文件，复制路径，粘到Excel里，批量删除每条前面的路径只留文件名，然后左侧加一列都是ren，右侧加一列都是新文件名，把这三列粘到txt里，再把txt改成bat。TA这个办法不能处理目录树。里面那句「批量删除每条前面的路径」，有多种实现办法，其中一种是利用Excel的「分列」功能

————————————————————————–  
选中路径列所有行  
菜单中选中”数据”  
分列  
————————————————————————–

分列时有一些具体操作，挺直白的，试试就会，参看

https://mp.weixin.qq.com/s/F1J1HB543xh917pPrPxZvw

微博用户uid(1926822403)用ChatGPT找了一段PowerShell实现，某种程度上这是可行的。他找来的方案并不比前述网友提供的PS实现优越，不展示。我自己倒是去试了试ChatGPT，中英提问都试过，其回答需要提问者自行鉴别后修正，有些不但不满足原始需求，并且从语法上讲就有问题，ChatGPT不是stackoverflow。

ChatGPT  
https://chat.openai.com/auth/login

非IT人士不必尝试ChatGPT，官面上不对大陆地区提供服务。

**版权声明**  
本站“技术博客”所有内容的版权持有者为绿盟科技集团股份有限公司（“绿盟科技”）。作为分享技术资讯的平台，绿盟科技期待与广大用户互动交流，并欢迎在标明出处（绿盟科技-技术博客）及网址的情形下，全文转发。  
上述情形之外的任何使用形式，均需提前向绿盟科技（010-68438880-5462）申请版权授权。如擅自使用，绿盟科技保留追责权利。同时，如因擅自使用博客内容引发法律纠纷，由使用者自行承担全部法律责任，与绿盟科技无关。