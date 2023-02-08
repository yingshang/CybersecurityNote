# PowerShell 搭建AD域渗透环境

> 同一台虚拟机需要重置SID
>
> ```
> %WINDIR%\system32\sysprep\sysprep.exe /generalize /restart /oobe
> ```

## 森林

```powershell
#更改主机名
#Rename-Computer -NewName "dc"

#设置静态地址
New-NetIPAddress –IPAddress  10.10.10.10 -DefaultGateway 10.10.10.1 -PrefixLength 24 -InterfaceIndex (Get-NetAdapter).InterfaceIndex
#设置DNS
Set-DNSClientServerAddress -InterfaceIndex(Get-NetAdapter).InterfaceIndex -ServerAddresses 10.10.10.10

#密码永不过期
Set-LocalUser -Name "administrator" -PasswordNeverExpires 1

#关闭密码复杂度
secedit /export /cfg c:\secpol.cfg
(gc C:\secpol.cfg).replace("PasswordComplexity = 1", "PasswordComplexity = 0") | Out-File C:\secpol.cfg
secedit /configure /db c:\windows\security\local.sdb /cfg c:\secpol.cfg /areas SECURITYPOLICY
rm -force c:\secpol.cfg -confirm:$false

#设置密码最长时间
net accounts /MAXPWAGE:999

#
Install-WindowsFeature RSAT-AD-PowerShell
Get-WindowsFeature -Name RSAT-AD-PowerShell


#获取域安装信息
Get-WindowsFeature "*ad*"
#安装域控
Install-WindowsFeature ad-domain-services -IncludeAllSubFeature -IncludeManagementTools

Get-WindowsFeature "*ad*"

$SecurePwd = ConvertTo-SecureString "abcABC123" -AsPlainText -Force
#建立域
Install-ADDSForest -DomainName "sectest.com" -InstallDNS -SafeModeAdministratorPassword $SecurePwd -NoRebootOnCompletion -Force



#关闭休眠
powercfg.exe -x -monitor-timeout-ac 0
powercfg.exe -x -monitor-timeout-dc 0
powercfg.exe -x -disk-timeout-ac 0
powercfg.exe -x -disk-timeout-dc 0
powercfg.exe -x -standby-timeout-ac 0
powercfg.exe -x -standby-timeout-dc 0
powercfg.exe -x -hibernate-timeout-ac 0
powercfg.exe -x -hibernate-timeout-dc 0

#关闭防火墙
Set-NetFirewallProfile -Profile Domain,Public,Private -Enabled False
Get-NetFirewallProfile

#关闭Windows AV
Set-MpPreference -DisableRealtimeMonitoring $true
Set-MpPreference -DisableArchiveScanning $True
Set-MpPreference -DisableRemovableDriveScanning $True
Set-MpPreference -SubmitSamplesConsent 2
Set-MpPreference -MAPSReporting Disable
#Uninstall-WindowsFeature -Name Windows-Defender

#设置域用户密码永不过期
#Import-Module ActiveDirectory
#Get-ADUser -Filter * | Set-ADUser -PasswordNeverExpires:$True

#重启电脑
Restart-Computer 

```

## 子域

```powershell
#设置静态地址
New-NetIPAddress –IPAddress  10.10.10.20 -DefaultGateway 10.10.10.1 -PrefixLength 24 -InterfaceIndex (Get-NetAdapter).InterfaceIndex
#设置DNS
Set-DNSClientServerAddress -InterfaceIndex(Get-NetAdapter).InterfaceIndex -ServerAddresses 10.10.10.10

#密码永不过期
Set-LocalUser -Name "administrator" -PasswordNeverExpires 1

#关闭密码复杂度
secedit /export /cfg c:\secpol.cfg
(gc C:\secpol.cfg).replace("PasswordComplexity = 1", "PasswordComplexity = 0") | Out-File C:\secpol.cfg
secedit /configure /db c:\windows\security\local.sdb /cfg c:\secpol.cfg /areas SECURITYPOLICY
rm -force c:\secpol.cfg -confirm:$false

#设置密码最长时间
net accounts /MAXPWAGE:999

#关闭休眠
powercfg.exe -x -monitor-timeout-ac 0
powercfg.exe -x -monitor-timeout-dc 0
powercfg.exe -x -disk-timeout-ac 0
powercfg.exe -x -disk-timeout-dc 0
powercfg.exe -x -standby-timeout-ac 0
powercfg.exe -x -standby-timeout-dc 0
powercfg.exe -x -hibernate-timeout-ac 0
powercfg.exe -x -hibernate-timeout-dc 0

#关闭防火墙
Set-NetFirewallProfile -Profile Domain,Public,Private -Enabled False
Get-NetFirewallProfile

#关闭Windows AV
Set-MpPreference -DisableRealtimeMonitoring $true
Set-MpPreference -DisableArchiveScanning $True
Set-MpPreference -DisableRemovableDriveScanning $True
Set-MpPreference -SubmitSamplesConsent 2
Set-MpPreference -MAPSReporting Disable


#
Install-WindowsFeature RSAT-AD-PowerShell
Get-WindowsFeature -Name RSAT-AD-PowerShell

#获取域安装信息
Get-WindowsFeature "*ad*"
#安装域控
Install-WindowsFeature ad-domain-services -IncludeAllSubFeature -IncludeManagementTools
#
$SecurePwd = ConvertTo-SecureString "abcABC123" -AsPlainText -Force
$username = "sectest.com\administrator"
$password = "abcABC123" | ConvertTo-SecureString -asPlainText -Force
$Credential =New-Object System.Management.Automation.PSCredential($username, $password)


Install-ADDSDomain -Credential $Credential -NewDomainName children -ParentDomainName "sectest.com" -domaintype childdomain  -NewDomainNetBIOSName chilesec -InstallDNS -CreateDNSDelegation  -NoRebootOnCompletion -SafeModeAdministratorPassword $SecurePwd 


#重启电脑
Restart-Computer 
```

## PC

> ```
> set-executionpolicy remotesigned 
> ```

```powershell
#设置静态地址
New-NetIPAddress –IPAddress  10.10.10.40 -DefaultGateway 10.10.10.1 -PrefixLength 24 -InterfaceAlias  Ethernet0

#设置DNS
Set-DNSClientServerAddress -InterfaceIndex(Get-NetAdapter).InterfaceIndex -ServerAddresses 10.10.10.10

#密码永不过期
Set-LocalUser -Name "administrator" -PasswordNeverExpires 1

#关闭密码复杂度
secedit /export /cfg c:\secpol.cfg
(gc C:\secpol.cfg).replace("PasswordComplexity = 1", "PasswordComplexity = 0") | Out-File C:\secpol.cfg
secedit /configure /db c:\windows\security\local.sdb /cfg c:\secpol.cfg /areas SECURITYPOLICY
rm -force c:\secpol.cfg -confirm:$false

#设置密码最长时间
net accounts /MAXPWAGE:999

#关闭休眠
powercfg.exe -x -monitor-timeout-ac 0
powercfg.exe -x -monitor-timeout-dc 0
powercfg.exe -x -disk-timeout-ac 0
powercfg.exe -x -disk-timeout-dc 0
powercfg.exe -x -standby-timeout-ac 0
powercfg.exe -x -standby-timeout-dc 0
powercfg.exe -x -hibernate-timeout-ac 0
powercfg.exe -x -hibernate-timeout-dc 0

#关闭防火墙
Set-NetFirewallProfile -Profile Domain,Public,Private -Enabled False
Get-NetFirewallProfile

#关闭Windows AV
Set-MpPreference -DisableRealtimeMonitoring $true
Set-MpPreference -DisableArchiveScanning $True
Set-MpPreference -DisableRemovableDriveScanning $True
Set-MpPreference -SubmitSamplesConsent 2
Set-MpPreference -MAPSReporting Disable

#加入域
$username = "sectest.com\administrator"
$password = "abcABC123" | ConvertTo-SecureString -asPlainText -Force
$Credential =New-Object System.Management.Automation.PSCredential($username, $password)
Add-Computer -DomainName sectest.com -Credential $Credential

#重启电脑
Restart-Computer 
```

