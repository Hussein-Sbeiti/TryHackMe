# TakeOver

## Intro

Hello there,

I am the CEO and one of the co-founders of **futurevera.thm**. In Futurevera, we believe that the future is in space. We do a lot of space research and write blogs about it. We used to help students with space questions, but we are rebuilding our support.

Recently, blackhat hackers approached us saying they could take over and are asking us for a big ransom. Please help us to find what they can take over.

Our website is located at:
`https://futurevera.thm`

**Hint:** Don’t forget to add the MACHINE\_IP in `/etc/hosts` for `futurevera.thm`.

---

## Nmap

Command used:

```bash
nmap -p- -T4 -A futurevera.thm -vvv
```

Scan output:

* Open ports: **22, 80, 443**
* Services:

  * `22/tcp` → OpenSSH 8.2p1 (Ubuntu)
  * `80/tcp` → Apache 2.4.41 (Ubuntu) – redirects to HTTPS
  * `443/tcp` → Apache 2.4.41 with SSL

SSL certificate info shows it is registered to `futurevera.thm` with organizational data.

<img width="1920" height="1080" alt="Screenshot From 2025-08-24 21-16-11" src="https://github.com/user-attachments/assets/944b671d-2d02-4b12-ae45-2b60a6cec23f" />


---

## Website Analysis

Checking the website source code reveals nothing interesting.

Next step: use Gobuster for virtual host enumeration.

Command used:

```bash
export MIP=10.X.X.X   # replace with actual target IP
gobuster vhost -u https://$MIP -w /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-5000.txt futurevera.thm -t 50 -k
```

Result:
```
Found: portal.futurevera.thm (Status: 200) [Size: 69]
```
<img width="1920" height="1080" alt="Screenshot From 2025-08-24 21-54-53" src="https://github.com/user-attachments/assets/7a749b00-6c48-416e-ad96-fa7cea6cd5f1" />

---

## Exploring Domains

Looking at the site shows a basic portal page.
The company mentions **blogs**:

<img width="1920" height="1080" alt="Screenshot From 2025-08-24 21-57-15" src="https://github.com/user-attachments/assets/fd0180ab-8409-4ff8-8c9d-2bd0b3639184" />

Nothing useful found here.

They also mention **support**, so we try that:

<img width="1920" height="1080" alt="Screenshot From 2025-08-24 21-59-46" src="https://github.com/user-attachments/assets/c2be599e-904c-423d-aec8-1c6efa89b200" />

---

## SSL Certificate Enumeration

By viewing the SSL certificate details:

1. Click the lock icon (upper left in browser).
2. Select **Connection not secure** → **More information**.
3. A new screen reveals more domains linked.

<img width="1920" height="1080" alt="Screenshot From 2025-08-24 22-01-20" src="https://github.com/user-attachments/assets/e52caf06-8eb4-4ae0-8818-f046c1e296ec" />

Add the discovered domain to `/etc/hosts`:

<img width="1920" height="1080" alt="Screenshot From 2025-08-24 22-03-20" src="https://github.com/user-attachments/assets/b6e01503-cafc-4509-a454-aeb56c1a4bef" />

---

## Flag

Once accessed, the flag is revealed:

```
flag{beea0d6edfcee06a59b83fb50ae81b2f}
```

---

## Conclusion

* Performed reconnaissance with `nmap` and discovered open services.
* Enumerated virtual hosts with `gobuster`.
* Checked SSL certificate information for hidden domains.
* Discovered and accessed the correct domain.
* Retrieved the final flag.

---
