---
title: Налаштування ISO файлу Windows 7
categories: [Windows]
---
Сформував інсталяційний образ Windows 7. Система, звісно, застаріла - але ще живуть комп'ютери на яких сімка швидка і прудка, а от новіші ОС працюють ледве-ледве.

Додав оновлення, доступні на 20 листопада 2025, та декілька програм:

- Mozilla Firefox ESR версії 115.28.0
- Microsoft Edge (він приходить разом з оновленнями)
- Microsoft .NET Framework 4.8
- Microsoft Visual C++ 2015-2022 Redistributable x86 (а також x64 у відповідній версії Windows)

Нижче занотував процес створення подібного образу. Процес апробувався в середовищі:

- Windows 11 з вбудованим PowerShell та Hyper-V
- утиліта [oscdimg](https://learn.microsoft.com/en-us/windows-hardware/manufacture/desktop/oscdimg-command-line-options), один із варіантів її завантажити - у складі [Windows Assessment and Deployment Kit](https://learn.microsoft.com/en-us/windows-hardware/get-started/adk-install)

Запускаємо PowerShell з правами адміністратора, переглядаємо та оголошуємо змінні:

```powershell
$RootDir        = "D:\win_repack\2025_11_20\"

$IsoOriginalDir = "D:\win_repack\iso_original\"
$IsoCapture     = $IsoOriginalDir + "win11.iso"

$VMBackupDir    = $RootDir + "vm_backup\"
$VMDrivesDir    = "D:\Hyper-V\vDisks\"
$VMNamePrefix   = "base-"
$HyperVSwitch   = "Default Switch"

$VirtualMachines = @{
    "win7sp1ua-home-basic-x32"   = "Windows 7 Home Basic"
    "win7sp1ua-home-premium-x32" = "Windows 7 Home Premium"
    "win7sp1ua-home-premium-x64" = "Windows 7 Home Premium"
    "win7sp1ua-starter-x32"      = "Windows 7 Starter"
}
```
У директорію $IsoOriginalDir знадобиться завантажити образи дисків. Назвати їх потрібно згідно лівої колонки у змінній $VirtualMachines, додаючи розширення .iso - наприклад, win7sp1ua-home-basic-x32.iso або win7sp1ua-starter-x32.iso

В тому ж вікні PowerShell запускаємо наступний скрипт щоб створити віртуальні машини:

```powershell
$VirtualMachines.keys | ForEach-Object {
    $VMName = $VMNamePrefix + $_
    
    $VMRAM = 4GB
    if ($VMName.Contains("starter")) {
        $VMRAM = 2GB # можна і більше, але ОС не побачить додаткову пам'ять
    }
    $VirtualDrive = $VMDrivesDir + $VMName + '.vhdx'
    $ISO = $IsoOriginalDir + $_ + ".iso"
    
    New-VM -Name $VMName -MemoryStartupBytes $VMRAM -NewVHDPath $VirtualDrive -NewVHDSizeBytes 40GB -Generation 1
    Set-VMMemory $VMName -DynamicMemoryEnabled $false
    Set-VMProcessor $VMName -Count 2
    Get-VMScsiController -VMName $VMName -ControllerNumber 0 | Remove-VMScsiController
    Get-VMNetworkAdapter -VMName $VMName | Remove-VMNetworkAdapter -Confirm:$false
    Add-VMNetworkAdapter -VMName $VMName -Name "Legacy Network Adapter" -SwitchName $HyperVSwitch -IsLegacy $true
    Set-VMDvdDrive -VMName $VMName -Path $ISO
    Set-VM -Name $VMName -AutomaticCheckpointsEnabled $false
}
```
Заходимо в консоль Hyper-V: Win+R, virtmgmt.msc

Вмикаєм віртуальні машини, встановлюєм ОС.

Після встановлення чекаємо на перший екран привітань із запитом про ім'я користувача - замість відповіді на питання натискаємо Ctrl+Shift+F3 - і система завантажується в режим аудиту.

Надалі ОС буде щоразу завантажуватись у режимі аудиту, при цьому на екрані з'являтиметься невеличке віконце утиліти Sysprep - поки що ми щоразу його закриватимем.

Після першого запуску заходимо у консоль Windows Update - вмикаємо автоматичні оновлення і ОС спробує перевірити наявність. Ймовірно ця спроба завершиться помилкою із кодом 80072EFE. Необхідно встановити оновлену версію агента Windows Update як описано в [https://learn.microsoft.com](https://learn.microsoft.com/uk-ua/troubleshoot/windows-client/installing-updates-features-roles/update-windows-update-agent#stand-alone-packages-for-windows-7-sp1-and-windows-server-2008-r2-sp1)

Якщо вбудований бравзер не зможе завантажити файли, один із варіантів обхідного рішення:

```powershell
# для x86
[Net.ServicePointManager]::ServerCertificateValidationCallback = {$true}
(New-Object System.Net.WebClient).DownloadFile("https://download.windowsupdate.com/windowsupdate/redist/standalone/7.6.7600.320/windowsupdateagent-7.6-x86.exe", "C:\Windows\Temp\windowsupdateagent-7.6-x86.exe")
```

```powershell
# для x64
[Net.ServicePointManager]::ServerCertificateValidationCallback = {$true}
(New-Object System.Net.WebClient).DownloadFile("https://download.windowsupdate.com/windowsupdate/redist/standalone/7.6.7600.320/windowsupdateagent-7.6-x64.exe", "C:\Windows\Temp\windowsupdateagent-7.6-x64.exe")
```
Завантажуємо, встановлюємо. Якщо Windows і після цього не може оновитись (я з таким стикнувся у x86 версіях), можливо доведеться встановити оновлення [KB3083324](https://support.microsoft.com/kb/KB3083324).

Якщо https сторінка завантаження не відкриється у вбудованому бравзері - можна спробувати завантажити оновлення із цього сайту, але не забудьте перевірити контрольні суми файлу:

```powershell
(New-Object System.Net.WebClient).DownloadFile("http://anykey.cv.ua/files/windows-custom-image/Windows6.1-KB3083324-x86.msu", "C:\Windows\Temp\Windows6.1-KB3083324-x86.msu")
```
Далі налаштовуємо ОС.

Після всіх налаштувань та оновлень запускаємо Disk Cleanup.

Після виконання Disk Cleanup вимикаємо віртуалки.

В процесі перетворення у образ ISO налаштовані віртуалки зазнають кардинальних змін і будуть видалені. Їхні дані можливо знадобляться у майбутньому, тому зараз гарний час створити резервні копії. Запускаємо в консолі PowerShell керуючого комп'ютера:

```powershell
$VirtualMachines.keys | ForEach-Object {
    $VMName = $VMNamePrefix + $_
    Export-VM -Name $VMName -Path $VMBackupDir
}
```
Після створення копії запускаємо кожну з віртуалок, і цього разу не закриваємо вікно sysprep - натомість обираємо опції "увійти до першого запуску", "узагальнення", "завершення роботи". Тиснемо ОК, чекаємо на виконання sysprep і вимкнення віртальних машин.

Після цього запускаємо у PowerShell керуючої машини блок коду, який підготує тимчасові .vhdx диски і збереже в них .wim образи кожної з операційних систем. Дивимось на екран і уважно виконуємо інструкції, які з'являтимуться у ході виконання сценарію:

```powershell
$VirtualMachines.GetEnumerator() | ForEach-Object {

    $VMName = $VMNamePrefix + $($_.key)
    $TempVHD = $RootDir + "temp-"  + $($_.key) + ".vhdx"

    New-VHD -Path $TempVHD -SizeBytes 15GB |
        Mount-VHD -Passthru |
        Initialize-Disk -PassThru |     
        New-Partition -AssignDriveLetter -UseMaximumSize | 
        Format-Volume -FileSystem NTFS -Confirm:$false -Force
    Dismount-VHD -Path $TempVHD

    Add-VMHardDiskDrive `
        -VMName $VMName `
        -ControllerType IDE `
        -ControllerNumber 0 `
        -ControllerLocation 1 `
        -Path $TempVHD

    Set-VMDvdDrive `
        -VMName $VMName `
        -Path $IsoCapture

    Write-Host "Зараз запустіть $VMName і пильнуйте напис Press any key to boot from CD"
    Write-Host
    Write-Host "Після запуску натисніть Shift+F10"
    Write-Host
    Write-Host "dism /capture-image /imageFile:`"d:\install-$($_.key).wim`" /captureDir:e:\ /name:`"$($_.value)`" /description:`"$($_.value)`""
    Write-Host "wpeutil shutdown"

    Read-Host "Натисніть Enter для видалення віртуальної машини та переходу до наступного"

    Write-Host "Відключення диску з результатами"
    Remove-VMHardDiskDrive `
        -VMName $VMName `
        -ControllerType IDE `
        -ControllerNumber 0 `
        -ControllerLocation 1

    Write-Host "Видаляємо віртуалку $VMName та її диск"
    GET-VM -VMName $VMName | Get-VMHardDiskDrive | Foreach { echo -path $_.Path }
    GET-VM -VMName $VMName | Get-VMHardDiskDrive | Foreach { Remove-item -path $_.Path -Recurse -Force -Confirm:$False}
    Remove-VM -VMName $VMName -force
    Write-Host "Видалено."
}
```
В тій же ж консолі запускаємо наступний блок. Він готує команду, яку потрібно виконати у консолі "Windows Deployment and Imaging Tools Environment" - посилання на неї можна знайти в меню Пуск після встановлення згаданого вище [Windows ADK](https://learn.microsoft.com/en-us/windows-hardware/get-started/adk-install). Запускаємо скрипт, читаємо підказки на екрані: 

```powershell
$VirtualMachines.GetEnumerator() | ForEach-Object {

    $TempVHD = $RootDir + "temp-"  + $($_.key) + ".vhdx"

    $IsoOriginal = $IsoOriginalDir + $($_.key) + ".iso"

    $IsoUnpacked = $RootDir + $($_.key)

    $IsoLetter = Mount-DiskImage -ImagePath $IsoOriginal -Access ReadOnly | Get-Volume
    $IsoLetter = ($IsoLetter | Get-Volume).DriveLetter

    robocopy $("$IsoLetter" + ":") $IsoUnpacked /e /NFL /NDL /NJH /NJS /nc /ns /np

    Dismount-DiskImage -ImagePath $IsoOriginal

    del $IsoUnpacked\sources\install.wim -force

    $VHDLetter = (Mount-VHD -Path $TempVHD -PassThru | Get-Disk | Get-Partition | Get-Volume).DriveLetter

    $wimpath = $VHDLetter + ':\install-' + $($_.key) + '.wim'
    dism /export-image /sourceImageFile:$wimpath /sourceIndex:1 /destinationImageFile:$IsoUnpacked\sources\install.wim /compress:max /checkIntegrity

    Dismount-VHD -Path $TempVHD

    $IsoRepacked = $RootDir + $($_.key) + '.iso'

    Write-Host
    Write-Host "oscdimg.exe -u2 -b$IsoUnpacked\boot\etfsboot.com -h $IsoUnpacked $IsoRepacked"
    Write-Host
    Read-Host "Скопіюйте в ADK попередню команду, дочекайтесь завершення виконання, натисніть тут Enter"

    Remove-Item -Path $TempVHD -force
    Write-Host "$TempVHD видалено"

    Remove-Item -Path $IsoUnpacked -Recurse -Force
    Write-Host "$IsoUnpacked видалено"
}
```
В результаті ми маємо отримати директорію $RootDir, а в ній:

- очікувані ISO-файли (чи файл)
- директорія vm_backup із копіями віртуальних машин, налаштованими до виконання sysprep

Віртуальні машини, над якими виконувалась операція sysprep, були автоматично видалені на попередніх етапах.

Наступний сценарій може підготувати нові віртуальні комп'ютери і запустити їх із щойно підготовлених ISO для перевірки результатів:

```powershell
$RootDir        = "D:\win_repack\2025_11_20\"

$VMNamePrefix   = "test-"
$HyperVSwitch   = "Default Switch"

$VirtualMachines = @{
    "win7sp1ua-home-basic-x32"   = "Windows 7 Home Basic"
    "win7sp1ua-home-premium-x32" = "Windows 7 Home Premium"
    "win7sp1ua-home-premium-x64" = "Windows 7 Home Premium"
    "win7sp1ua-starter-x32"      = "Windows 7 Starter"
}

$VirtualMachines.keys | ForEach-Object {
    $VMName = $VMNamePrefix + $_

    $VMRAM = 4GB
    if ($VMName.Contains("starter")) {
        $VMRAM = 2GB # можна і більше, але ОС не побачить додаткову пам'ять
    }
    $VirtualDrive = $VMDrivesDir + $VMName + '.vhdx'
    $ISO = $RootDir + $_ + ".iso"
    
    New-VM -Name $VMName -MemoryStartupBytes $VMRAM -NewVHDPath $VirtualDrive -NewVHDSizeBytes 40GB -Generation 1
    Set-VMMemory $VMName -DynamicMemoryEnabled $false
    Set-VMProcessor $VMName -Count 2
    Get-VMScsiController -VMName $VMName -ControllerNumber 0 | Remove-VMScsiController
    Get-VMNetworkAdapter -VMName $VMName | Remove-VMNetworkAdapter -Confirm:$false
    Add-VMNetworkAdapter -VMName $VMName -Name "Legacy Network Adapter" -SwitchName $HyperVSwitch -IsLegacy $true
    Set-VMDvdDrive -VMName $VMName -Path $ISO
    Set-VM -Name $VMName -AutomaticCheckpointsEnabled $false
}
```
