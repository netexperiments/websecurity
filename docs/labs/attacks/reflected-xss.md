# Reflected XSS

In this attack, the attacker tricks the victim into clicking a malicious link which results in a JavaScript code being executed in the victim-browser, hence the name reflected XSS. The vulnerable endpoint that will be exploited is ```/users```, which allows searching for users by providing a search string. The string is passed to Hackergram as an argument (called ```search```) of a GET Request. The procedure is the following:

1.	Install a Wireshark probe at the Hackergram interface.
2.	In the victim-browser enter:
```http://192.168.0.100/users?search=<script>alert%28"XSS"%29<%2Fscript>```
Note that ```%28``` and ```%29``` encode the ```(``` and ```)``` characters, and ```%2F``` encode the ```/``` character.
3.	Check that a pop-up box with the message ```XSS``` is displayed.
4.	Analyze the traffic exchanged by Hackergram using an ```HTTP``` filter. Identify the ```HTTP``` message that sends the JavaScript code to Hackergram and the one where it is reflected to the victim-browser.

!!! note "Additional exercises"

    1. Hackergram has another endpoint vulnerable to this attack. Discover it and perform the attack.
    2. Using the bleach library, sanitize the ```/users``` endpoint so that the attack is no longer possible. 