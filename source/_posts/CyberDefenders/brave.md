---
title: CyberDefenders Writeup Brave
date: 2022-09-06 11:29:37
tags:
cover: https://cyberdefenders.org/media/terraform/Brave/brave2.png
categories:
- [CyberDefenders]
---

This CTF challenge is about retrieving data from a memory dump, and analyzing the processes and connections and dealing with registries!

# Challenge Information 


+++info Description

A memory image was taken from a seized Windows machine. Analyze the image and answer the provided questions.

+++

Challenge Link & Author 

{% links %}
- site: Challenge Link 
  url: https://cyberdefenders.org/blueteam-ctf-challenges/67
  desc: CyberDefenders - Brave 
  image: https://cyberdefenders.org/static/img/cyberdefenders-logo-white.png
  color: "#2296fd"
- site: Author - DFIRScience
  desc: DFIRScience Twitter Profile
  url: https://twitter.com/DFIRScience
  image: https://pbs.twimg.com/profile_images/1517319188167204866/lgRHWtGk_400x400.jpg
  color: "#fcc11c"

{% endlinks %}

# Walkthrough 
Memory acquisition  involves copying the contents of volatile memory to non-volatile storage. This is arguably one of the most important and precarious steps in the memory forensics process
The memory dump will contains many usefull information like the time of the memory acquisition, The KB version, The build number and version of the operating system 

> 1- What time was the RAM image acquired according to the suspect system? (YYYY-MM-DD HH:MM:SS)

We can inspect these information using `imageinfo` plugin in volatility
```bash command line prompt
vol3 -f 20210430-Win10Home-20H2-64bit-memdump.mem windows.info 

Kernel Base     0xf8043cc00000
DTB     0x1aa000
Symbols ........
Is64Bit True
IsPAE   False
layer_name      0 WindowsIntel32e
memory_layer    1 FileLayer
KdVersionBlock  0xf8043d80f368
Major/Minor     15.19041
MachineType     34404
KeNumberProcessors      4
SystemTime      2021-04-30 17:52:19
NtSystemRoot    C:\Windows
NtProductType   NtProductWinNt
NtMajorVersion  10
NtMinorVersion  0
PE MajorOperatingSystemVersion  10
PE MinorOperatingSystemVersion  0
PE Machine      34404
PE TimeDateStamp        Tue Oct 11 07:04:26 1977
```

:::success
Answer:  2021-04-30 17:52:19
:::


> 2- What is the SHA256 hash value of the RAM image?

```bash command line prompt
sha256sum 20210430-Win10Home-20H2-64bit-memdump.mem
9db01b1e7b19a3b2113bfb65e860fffd7a1630bdf2b18613d206ebf2aa0ea172 20210430-Win10Home-20H2-64bit-memdump.mem
```
:::success
Answer:  9db01b1e7b19a3b2113bfb65e860fffd7a1630bdf2b18613d206ebf2aa0ea172
:::
> 3- What is the process ID of "brave.exe"?

```bash command line prompt
vol3 -f 20210430-Win10Home-20H2-64bit-memdump.mem windows.pslist | grep -i brave
PID     PPID    ImageFileName   Offset(V)       Threads Handles SessionId       Wow64   CreateTime              ExitTime       
4856    1872    brave.exe       0xbf0f6ca782c0  0       -       1               False    2021-04-30 17:48:45    2021-04-30 17:50:56
```
:::success
Answer: 4856
:::
> 4- How many established network connections were there at the time of acquisition? (number)
```bash command line prompt
vol3 -f 20210430-Win10Home-20H2-64bit-memdump.mem windows.netscan  | grep -i "2021-04-30" | grep -i ESTABLISHED
0xbf0f6a53ca20  TCPv4   10.0.2.15       49833   52.230.222.68   443     ESTABLISHED     2812    svchost.exe     2021-04-30 17:50:07
0xbf0f6ad16050  TCPv4   10.0.2.15       49829   142.250.191.208 443     ESTABLISHED     5624    svchost.exe     2021-04-30 17:49:58
0xbf0f6ad1fad0  TCPv4   10.0.2.15       49847   52.230.222.68   443     ESTABLISHED     2812    svchost.exe     2021-04-30 17:52:17
0xbf0f6c6352b0  TCPv4   10.0.2.15       49842   52.113.196.254  443     ESTABLISHED     5104    SearchApp.exe   2021-04-30 17:51:25
0xbf0f6c7104d0  TCPv4   10.0.2.15       49778   185.70.41.130   443     ESTABLISHED     1840    chrome.exe      2021-04-30 17:45:00
0xbf0f6cd4fa20  TCPv4   10.0.2.15       49837   204.79.197.200  443     ESTABLISHED     5104    SearchApp.exe   2021-04-30 17:51:18
0xbf0f6d0c64a0  TCPv4   10.0.2.15       49843   204.79.197.222  443     ESTABLISHED     5104    SearchApp.exe   2021-04-30 17:51:26
0xbf0f6d51c4a0  TCPv4   10.0.2.15       49838   13.107.3.254    443     ESTABLISHED     5104    SearchApp.exe   2021-04-30 17:51:23
0xbf0f6d525a20  TCPv4   10.0.2.15       49845   23.101.202.202  443     ESTABLISHED     1156    MsMpEng.exe     2021-04-30 17:51:36
0xe80000193a20  TCPv4   10.0.2.15       49845   23.101.202.202  443     ESTABLISHED     1156    MsMpEng.exe     2021-04-30 17:51:36
```
Oh wait man! Don't count it manually, just use the magic of `wc -l` !
```bash command line prompt
vol3 -f 20210430-Win10Home-20H2-64bit-memdump.mem windows.netscan  | grep -i "2021-04-30" | grep -i ESTABLISHED | wc -l

10
```
:::success
Answer: 10 
:::
> 5- What FQDN does Chrome have an established network connection with?
```bash command line prompt
 vol3 -f 20210430-Win10Home-20H2-64bit-memdump.mem windows.netscan | grep -i chrome
0xbf0f6a896ae0.0TCPv4   10.0.2.15DB scan49773fin185.70.41.35    443     FIN_WAIT2       1840    chrome.exe      2021-04-30 17:44:58
0xbf0f6c7104d0  TCPv4   10.0.2.15       49778   185.70.41.130   443     ESTABLISHED     1840    chrome.exe      2021-04-30 17:45:00
0xbf0f6c85bb20  TCPv4   10.0.2.15       49775   185.70.41.35    443     FIN_WAIT2       1840    chrome.exe      2021-04-30 17:44:58
0xbf0f6ca71a20  TCPv4   10.0.2.15       49769   142.250.190.14  443     CLOSE_WAIT      1840    chrome.exe      2021-04-30 17:44:55
0xbf0f6cbb9530  TCPv4   10.0.2.15       49772   185.70.41.35    443     FIN_WAIT2       1840    chrome.exe      2021-04-30 17:44:58
0xbf0f6cfd17f0  TCPv4   10.0.2.15       49777   35.186.220.63   443     CLOSE_WAIT      1840    chrome.exe      2021-04-30 17:44:58
0xbf0f6d51c010  TCPv4   10.0.2.15       49763   172.217.4.35    443     CLOSE_WAIT      1840    chrome.exe      2021-04-30 17:44:53
0xbf0f6d5c8a70  TCPv4   10.0.2.15       49797   172.217.4.74    443     CLOSE_WAIT      1840    chrome.exe      2021-04-30 17:48:27
0xbf0f6d5d1ac0  TCPv4   10.0.2.15       49770   185.70.41.35    443     FIN_WAIT2       1840    chrome.exe      2021-04-30 17:44:57
0xbf0f6d8a1010  TCPv4   10.0.2.15       49771   185.70.41.35    443     CLOSE_WAIT      1840    chrome.exe      2021-04-30 17:44:57
```
Wow man! we found a single established connection with IP `185.70.41.130`
I used a random website that give me the FQDN from the IP address!

![](https://imgur.com/XJSnlsS.png)

:::success
Answer: `protonmail.ch`
:::
> 6- What is the MD5 hash value of process executable for PID 6988?

Let's dump the executable of the process with pid 6988, I used to do that with `procdump` plugin with volatility2, but this time i am trying to use volatility3 so with somesearch i found that `windows.pslist` plugin has `--dump` argument that can dump the excutable of the process. Let's do it !
```bash command line prompt
vol3 -f 20210430-Win10Home-20H2-64bit-memdump.mem windows.pslist --pid 6988 --dump 
Volatility 3 Framework 2.2.0
Progress:  100.00               PDB scanning finished
PID     PPID    ImageFileName   Offset(V)       Threads Handles SessionId       Wow64   CreateTime      ExitTime        File output

6988    4352    OneDrive.exe    0xbf0f6d4262c0  26      -       1       True    2021-04-30 17:40:01     N/A     pid.6988.0x1c0000.dmp
```
Now it's time to use `md5sum` to get the md5 hash of the executable!
```bash command line prompt
md5sum pid.6988.0x1c0000.dmp
0b493d8e26f03ccd2060e0be85f430af  pid.6988.0x1c0000.dmp
```
:::success
Answer: 0b493d8e26f03ccd2060e0be85f430af
:::
> 7- What is the word starting at offset 0x45BE876 with a length of 6 bytes?
OK we need to know the word at offset 0x45BE876! we can do it using `xxd` just we start from this offest and stop after 6 bytes! to specify the offset that you want to start from it just use `-s` options and to specify the length to display use `-l` option

```bash command line prompt
xxd -g1 --s 0x45be876 -l 6 20210430-Win10Home-20H2-64bit-memdump.mem 
045be876: 68 61 63 6b 65 72                                hacker
```
:::success
Answer: hacker
:::
> 8- What is the creation date and time of the parent process of "powershell.exe"? (YYYY-MM-DD HH:MM:SS)

We need to know the parent process of powershell! we can do it using `pslist` and check the `PPID`. But I'll go with `pstree` plugin! it will make it easier for me. I don't want to waste my time searching at the parent process using PPID !

```bash command line prompt
vol3 -f 20210430-Win10Home-20H2-64bit-memdump.mem windows.pstree |  grep -i powershell -B 4
* 564   668     LogonUI.exe     0xbf0f6b7b7100  0       -       1       False   2021-04-30 12:39:44      2021-04-30 17:39:58
* 4296  668     userinit.exe    0xbf0f6ca8f080  0       -       1       False   2021-04-30 17:39:48      2021-04-30 17:40:12
** 4352 4296    explorer.exe    0xbf0f6ca662c0  82      -       1       False   2021-04-30 17:39:48      N/A
*** 6884        4352    VBoxTray.exe    0xbf0f6d186080  11      -       1       False   2021-04-30 17:40:01      N/A
*** 5096        4352    powershell.exe  0xbf0f6d97f2c0  12      -       1       False   2021-04-30 17:51:19      N/A
```
:::success
Answer: 2021-04-30 17:39:48
:::

> 9- What is the full path and name of the last file opened in notepad?

Notepad is an executable that use the file as an argument ! so `cmdline` plugin will help us the get the files opened with notepad ! 

```bash command line prompt
vol3 -f 20210430-Win10Home-20H2-64bit-memdump.mem windows.cmdline | grep -i notepad   
2520 notepad.exe     "C:\Windows\system32\NOTEPAD.EXE" C:\Users\JOHNDO~1\AppData\Local\Temp\7zO4FB31F24\accountNum
```
:::success
Answer:  C:\Users\JOHNDO~1\AppData\Local\Temp\7zO4FB31F24\accountNum
:::
> 10- How long did the suspect use Brave browser? (hh:mm:ss)

Let me tell you something! When we speak about more details about system apps nothing will be usefull more than the windows registries! One of the most important registries is `UserAssist`
The `UserAssist` key, a part of Microsoft Windows registry, records the information related to programs run by a user on a Windows system

Volatility is powerful! we can use the `userassist` plugin to get what we want!

``` bash command line prompt 
vol3 -f 20210430-Win10Home-20H2-64bit-memdump.mem windows.registry.userassist | grep -i brave 
* 0xa80333cda0000       \??\C:\Users\John Doe\ntuser.dat        ntuser.dat\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\UserAssist\{CEBFF5CD-ACE2-4F4F-9178-9926F41749EA}\Count  2021-04-30 17:52:18      Value   %ProgramFiles%\BraveSoftware\Temp\GUM20E0.tmp\BraveUpdate.exe N/A      0       0       0:00:03  N/A
* 0xa80333cda000        \??\C:\Users\John Doe\ntuser.dat        ntuser.dat\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\UserAssist\{CEBFF5CD-ACE2-4F4F-9178-9926F41749EA}\Count  2021-04-30 17:52:18      Value   %ProgramFiles%\BraveSoftware\Update\BraveUpdate.exe     N/A   0:00:24.797000   N/A
* 0xa80333cda000        \??\C:\Users\John Doe\ntuser.dat        ntuser.dat\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\UserAssist\{CEBFF5CD-ACE2-4F4F-9178-9926F41749EA}\Count  2021-04-30 17:52:18      Value   Brave   N/A     9       22      4:01:54  2021-04-30 17:48:45
* 0xa80333cda000        \??\C:\Users\John Doe\ntuser.dat        ntuser.dat\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\UserAssist\{F4E57C4B-2036-45F0-A9AB-443BCFE33D9F}\Count  2021-04-30 17:51:18      Value   C:\Users\Public\Desktop\Brave.lnk       N/A     8       0     0:00:00.508000   2021-04-30 17:48:45
```
:::success
Answer:  04:01:54
:::


