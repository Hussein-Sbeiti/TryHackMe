# MD2PDF

## Nmap

```bash
nmap -sV -p- -T4 <IP> -vv
```

```
PORT     STATE SERVICE REASON
22/tcp   open  ssh     syn-ack ttl 64
80/tcp   open  http    syn-ack ttl 63
5000/tcp open  upnp    syn-ack ttl 63
```

---

## Web Enumeration

<img width="1858" height="972" alt="Screenshot From 2025-09-06 22-55-42" src="https://github.com/user-attachments/assets/73e36ffe-6a02-48ad-843a-6a74ce31fc64" />

Looking at this page, anything typed into the Markdown box gets converted into a PDF file.

I suspected the PDF conversion feature could be abused for **SSRF**. To confirm, I hosted a Python HTTP server on my own machine:

```bash
python3 -m http.server 8000
```

When I submitted content, the target made requests to me — confirming the backend fetched external resources.

---

## Exploit

Next, I tested internal resources by embedding an iframe pointing to localhost. Since Nmap showed port 5000 open and dirsearch revealed `/admin`, I used this payload inside the Markdown box:

```html
<iframe src="http://localhost:5000/admin"></iframe>
```

When converted to a PDF, the server rendered the page it fetched from its own localhost. That gave me direct access to the `/admin` interface.

Inside the PDF output, I found the flag.

---

## Flag

> Q1: Flag
> A1: flag{1f4a2b6ffeaf4707c43885d704eaee4b}
