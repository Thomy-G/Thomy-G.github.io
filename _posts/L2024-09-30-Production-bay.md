---
title: Production-bay
date: 2024-09-30 14:09:14 +0300
categories:
  - Web exploitation
  - CTF Writeup
tags:
  - Nginx
media_subpath: /assets/img/
---
# Production-bay
Production-bay is a web challenge for Defcamp CTF 2024

We're presented with an instance and the challenge description
![Pasted image 20240930143242](Pasted image 20240930143242.png)
The site is a cute cat picture generator
![Pasted image 20240930155253](Pasted image 20240930155253.png)

Trying to go to the flag endpoint results in a 403
![Pasted image 20240930160542](Pasted image 20240930160542.png)

Looking at the source we can see the at the cat pictures are gotten from the /api/data/cat endpoint 
![Pasted image 20240930155140](Pasted image 20240930155140.png)

checking for other endpoints we can go to the /api/data and see a warning
![Pasted image 20240930155710](Pasted image 20240930155710.png)
debug is enabled.

Going to the debug endpoint we see another message
![Pasted image 20240930160156](Pasted image 20240930160156.png)
Now we see that there needs to be a host parameter probably with a url.

Giving the endpoint the url of a webhook we see that a request is made
![Pasted image 20240930160400](Pasted image 20240930160400.png)
Next logical step would be to try the /flag endpoint on localhost:5000

![Pasted image 20240930162814](Pasted image 20240930162814.png)
weirdly it says the host is :5000 and not the full localhost:5000, but looking back at the webhook we see that x-original-host is :5000
![Pasted image 20240930163003](Pasted image 20240930163003.png)
So maybe trying to change that should work.

Giving the request the x-original-host of localhost takes us to the finish line

![Pasted image 20240930163425](Pasted image 20240930163425.png)

ctf{89b52b00fd39c0410372b898632e6bf 0648ae9f43d500762d03af9e7768bcbfd}
