# OhSINT – TryHackMe Walkthrough  
**Room:** https://tryhackme.com/room/ohsint

## Introduction  
Hello, my name is Shabha, and today I’ll be walking you through the TryHackMe room **OhSINT**. This challenge serves as a fantastic introduction to Open Source Intelligence (OSINT) and highlights how publicly available information can be used to uncover a surprising amount of personal data. Let's dive in.

After launching the room and downloading the image file `WindowsXP.jpg`, my initial inspection showed nothing out of the ordinary. It looked like a standard image. Curious to dig deeper, I opened the HTML source code related to the file but didn’t find anything significant there.

![image1](https://github.com/user-attachments/assets/8ff014a5-920d-4690-a164-91dbeef433a9)

To uncover any hidden metadata, I turned to a trusty tool: exiftool. Running the command exiftool WindowsXP.jpg revealed a treasure trove of information, including GPS coordinates and a copyright entry labeled “OWoodflint.” This was my first major clue.

ExifTool Output – Showing GPS Coordinates and Author Info
---------------------------------------------------------

ExifTool Version Number:
11.88

File Name:
WindowsXP.jpg

Directory:
.

File Size:
229 kB

File Modification Date/Time:
2022:09:02 08:48:02+01:00

File Access Date/Time:
2025:05:12 02:07:01+01:00

File Inode Change Date/Time:
2025:05:12 02:07:01+01:00

File Permissions:
rw-rw-r--

File Type:
JPEG

File Type Extension:
jpg

MIME Type:
image/jpeg

XMP Toolkit:
Image::ExifTool 11.27

GPS Latitude:
54 deg 17' 41.27" N

GPS Longitude:
2 deg 15' 1.33" W

Copyright:
OWoodflint

Image Width:
1920

Image Height:
1080

Encoding Process:
Baseline DCT, Huffman coding

Bits Per Sample:
8

Color Components:
3

Y Cb Cr Sub Sampling:
YCbCr4:2:0 (2 2)

Image Size:
1920x1080

Megapixels:
2.1

GPS Latitude Ref:
North

GPS Longitude Ref:
West

GPS Position:
54 deg 17' 41.27" N, 2 deg 15' 1.33" W

---------------------------------------------------------


The GPS coordinates pointed to a specific location, and the name “OWoodflint” seemed like a unique identifier—possibly a username.

Naturally, I searched for “OWoodflint” online. It didn’t take long to find a Twitter account with that username and a profile picture of a cat. This answered the first question in the room.

![Screenshot From 2025-05-11 21-17-18](https://github.com/user-attachments/assets/7fbb251e-52bb-4fde-82d4-8de5455aea90)

Screenshot of OWoodflint's Twitter Profile  
>Q1: What is this user's avatar of?  
>A1: Cat

Continuing the investigation, I discovered a GitHub repository under the username OWoodfl1nt. One of the repositories, titled people_finder, contained enough personal data to identify the user's location as London.

![Screenshot From 2025-05-11 21-20-19](https://github.com/user-attachments/assets/7f645ac6-d47f-4a34-ad89-c8f6280449d2)

GitHub profile showing city  
>Q2: What city is this person in?  
>A2: London

Going back to the image metadata, I noticed a BSSID (MAC address of a wireless access point). Using this BSSID—B4:5D:50:AA:86:41—I visited wigle.net, a site that lets users search wireless network information. By entering the BSSID into the search form, I was able to pinpoint the SSID (network name) the user had connected to.

Wigle.net results with SSID  
>Q3: What is the SSID of the WAP he connected to?  
>A3: UnileverWiFi

Returning to the GitHub profile, I noticed the user had publicly listed their email address. This answered two more questions with one clue.

>Q4: What is his personal email address?  
>A4: OWoodflint@gmail.com  
>Q5: What site did you find his email address on?  
>A5: GitHub

The GitHub page also linked to the user’s personal WordPress blog. Exploring the blog, I came across a post where the user mentioned planning a trip to New York. That gave us the answer to another room question.

![Screenshot From 2025-05-11 21-42-15](https://github.com/user-attachments/assets/c637a031-49da-4726-945f-583820f17e42)

WordPress blog post about travel  
>Q6: Where has he gone on holiday?  
>A6: New York

Finally, I inspected the source code of the WordPress blog. Tucked away in the code was a string that appeared to be a password—likely our final flag.

![Untitled design](https://github.com/user-attachments/assets/c8e3fda6-5c23-4bdf-818a-19580c2b9471)

Source code snippet from the WordPress page  
>Q7: What is the person’s password?  
>A7: pennYDr0pper.!

Conclusion  
The OhSINT room was honestly a blast. It felt more like solving a digital mystery than just answering questions. I got to dig into image metadata, hunt people down across platforms like Twitter and GitHub, trace Wi-Fi networks, and even snoop through blog source code for hidden info. Pretty wild how much you can find with just a single picture.

This challenge really opened my eyes to the power of OSINT—and how careful people need to be with what they put out online. I learned a lot, had fun, and definitely feel more confident using tools like exiftool and sites like Wigle.net.

Thanks for checking out my write-up! More to come soon 🙌
