# Extending Hackergram

Hackergram is designed to be extended, not just attacked. This page explains how to add new functionality (including new vulnerabilities or defenses) and how to turn that work into a new lab page on this site.

## Ways to extend Hackergram

- **Add a Flask route.** New routes go in `views.py`, following the pattern used by existing endpoints
  (read `request.form`/`request.args`/`request.json`, call into `models.py`, render a template).
- **Add a new vulnerable endpoint.** Reuse one of the existing failure patterns documented on this site
  (string-formatted SQL, unsanitized MongoDB queries, unsanitized HTML rendering, unrestricted server-side
  fetch, missing CSRF token, missing frame-ancestors policy) so the new endpoint is a genuine, reproducible
  example of that vulnerability class rather than something bespoke and hard to explain.
- **Add a new attack script.** Follow the shared structure used throughout this site's exploit scripts:
  `reset()` → `register()` → `login()` → `exploit()`, parameterized by `host`/`port` from the command line. See any script on the [SQL Injection](labs/attacks/sqli.md) page for the template.
- **Add a new Jinja2/browser-side scenario.** For attacks that need client-side JavaScript (like the [XSS
  Worm](labs/attacks/xss.md)), keep the payload self-contained and reconstructible from the DOM, so it can be
  demonstrated without a separate hosted file.
- **Add another database interaction.** Decide up front whether it belongs in MySQL (structured, relational
  data) or MongoDB (the pattern already used for chat/message history); see [Architecture](architecture.md).
- **Add another external-resource interaction.** If the new feature fetches a URL server-side, treat it as a
  potential SSRF/indirect-prompt-injection surface from the start, and document which allowlist/blocklist
  checks it does or deliberately omits.
- **Add an LLM-assisted workflow.** Route it through Ollama the same way `/leaderboard`, `/ai_summarize`, and
  `/generate_post` do, and decide deliberately whether its output reaches an interpreter (SQL, HTML). That
  decision is what determines whether the new feature is vulnerable by design or safe by design.
- **Implement a countermeasure.** Every attack page's "Inspect and modify" section is a self-contained patch: apply it directly to test that the fix actually closes the exploit before writing it up.
- **Add a new lab page to the companion website.** Use the [standardized attack-page
  template](#standardized-attack-page-template) below.

## Worked example: duplicate a route, introduce a vulnerability, register the experiment

1. **Duplicate an existing route.** Copy `/create_post`'s handler in `views.py` to a new route, e.g.
   `/create_announcement`, that stores a short broadcast message instead of a post.
2. **Introduce a vulnerability or a defense.** For a vulnerability: store the announcement text with the same
   unsanitized string handling `/create_post` currently has, so it's Stored-XSS-vulnerable in the same way.
   For a defense: apply `bleach.clean()` to the input before storing it, and write the announcement page so a
   `<script>` payload visibly fails to execute.
3. **Register a new experiment page.** Add `docs/labs/attacks/announcement-xss.md` (or `-defense.md`) using
   the template below, and add it to `mkdocs.yml`'s `nav:` under the appropriate category.

## Standardized attack-page template

Every experiment page on this site uses these headings, in this order. See any page under [Web Security
Experiments](labs/hackergram.md#covered-attacks) for a filled-in example:

```markdown
# <Experiment name>

## Objective
## Affected Hackergram functionality
## Prerequisites
## Initial state
## Vulnerable implementation
## Experiment
## Expected result
## Why it works
## Reset / cleanup
## Inspect and modify
## Exercise
## Hint
```

## Contribution template

A contributor adding a new lab page should provide the following before it's merged:

```markdown
- Experiment name:
- Vulnerability type:
- Affected endpoint(s):
- Source-code location (file/function):
- Prerequisites (deployment level, tools):
- Reset requirement (does /reset need to run first?):
- Attack steps:
- Expected result:
- Cleanup:
- Optional script (attach or link):
- Exercise:
- Hint (keep collapsible):
- Defense/modification task:
```

This keeps every new lab page structurally consistent with the rest of the site, and gives a reviewer enough
information to actually reproduce the experiment before merging it.
