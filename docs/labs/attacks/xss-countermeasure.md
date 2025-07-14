# XSS Countermeasure
Effective prevention of XSS attacks involves a multi-layer defense
strategy. In particular, it is essential to combine input sanitization, output encoding and the enforcement of a [Content-Security-Policy](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/CSP).

## Stored XSS Mitigation

For mitigating stored XSS, the use of html-sanitizer, a maintained allowlist-based HTML sanitizing tool, is recommended. This library should be applied to all user-supplied input prior to rendering or storage. For example, the original code handling the username field in the settings page:

```python
new_name = request.form['name']
```

Should be replaced with the sanitized version:

```python
cleaned_new_name = bleach.clean(request.form['name'])
```

## Reflected XSS Mitigation

To mitigate reflected XSS, all Jinja2 templates must avoid disabling automatic HTML escaping. This can be achieved by removing directives such as `{% autoescape false %}` and `{% endautoescape %}`, which would otherwise allow raw user input to be rendered unescaped.

## Content Security Policy Implementation

In addition to sanitization and encoding, the implementation of a strict CSP is advised to further reduce the risk of script execution in the event of injection. Specifically, inline JavaScript should be disallowed via the following directive, which can be added to the base.html file:

```html
<meta http-equiv="Content-Security-Policy" content="default-src 'self'; script-src 'none'; object-src 'none';">
```