---

title: CyberDefenders Writeup  Pwned-DC
#tags:
#cover: https://cyberdefenders.org/media/terraform/Phishy/phishy.png
#categories:
#- [CyberDefenders]
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

> 1- What is the OS Product name of PC01?

Software\Microsoft\WindowsNT\CurrentVersion

![](2022-09-08-12-19-26.png)

Answer:  Windows 10 Enterprise 2016 LTSB

> 2- On 21st November, there was unplanned power off for PC01 machine. How long was PC01 powered on till this shutdown?

In this type of question windows events will be our saver! every event in windows has his own Event ID 

The shutdown event ID is 1074 ! Wow this is awesome right ? the more you know about the event ID the more you are good investigator ! Let's get the system.evtx file and search about events with 1074 ID in 11/21/2021

![](2022-09-08-12-24-04.png)

![](2022-09-08-12-55-33.png)

![](2022-09-08-14-23-18.png)

ٍ![](2022-09-08-14-25-45.png)

> 3 - 

![](2022-09-08-14-28-41.png)

> 4 - 

![](2022-09-08-14-31-24.png)

> 5- 

System32\drivers\etc\services.
![](2022-09-08-14-46-55.png)


> 6- 

![](2022-09-08-15-42-07.png)


> 7- 

NTUSER.DAT: SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\MountPoints2

![](2022-09-08-15-19-57.png)

![](2022-09-08-15-54-52.png)


> 8-
![](2022-09-08-15-54-29.png)


> 11 - 

![](2022-09-08-16-07-03.png)


> 12 - 

```bash command line prompt 
vol.py -f memory.dmp --profile=Win2016x64_14393 hivelist                                                                                 ─╯
Volatility Foundation Volatility Framework 2.6.1
Virtual            Physical           Name
------------------ ------------------ ----
0xffff860c0e681000 0x000000012bfa0000 \??\C:\Windows\AppCompat\Programs\Amcache.hve
0xffff860c0e915000 0x00000000047de000 \??\C:\Users\Administrator\ntuser.dat
0xffff860c0e9ff000 0x000000000eb6a000 \??\C:\Users\Administrator\AppData\Local\Microsoft\Windows\UsrClass.dat
0xffff860c0ed2c000 0x0000000012a19000 \??\C:\ProgramData\Microsoft\Windows\AppRepository\Packages\Microsoft.Windows.
0xffff860c07c28000 0x000000000020c000 [no name]
0xffff860c07c41000 0x0000000000f9a000 \REGISTRY\MACHINE\SYSTEM
0xffff860c07c83000 0x000000010c3b2000 \REGISTRY\MACHINE\HARDWARE
0xffff860c09a71000 0x0000000004644000 \Device\HarddiskVolume2\EFI\Microsoft\Boot\BCD
0xffff860c0831e000 0x00000000040f7000 \SystemRoot\System32\Config\SOFTWARE
0xffff860c0dbc4000 0x000000000220b000 \SystemRoot\System32\Config\DEFAULT
0xffff860c09545000 0x0000000109934000 \SystemRoot\System32\Config\SECURITY
0xffff860c0959f000 0x000000010a3bf000 \SystemRoot\System32\Config\SAM
0xffff860c0dbcc000 0x000000010b689000 \??\C:\Windows\ServiceProfiles\NetworkService\NTUSER.DAT
0xffff860c0dcb6000 0x000000010d3f8000 \SystemRoot\System32\Config\BBI
0xffff860c0dcba000 0x000000010d411000 \??\C:\Windows\ServiceProfiles\LocalService\NTUSER.DAT

```

0x00000000040f7000

