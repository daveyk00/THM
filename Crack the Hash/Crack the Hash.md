# Task 1 Level 1
Can you complete the level 1 tasks by cracking the hashes?
### 48bb6e862e54f2a795ffc4e541caed4d

[hash-id.py](https://gitlab.com/kalilinux/packages/hash-identifier/-/tree/kali/master)

`hash-id.py 48bb6e862e54f2a795ffc4e541caed4d` reports MD5
[crackstation.net](crackstation.net) indicates it is `easy`

We get the same result using hashcat
`hashcat -a 0 -m 0 hash.txt rockyou.txt` where `hash.txt` contains the hash

Answer: `easy`

### CBFDAC6008F9CAB4083784CBD1874F76618D2A97

`python /usr/share/hash-identifier/hash-id.py CBFDAC6008F9CAB4083784CBD1874F76618D2A97` indicates `SHA1`

Hashcat again
`hashcat -a 0 -m 100 hash.txt rockyou.txt`

Answer: `password123`

### 1C8BFE8F801D79745C4631D09FFF36C82AA37FC4CCE4FC946683D7B336B63032

`hash-id.py` indicates SHA256

`hashcat -a 0 -m 1400 hash.txt rockyou.txt`

Answer: `letmein`

### `$2y$12$Dwt1BZj6pcyc3Dy1FWZ5ieeUznr71EeNkJkUlypTsgbX1H68wsRom`


`hash-id.py` is unable to identify this hash.
https://medium.com/infosec-adventures/identifying-and-cracking-hashes-7d580b9fd7f1 shows it as blowfish, lets try that.

It took some time and then looking at the hint it indicated that it would and to limit the number of words in the wordlist.

Attempting brute force 4 lowercase characters would take 9 hours.

To limit the rock you word list

`grep -E "^.{4}$" rockyou.txt > rockyou4.txt`
then hashcat on the `rockyou4.txt`

`hashcat -a 0 -m 3200 hash.txt rockyou4.txt`

Answer: `bleh`

### 279412f945939ba78ce0758d3fd83daa

This looks to be MD4 as well
`hashcat -a 0 -m 900 hash.txt rockyou.txt` did not resolve a solution, crackstation.net did, of `Eternity22`.  This word is not in the rockyou.txt set.

Answer: `Eternity22`

# Task 2 Level 2

This task increases the difficulty. All of the answers will be in the classic [rock you](https://github.com/brannondorsey/naive-hashcat/releases/download/data/rockyou.txt) password list.

You might have to start using hashcat here and not online tools. It might also be handy to look at some example hashes on [hashcats page](https://hashcat.net/wiki/doku.php?id=example_hashes).

### F09EDCB1FCEFC6DFB23DC3505A882655FF77375ED8AA2D1C13F640FCCC2D0C85

Looks like SHA-256

`hashcat -a 0 -m 1400 hash.txt rockyou.txt`

Answer: `paule`

### 1DFECA0C002AE40B8619ECF94819CC1B

Hint tell us it is NTLM

`hashcat -a 0 -m 1000 hash.txt rockyou.txt`

Answer: `n63umy8lkf4i`

### `$6$aReallyHardSalt$6WKUTqzq.UQQmrm0p/T7MPpMbGNnzXPMAXi4bJMl9be.cfi3/qxIf.hsGpS41BqMhSrHVXgMpdjS6xeKZAs02.`

##### Salt: aReallyHardSalt

Lets try `sha512crypt $6$, SHA512 (Unix) 2`

It is important that the full stop remains on the line in hash.txt

`hashcat -a 0 -m 1720 hash.txt rockyou.txt`

I also trimmed rockyou to words with 6 characters.

Answer: `waka99`

### Hash: e5d8870e5bdd26602cab8dbe07a942c8669e56d6

##### Salt: tryhackme

Answer mask is 12 characters, so create a rockyou word list for that
`grep -E "^.{12}$" rockyou.txt > rockyou12.txt`

`hashcat -a 0 -m 150 hash.txt rockyou12.txt` didn't find the solution, however

`hashcat -a 0 -m 160 hash.txt rockyou12.txt` did

Answer:  `481616481616`