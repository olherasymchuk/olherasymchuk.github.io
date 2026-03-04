---
title: Hyper-V у Windows Home
categories: [Windows]
---
Хоч загальноприйнято вважати що [Hyper-V role can't be installed on Windows 10 Home or Windows 11 Home](https://learn.microsoft.com/en-us/windows-server/virtualization/hyper-v/get-started/install-hyper-v) - у мережі нескладно відшукати рецепт увімкнення Hyper-V в домашній редакції Windows.

Як і з будь-якими іншими гіпервізорами, бажано щоб комп'ютер підтримував і мав увімкненою в BIOS апаратну підтримку віртуалізації. Окрім цього, варто щоб гіпервізор на комп'ютері був встановлений лише один.

Якщо у більш професійних редакціях, Hyper-V можна увімкнути через GUI або PowerShell - то в Windows Home знадобиться створити й запустити із правами адміністратора .bat файл з наступним вмістом:

```batch
pushd "%~dp0"
dir /b %SystemRoot%\servicing\Packages\*Hyper-V*.mum >hyper-v.txt
for /f %%i in ('findstr /i . hyper-v.txt 2^>nul') do dism /online /norestart /add-package:"%SystemRoot%\servicing\Packages\%%i"
del hyper-v.txt
Dism /online /enable-feature /featurename:Microsoft-Hyper-V -All /LimitAccess /ALL
pause
```
