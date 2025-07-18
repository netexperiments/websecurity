# Server Side Request Forgery

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
