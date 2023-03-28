---
layout: post
title: change rdp port bat
date: 2023-03-28
tags: win rdp port
---

三不五時要改 windows rdp port 就找了一下

```
@echo off
echo - Windows Remote Desktop port modification
echo - Tip: The remote port defaults to 3389 (hex 0xd3d)
echo -
echo - Current port (hex):
reg query "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp" /v "PortNumber"
echo -
:: check admin
net session >nul 2>&1
if %errorLevel% == 0 (echo [administrator mode]) else (echo Error: Please right click on the file and run it as administrator & pause & goto :EOF)
:: check admin
set /p rdp_port="Enter the port number to modify (default is 3389):"
if "%rdp_port%" EQU "" set rdp_port=3389
echo - Press any key to confirm to set the remote desktop port to: %rdp_port%
pause
reg add "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp" /v "PortNumber" /t REG_DWORD /d %rdp_port% /f
echo - New port (hex):
reg query "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp" /v "PortNumber"
echo - Add new port to firewall exceptions...
netsh advfirewall firewall add rule name="RDP Port %rdp_port%" profile=any protocol=TCP action=allow dir=in localport=%rdp_port%
echo ---- Press any key to restart the TermService service for the new settings to take effect (the remote desktop will be disconnected)
echo ---- If the remote desktop cannot be connected after being disconnected, try restarting the system to take effect
pause
echo - Restart Remote Desktop Services...
net stop TermService /y
net start TermService /y
:DONE
echo ---- Finish
pause
```

[change_rdp.bat](/images/change_rdp.bat)
