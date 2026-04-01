# Path Traversal
A path traversal, also called a directory traversal attack, is a web vulnerability that allows attackers to
access files and directories that are outside the application’s restricted access area, such as the web root or other protected folders. This vulnerability surges because the website uses unsanitized input data
to reference files and directories on the backend. Attackers exploit file path manipulation sequences, such as ../ , (commonly used in Unix operating systems to access the parent directory) to traverse
directories and even read or write unauthorized files [37]. While many popular web servers are designed to prevent directory traversal, some encoding techniques can be used to bypass these protections. Examples of those encoding methods include Unicode, URL encoding, double URL encoding, and theuse of backslash characters on Windows servers.

## Attack
In this attack, the goal is to manipulate the file upload functionality in the Hackergram application to gain unauthorized access to internal files on the web server. This vulnerability is present in the `/settings` endpoint.

The application stores uploaded profile pictures on the server. However, because the server does not properly sanitize the filename parameter, it may be possible to craft a malicious filename containing directory traversal sequences. This can potentially allow access to or overwriting of unintended files on the system. 

To exploit this vulnerability:

1.	Intercept the HTTP request sent during the profile update using a tool such as Burp Suite.

2.	Modify the value of the filename field in the multipart form-data to include path traversal sequences. 

3.	Attemp to change the icon of the Hackergram app. 

    !!! tip
        Use your browser's inspection tool to identify the filename of the image stored on the web server.

4.	Submit the request, checking that the manipulated file will overwrite the target file on disk.

!!! note "Additional exercise"
    Try to change another user's profile picture.

## Countermeasure

The fundamental idea to prevent this type of attacks is to implement a secure file upload mechanism.

To achieve this, on the Hackergram container access the `app/` directory, go to the views.py file and modify the `/settings` endpoint to sanitize filenames using [`werkzeug.utils.secure_filename`](https://tedboy.github.io/flask/generated/werkzeug.secure_filename.html) (`secure_filename(new_photo.filename)`), which strips out dangerous characters and ensures only safe filenames are used. Also, enforce directory constraints by computing the absolute resolved path of the uploaded file using [`os.path.realpath()`](https://docs.python.org/3/library/os.path.html) and verifying it resides within the intended directory.
