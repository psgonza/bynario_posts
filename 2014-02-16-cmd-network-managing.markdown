title: Network manager for windows 7 using cmd.exe
date: 2014-02-16 13:14
slug: 2014-02-16-cmd-network-managing
category: posts
tag: - windows , networking , coding
---
{% img center /img/cmd_exe.JPG 'cmd.exe' %}

> Updated version of the script. Now it allows you to enable/disable the network interfaces

Hi there

Lately I have been changing the network properties (ip addresses, netmask, etc etc) in my laptop (W7 Enterprise 64b) several times per day, and to be honest I am growing tired of it, so I created a small bat script in order to make my life easier

The script is pretty simple... It might be handy for you as well, so here you are:


```
@ECHO OFF
SET OPTION=

goto :MENU

:MENU
cls
Echo #### CLI NETWORK MANAGEMENT ####
Echo.  
Echo ****** LAN ******
Echo 1) LAN - DHCP
Echo 2) LAN - STATIC IP:192.168.2.10/24 - GW:192.168.2.99 - DNS:8.8.8.8
Echo 3) LAN - DEFINE NEW STATIC IP
Echo.
Echo ****** WIRELESS LAN ******
Echo 4) WLAN - DHCP
Echo 5) WLAN - STATIC IP:192.168.2.20/24 - GW:192.168.2.99 - DNS:8.8.8.8
Echo 6) WLAN - DEFINE NEW STATIC IP
Echo.
Echo ****** INTERFACES ******
Echo 7) SHOW INTERFACES STATUS
Echo 8) ENABLE LAN
Echo 9) DISABLE LAN
Echo 10) ENABLE WLAN
Echo 11) DISABLE WLAN
Echo.
Echo 0) Exit
Echo. 
set /p OPTION=Option: 
Echo. 

IF "%OPTION%"=="0" call :END
IF "%OPTION%"=="1" call :Opt1 "Local Area Connection"
IF "%OPTION%"=="2" call :Opt2 "Local Area Connection" "192.168.2.10" "255.255.255.0" "192.168.2.99" "8.8.8.8"
IF "%OPTION%"=="3" call :Opt3 "Local Area Connection"
IF "%OPTION%"=="4" call :Opt1 "Wireless Network Connection"
IF "%OPTION%"=="5" call :Opt2 "Wireless Network Connection" "192.168.2.20" "255.255.255.0" "192.168.2.99" "8.8.8.8"
IF "%OPTION%"=="6" call :Opt3 "Wireless Network Connection"
IF "%OPTION%"=="7" call :Opt4 
IF "%OPTION%"=="8" call :Opt41 "Local Area Connection" "ENABLED"
IF "%OPTION%"=="9" call :Opt41 "Local Area Connection" "DISABLED"
IF "%OPTION%"=="10" call :Opt41 "Wireless Network Connection" "ENABLED"
IF "%OPTION%"=="11" call :Opt41 "Wireless Network Connection" "DISABLED"


goto :MENU

:Opt1
REM DHCP
REM "%~1" - Interface Name
cls
Echo Configuring %~1 as DHCP...
Echo. 
Echo Configuring DHCP for IP
netsh interface ip set address "%~1" dhcp
Echo Configuring DHCP for dns
netsh interface ip set dns "%~1" dhcp
Echo Configuring DHCP for wins
netsh int ip set wins name="%~1" dhcp
Echo. 
Echo Checking configuration
netsh interface ip show config name="%~1"
Echo. 
pause
goto :MENU


:Opt2
REM STATIC IP
REM "%~1" - Interface Name
REM "%~2" - IP
REM "%~3" - NETMASK
REM "%~4" - GW
REM "%~5" - DNS
cls
Echo Configuring static IP address for %~1
Echo =============================================
Echo.
Echo Configuring IP %~2 %~3 %~4
netsh interface ip set address name="%~1" static "%~2" "%~3" "%~4"
Echo Configuring DNS %~5
netsh interface ip set dns name="%~1" static "%~5"
netsh interface ip add dns name="%~1" 8.8.4.4
Echo. 
Echo Checking configuration
netsh interface ip show config name="%~1"
pause
goto :MENU

:Opt3
REM STATIC IP - NEW
REM "%~1" - Interface Name
cls
set /p IP=IP:
set /p NET=NETMASK:
set /p GW=GATEWAY:
set /p DNS=DNS:
Echo Configuring new static IP address for %~1
Echo =============================================
Echo.
Echo Configuring IP %IP% %NET% %GW%
netsh interface ip set address name="%~1" static "%IP%" "%NET%" "%GW%"
Echo Configuring DNS %DNS%
netsh interface ip set dns name="%~1" static "%DNS%" 
Echo Configuring secondary DNS 8.8.4.4
netsh interface ip add dns name="%~1" 8.8.4.4
Echo. 
Echo Checking configuration
netsh interface ip show config name="%~1"
pause
goto :MENU

:Opt4
REM INTERFACES
REM "%~1" - Interface Name
REM "%~2" - status
cls
Echo ****** INTERFACES ******
Echo. 
Echo Checking network interface status...
Echo.
netsh interface  show interface | find /i "connection"
Echo.
pause
goto :MENU

:Opt41
cls
Echo ****** INTERFACES ******
Echo. 
Echo Setting %~1 status to ... %~2 
netsh interface set interface name="%~1" admin="%~2" 
goto :Opt4


:END
exit
```

The script needs to be executed as administrator...
