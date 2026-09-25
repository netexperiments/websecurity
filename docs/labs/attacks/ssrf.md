# Server-Side Request Forgery

## Objective

This experiment demonstrates that Hackergram's server-side URL fetcher, used to accept a profile-picture URL,
retrieves *any* URL scheme the attacker supplies (including `file://`), letting an attacker read files off
the Hackergram host itself.

## Affected Hackergram functionality

- `/settings`: profile-picture URL field.

## Prerequisites

Simple/local deployment; a web browser is enough, no attacker HTTP server needed.

## Initial state

Log in as any user (or run `/reset` first for a clean starting point).

## Vulnerable implementation

`views.py`'s `/settings` handler (around line 141) fetches whatever URL is supplied in the profile-picture
field without restricting the scheme, host, or destination, so a `file://` URL is followed just as readily
as an `http://` one.

## Experiment

This attack targets the `/settings` endpoint. Unlike the request-forgery attacks above, the objective is to
manipulate the server itself into making an unintended request.

1. In the web browser, navigate to the `/settings` endpoint.
2. Insert the payload `file:///etc/passwd` inside the profile picture URL field.
3. Enter the right password and save the settings.
4. Once the settings have been saved, right click on your new profile picture and open it on another tab.
5. Download the image and inspect it with `cat <name_of_the_image>`, and check for content relative to the
   Hackergram web application.

!!! note "Additional exercise"
    Try to take advantage of this attack to find one other sensitive file related to the web application,
    such as the main Flask app file.

## Expected result

The "profile picture" the browser downloads is not an image. It's the raw contents of `/etc/passwd` (or
whichever local file was requested), served back through the profile-picture field.

## Why it works

The endpoint treats the profile-picture field as an arbitrary URL to fetch server-side, with no scheme
allowlist and no check on the resolved destination. `file://` is a valid URL scheme to most HTTP client
libraries, so the same code path that would legitimately fetch an image from a CDN just as happily reads a
local file and returns its bytes.

## Reset / cleanup

Run `/reset`, or manually reset your profile picture via `/settings`.

## Inspect and modify

Right-click the Hackergram machine and select the auxiliary console button. Inside the console, navigate to
the `hackergram/hackergram-lab/app` folder and open the `views.py` file (for example, run `vim views.py`).
Navigate to the `/settings` endpoint (starting around line 141) and identify the variable that introduces the
SSRF: the one passed straight into the URL-fetching call.

The safest fix is to remove server-side URL fetching altogether. If the feature must remain, harden it across
multiple layers:

1. Scheme allowlist: only `http`/`https`.
2. Domain allowlist: only fetch from explicitly approved hosts (e.g. your own CDN, or a small curated set).
3. Port allowlist: only `80` and `443`.
4. IP blocklist (default deny): reject loopback, private, link-local, multicast, unique-local IPv6, and
   known cloud-metadata IPs.

Fix the Hackergram application by implementing a function that enforces allowlist and blocklist checks for
this endpoint, and apply it to the currently vulnerable endpoint. Repeat the attack to confirm it no longer
takes effect.

## Exercise

Use this SSRF to find one other sensitive file related to the web application, for example the main Flask
app file.

## Hint

<details>
<summary>💡 Hint</summary>

The app's entry point is typically named after the project itself (`hackergram.py`), so try requesting it the
same way you requested `/etc/passwd`.

</details>
