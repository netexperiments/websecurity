# XXE Injection

XML External Entity (XXE) Injection happens when an application parses XML input with insecure settings that allow external entities.

In vulnerable systems, attackers can abuse this behavior to:

- Read local files from the server
- Trigger server-side requests (SSRF-like behavior)
- Cause denial-of-service conditions

## Why it belongs to parser-driven injections

XXE exploits weaknesses in how an XML parser processes structured input.
The parser itself becomes the attack surface, which is why this attack is categorized under parser-driven injections.

