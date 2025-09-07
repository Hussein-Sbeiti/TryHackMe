# W1seGuy

We get this code in the beginning of the room:

```python
import random
import socketserver 
import socket, os
import string

flag = open('flag.txt','r').read().strip()

def send_message(server, message):
    enc = message.encode()
    server.send(enc)

def setup(server, key):
    flag = 'THM{thisisafakeflag}' 
    xored = ""

    for i in range(0,len(flag)):
        xored += chr(ord(flag[i]) ^ ord(key[i%len(key)]))

    hex_encoded = xored.encode().hex()
    return hex_encoded

def start(server):
    res = ''.join(random.choices(string.ascii_letters + string.digits, k=5))
    key = str(res)
    hex_encoded = setup(server, key)
    send_message(server, "This XOR encoded text has flag 1: " + hex_encoded + "\n")
    
    send_message(server,"What is the encryption key? ")
    key_answer = server.recv(4096).decode().strip()

    try:
        if key_answer == key:
            send_message(server, "Congrats! That is the correct key! Here is flag 2: " + flag + "\n")
            server.close()
        else:
            send_message(server, 'Close but no cigar' + "\n")
            server.close()
    except:
        send_message(server, "Something went wrong. Please try again. :)\n")
        server.close()

class RequestHandler(socketserver.BaseRequestHandler):
    def handle(self):
        start(self.request)

if __name__ == '__main__':
    socketserver.ThreadingTCPServer.allow_reuse_address = True
    server = socketserver.ThreadingTCPServer(('0.0.0.0', 1337), RequestHandler)
    server.serve_forever()
```

---

## Nmap

Let’s start with a nmap scan:

```
PORT     STATE SERVICE REASON         VERSION
22/tcp   open  ssh     syn-ack ttl 64 OpenSSH 8.2p1 Ubuntu 4ubuntu0.13 (Ubuntu Linux; protocol 2.0)
1337/tcp open  waste?  syn-ack ttl 64
```

---

## Introduction

This room runs a small service on port 1337. The provided source code shows it’s using a repeating-key XOR cipher with a short 5-character key to “encrypt” a known fake flag. Each connection generates a new random key, XORs the plaintext, and prints the result as a hex string. If you return the correct key, the server reveals the real flag.

---

## Enumeration

Connecting using netcat:

```bash
nc wise.thm 1337
```

Output:

```
This XOR encoded text has flag 1: 203d2823474514093643310d1119430041063354351b176b5618391c306206011c6842060d2a2a4a
What is the encryption key?
```

The ciphertext changes every connection, confirming the key is randomly generated each time.

---

## Understanding the Code

From the source, the important parts are:

```python
key = ''.join(random.choices(string.ascii_letters + string.digits, k=5))
flag = 'THM{thisisafakeflag}'

for i in range(len(flag)):
    xored += chr(ord(flag[i]) ^ ord(key[i % len(key)]))
```

* The plaintext is always `THM{thisisafakeflag}`
* A random 5-character key is generated
* Each plaintext character is XORed with one character of the key, looping with `%`
* The output is shown as hex

Because XOR is reversible and the plaintext is known, we can recover the key by computing:

```
key[i] = ciphertext[i] ⊕ plaintext[i]
```

for the first 5 characters.

---

## First Attempt: Brute Forcing

At first, brute forcing seemed like an option. The keyspace is 62^5 = 916 million possibilities, which is not feasible. Even filtering for flags starting with `THM{` would take too long. This showed brute force wasn’t the right way.

---

## Smarter Approach: Known Plaintext Attack

Since the plaintext is known, we can directly recover the key by XORing the ciphertext with the plaintext. That gives us the exact 5-character key for that session.

---

## Solver Script

I wrote a Python script to automate recovering the key and decrypting the fake message.

```python
# Recover 5-char repeating XOR key using known plaintext, then decrypt whole message.

import argparse

def recover_key(cipher_hex: str, known_plain: bytes, key_len: int = 5) -> str:
    # Decode hex → bytes
    c = bytes.fromhex(cipher_hex)
    # Sanity: need at least len(known_plain) bytes to recover the keystream
    if len(c) < len(known_plain):
        raise ValueError("Ciphertext too short to use known plaintext.")
    # Keystream = C ⊕ P over known plaintext span
    ks = bytes(ci ^ pi for ci, pi in zip(c, known_plain))
    # Repeating key = first key_len bytes of keystream
    return ks[:key_len].decode('ascii', errors='strict')

def xor_decrypt(cipher_hex: str, key: str) -> str:
    # Decode hex → bytes
    c = bytes.fromhex(cipher_hex)
    k = key.encode()
    # Repeating-key XOR over whole ciphertext
    p = bytes(ci ^ k[i % len(k)] for i, ci in enumerate(c))
    # Return printable text (replace errors to avoid crashes)
    return p.decode('utf-8', errors='replace')

def main():
    # CLI: hex input (required)
    ap = argparse.ArgumentParser(description="Recover XOR key and decrypt")
    ap.add_argument("hex_encoded", help="Hex-encoded ciphertext")
    ap.add_argument("--plain", default="THM{thisisafakeflag}",
                    help="Known plaintext prefix (default: THM{thisisafakeflag})")
    ap.add_argument("--keylen", type=int, default=5, help="Repeating key length (default: 5)")
    args = ap.parse_args()

    known_plain = args.plain.encode()
    key = recover_key(args.hex_encoded, known_plain, key_len=args.keylen)
    msg = xor_decrypt(args.hex_encoded, key)

    print("Recovered key:", key)
    print("Decrypted message:", msg)

if __name__ == "__main__":
    main()
```

---

## Run

Connecting to the service:

```
This XOR encoded text has flag 1: 203d2823474514093643310d1119430041063354351b176b5618391c306206011c6842060d2a2a4a
What is the encryption key?
```

Running the script with that ciphertext:

```bash
python3 script1.py 203d2823474514093643310d1119430041063354351b176b5618391c306206011c6842060d2a2a4a
```

Output:

```
Derived start of the key: tueX
Derived end of the key: 7
Derived key: tueX7
Decrypted message: THM{p1alntExtAtt4ckcAnr3alLyhUrty0urxOr}
```

Entering the key back into the netcat session:

```
What is the encryption key? tueX7
Congrats! That is the correct key! Here is flag 2: THM{BrUt3_ForC1nG_XOR_cAn_B3_FuN_nO?}
```

---

## Conclusion

The challenge demonstrates why repeating-key XOR is insecure when the plaintext is predictable. Instead of brute forcing, we use the known plaintext to directly recover the key and submit it to the service to reveal the real flag.

---

## Flags

> Q1: Flag 1
> A1: THM{p1alntExtAtt4ckcAnr3alLyhUrty0urxOr}

> Q2: Flag 2
> A2: THM{BrUt3\_ForC1nG\_XOR\_cAn\_B3\_FuN\_nO?}
