# Path Traversal

## Objective

This experiment demonstrates that Hackergram's profile-picture upload accepts filenames containing directory
traversal sequences, letting an attacker overwrite arbitrary files on the server (including files belonging to other users, or the application's own static assets) from an ordinary upload form.

## Affected Hackergram functionality

- `/settings`: profile-picture upload (`filename` parameter of the multipart form-data request).

## Prerequisites

Simple/local deployment, plus an intercepting proxy such as Burp Suite to edit the raw multipart request.

## Initial state

Log in as any user (or run `/reset` first for a clean starting point).

## Vulnerable implementation

`views.py`'s `/settings` handler stores the uploaded profile picture using the client-supplied filename
without sanitizing it (no `secure_filename()` call, no check that the resolved path stays inside the intended
upload directory), so `../` sequences in the filename are honored as-is.

## Experiment

In this attack, the goal is to manipulate the file upload functionality in the Hackergram application to gain
unauthorized access to internal files on the web server. This vulnerability is present in the `/settings`
endpoint.

1. Intercept the HTTP request sent during the profile update using a tool such as Burp Suite.
2. Modify the value of the filename field in the multipart form-data to include path traversal sequences.
3. Attempt to change the icon of the Hackergram app.

    !!! tip
        Use your browser's inspection tool to identify the filename of the image stored on the web server.

4. Submit the request, checking that the manipulated file overwrites the target file on disk.

!!! note "Additional exercise"
    Try to change another user's profile picture.

## Expected result

The Hackergram app icon (or whichever file was targeted) changes to the uploaded image, confirming the write
landed outside the intended per-user upload directory.

## Why it works

The upload handler trusts the client-supplied filename as a literal path component. A filename such as
`../../static/favicon.jpg` resolves, once joined with the server's upload directory, to a path outside that
directory entirely (the classic directory-traversal pattern), and the server writes to it without checking.

## Reset / cleanup

Run `/reset` to restore the original static assets and profile pictures.

## Inspect and modify

On the Hackergram container, access the `app/` directory, go to the `views.py` file and modify the
`/settings` endpoint to sanitize filenames using
[`werkzeug.utils.secure_filename`](https://tedboy.github.io/flask/generated/werkzeug.secure_filename.html)
(`secure_filename(new_photo.filename)`), which strips out dangerous characters and ensures only safe
filenames are used. Also enforce directory constraints by computing the absolute resolved path of the
uploaded file using [`os.path.realpath()`](https://docs.python.org/3/library/os.path.html) and verifying it
resides within the intended directory. Repeat the attack afterward and confirm the traversal sequence is
neutralized.

## Exercise

Use the same technique to change another user's profile picture, without having their password.

## Hint

<details>
<summary>💡 Hint</summary>

Upload paths are usually keyed by username or user ID. If you can guess or discover another user's filename
convention, a traversal payload that targets their file (rather than a static asset) will overwrite their
picture instead of yours.

</details>
