# Corridor

You have found yourself in a strange corridor. Can you find your way back to where you came?

In this challenge, you will explore potential IDOR vulnerabilities. Examine the URL endpoints you access as you navigate the website and note the hexadecimal values you find (they look an awful lot like a hash, don't they?). This could help you uncover website locations you were not expected to access.

---

nmap scan 

```abap
PORT   STATE SERVICE REASON
80/tcp open  http    syn-ack ttl 63
```

Going into the website we get a image of corridors and when we click each one it gives a set path and a empty room’

<img width="1856" height="971" alt="Screenshot From 2025-09-09 17-07-44" src="https://github.com/user-attachments/assets/e1d33a40-4dfb-493b-964a-bffbaad01a4a" />


<img width="1856" height="971" alt="Screenshot From 2025-09-09 17-07-59" src="https://github.com/user-attachments/assets/59d21731-1929-46ad-832f-75911c85b606" />


Looking at the hint above we know that the paths are all hashes. After copying down all the hashes and dehashing them we get this

```abap
Hash	Type	Result
e4da3b7fbbce2345d7772b0674a318d5	md5	5
1679091c5a880faf6fb5e6087eb1b2dc	md5	6
a87ff679a2f3e71d9181a67b7542122c	md5	4
eccbc87e4b5ce2fe28308fd9f2a7baf3	md5	3
c81e728d9d4c2f636f067f89cc14862c	md5	2
c4ca4238a0b923820dcc509a6f75849b	md5	1
8f14e45fceea167a5a36dedd4bea2543	md5	7
c51ce410c124a10e0db5e4b97fc2af39	md5	13
c20ad4d76fe97759aa27a0c99bff6710	md5	12
6512bd43d9caa6e02c990b0a82652dca	md5	11
d3d9446802a44259755d38e6d163e820	md5	10
45c48cce2e2d7fbdea1afc51c7c6ad26	md5	9
c9f0f895fb98ab9159f51fd0297e236d	md5	8
```

Which got me thinking we can hash a number and change the parameter. I know its a md5 hash. 

So lets try 0 with this easy command

```abap
echo -n "0" | md5sum

cfcd208495d565ef66e7dff9f98764da

http://corridor.thm/cfcd208495d565ef66e7dff9f98764da
```

Flag 

> flag{2477ef02448ad9156661ac40a6b8862e}
>
