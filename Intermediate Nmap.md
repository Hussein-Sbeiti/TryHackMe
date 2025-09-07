# Intermediate Nmap

## Nmap

```bash
nmap -sV -p- -T4 <IP> -vv
```

```
PORT      STATE SERVICE REASON         VERSION
22/tcp    open  ssh     syn-ack ttl 63 OpenSSH 8.2p1 Ubuntu 4ubuntu0.4 (Ubuntu Linux; protocol 2.0)
2222/tcp  open  ssh     syn-ack ttl 64 OpenSSH 8.2p1 Ubuntu 4ubuntu0.4 (Ubuntu Linux; protocol 2.0)
31337/tcp open  Elite?  syn-ack ttl 63
```

---

## Enumeration

The hint says this VM is listening on a high port, and if you connect to it, it may give you information to use on a lower port commonly used for remote access.

I connected with netcat to port 31337 and got credentials:

```
In case I forget - user:pass
ubuntu:Dafdas!!/str0ng
```

---

## SSH

Using those credentials, I logged in:

```bash
ssh ubuntu@10.201.76.159 -p 22
```

Once in, I got the flag:

> Q1: Flag
> A1: flag{251f309497a18888dde5222761ea88e4}
