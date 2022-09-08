---
title: CyberDefenders Writeup AfricanFalls
date: 2022-09-06 11:29:37
tags:
cover: https://cyberdefenders.org/media/terraform/AfricanFalls/a2.png
categories:
- [CyberDefenders]
---

This CTF challenge is from AfricaFalls Digital Forensics contest , We have disk image that have a lot of information like password, registries, browser history etc...
We will investigate some useful informations! Be ready 

# Challenge Information 


+++info Description

John Doe was accused of doing illegal activities. A disk image of his laptop was taken. Your task is to analyze the image and understand what happened under the hood.

+++

Challenge Link & Author 

{% links %}
- site: Challenge Link 
  url: https://cyberdefenders.org/blueteam-ctf-challenges/66
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


> 1- What is the MD5 hash value of the suspect disk?

Just inspec the `ad1.txt` and you'll find useful information about the disk image like the acquisition time and the checksum

```bash command line prompt 
cat DiskDrigger.ad1.txt 
Created By AccessData® FTK® Imager 4.5.0.3

Case Information:
Acquired using: ADI4.5.0.3
...
[Computed Hashes]
 MD5 checksum:    9471e69c95d8909ae60ddff30d50ffa1
 SHA1 checksum:   167aa08db25dfeeb876b0176ddc329a3d9f2803a

Image information:
 Acquisition started:   Tue Jun 15 12:28:20 2021
 Acquisition finished:  Tue Jun 15 12:33:10 2021
 Segment list:
  D:\Users\Mawso3a\Desktop\DiskDrigger.ad1

Image Verification Results:
 Verification started:  Tue Jun 15 12:33:18 2021
 Verification finished: Tue Jun 15 12:33:51 2021
 MD5 checksum:    9471e69c95d8909ae60ddff30d50ffa1 : verified
 SHA1 checksum:   167aa08db25dfeeb876b0176ddc329a3d9f2803a : verified
```

:::success
Answer: 9471e69c95d8909ae60ddff30d50ffa1
:::

> 2- What phrase did the suspect search for on 2021-04-29 18:17:38 UTC? (three words, two spaces in between)
We found that the user used a chrome as his main browser! So we decide to get the database that contains the history of browsing. We can inspect it with DB Browser SQLite
![](https://imgur.com/YdY9w1L.png)
After Opening the database you can search about the history and find the correct answer using the right timestamp

![](https://imgur.com/jL9EEDB.png)

:::success
Answer:  password cracking lists
:::

> 3- What is the IPv4 address of the FTP server the suspect connected to?

Wait man! FileZilla is installed in our system. I am sure that we will find the information in `filezilla.xml`

![](https://imgur.com/uOJpYIM.png)

:::success
Answer:  192.168.1.20
:::

> 4- What date and time was a password list deleted in UTC? (YYYY-MM-DD HH:MM:SS UTC)

This is EASY DUDE! just check the Recycle Bin ! You'll find your target ! 
![](https://imgur.com/TuEyvM2.png)

:::success
Answer:  2021-04-29 18:22:17 UTC
:::

> 5- How many times was Tor Browser ran on the suspect's computer? (number only)

I guess this question is tricky! where is tor man !!! OK calm down you'll find `lnk` file about tor but i think it's fake one ! Let's try an LNK Parser. I'll use `ericzimmerman` tool ! 

![](https://imgur.com/GelvVNQ.png)

Wow firefox.exe in TOR LNK file! This is Joke 

:::success
Answer:  0
:::
> 6- What is the suspect's email address?

OK backing again to the browser history to check if the user visit a mailing website !
![](https://imgur.com/BmHa8m2.png)

:::success
Answer:   dreammaker82@protonmail.com
:::

> 7- What is the FQDN did the suspect port scan?

He asked about port scan?? mmm I guess nmap is here ! Nmap are you here ? 
![](https://imgur.com/UMN9Zbr.png "NMAP: YES I AM HERE !")

Just i checked the PowerShell history. You'll find it in `AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine`

![](https://imgur.com/OwOcsXB.png)

:::success
Answer:  dfir.science
:::

> 8- What country was picture "20210429_152043.jpg" allegedly taken in?

I tried to inspect the metadata of the image and i found GPS position information! 
```bash command line prompt
exiftool 20210429_152043.jpg
....
....
GPS Latitude                    : 16 deg 0' 0.00" S
GPS Longitude                   : 23 deg 0' 0.00" E
Focal Length                    : 3.7 mm
GPS Position                    : 16 deg 0' 0.00" S, 23 deg 0' 0.00" E
Light Value                     : 13.7
```
Ok let's try to find the country using GPS Coordinates Finder

![](https://imgur.com/freR9Vd.png)

:::success
Answer:  Zambia
:::

> 9- What is the parent folder name picture "20210429_151535.jpg" was in before the suspect copy it to "contact" folder on his desktop?

Shellbags explorer will solve our problem here! But wait man ! What are ShellBag artifacts?
`ShellBags` are a popular artifact in Windows forensics often used to identify the existence of directories on local, network, and removable storage devices. ShellBags are stored as a highly nested and hierarchal set of subkeys in the UsrClass
This registyr hive will save us : `[root]\Users\John Doe\AppData\Local\Microsoft\Windows\Usrclass.dat`

ٍ![](https://imgur.com/IAJ0GBZ.png)

:::success
Answer:  Camera
:::

> 10- A Windows password hashes for an account are below. What is the user's password?

Just try an online hash cracker 

ٍ![](https://imgur.com/eQ1U5w7.png)

:::success
Answer:   AFR1CA!
:::



