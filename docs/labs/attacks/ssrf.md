# Server-Side Request Forgery
In modern web applications, it is common for web applications to accept complete URLs as user input.
In such cases, the web server initiates a HTTP request to the provided URL. However, if the server does not properly validate the destination, and the URL points to an internal service, the application may become vulnerable to a SSRF attack.
An SSRF attack happens when an attacker exploits server-side functionality to make arbitrary requests to unintended destinations. This vulnerability allows attackers to access services in the loopback interface, scan internal networks, read local files, abuse cloud service metadata endpoints and potentially move laterally into the internal environment.

This attack targets the `/settings` endpoint. Unlike the previous attack, the objective is to manipulate the server to make unintended requests.

## Attack

To perform the attack, do the following:

1.	In the web browser, navigate to the `/settings` endpoint
2. 	Insert the payload `file:///etc/passwd` inside the profile picture URL field.
3.	Enter the right password and save the settings.
4.	Once the settings have been saved, right click on your new profile picture and open it on another tab.
5.	Download the image and inspect it with cat <name_of_the_image> and check the sensitive content relative to the Hackergram web application.

!!! note "Additional exercise"
    Try to take advantage of this attack to find one other sensitive file related to the web application, such as the main Flask app file.

## Countermeasure
Right-click the Hackergram machine and select the auxiliary console button. Inside the console, navigate to the `hackergram/hackergram-lab/app` folder and open the `views.py` file. To view `views.py`, open it in a text editor (for example, run `vim views.py`). Navigate to the `/settings` endpoint (starting around line 141). Identify the part of the function that enables the SSRF vulnerability. Which variable introduces the issue here?
In this case, the safest fix is to remove server-side URL fetching altogether. If the feature must remain, harden it across multiple layers:

1. Scheme allowlist: only http/https.
2. Domain allowlist: only fetch from explicitly approved hosts (e.g., your own CDN or a small curated set).
3. Port allowlist: only 80 and 443.
4. IP blocklist (default deny): reject loopback, private, link-local, multicast, unique-local IPv6, and known cloud metadata IPs.

Fix the Hackergram application by implementing a function that enforces allowlist and blocklist checks for this endpoint, and apply it to the currently vulnerable endpoint.

Repeat the attack to confirm that it no longer takes effect.