# Safety and Ethical Use

Hackergram is **intentionally vulnerable**. Everything on this site (the classical attacks, the LLM-driven attacks, the attack scripts) targets flaws that were put there deliberately, for teaching and research
purposes, inside the lab environments described on the [Lab Setup](labs/attacks/lab-setup.md) page.

!!! danger "Read this before running anything on this site"

    - **Hackergram is deliberately vulnerable.** Do not use its code, configuration, or dependency choices as
      a reference for how to build a real application.
    - **Experiments are authorized only against the supplied lab.** Every attack, script, and payload on this
      site is written to target *your own* Hackergram instance: the one you deployed locally, via Docker, or
      in GNS3.
    - **Never expose Hackergram to the public Internet.** It must only run in isolated, controlled
      environments (an isolated VM, a private Docker network, or a GNS3 topology with no route to the
      Internet). Do not port-forward it, do not deploy it to a public cloud instance, and do not put it behind
      a public domain name.
    - **Never reuse these attack scripts against systems you don't own or have explicit written authorization
      to test.** The scripts on this site assume the target *is* Hackergram; pointing them at anything else
      without authorization is unauthorized access, regardless of intent.
    - **Run destructive or denial-of-service experiments only in isolated environments.** Some experiments
      (the [XXE Billion Laughs attack](labs/attacks/xxe.md), the piggybacked `DROP TABLE` in [SQL
      Injection](labs/attacks/sqli.md)) can crash or corrupt the running instance. That's expected, and
      exactly why they must never run against a shared or Internet-facing deployment.
    - **`/reset` may destroy or replace the current lab state.** If you or a classmate have work in the
      current Hackergram instance that isn't part of this experiment, `/reset` will discard it. Coordinate
      before resetting a shared instance.

## Why this matters

This site exists to teach *how* these vulnerabilities work and *how* to fix them, in a purpose-built,
disposable, disconnected environment. The same techniques, applied to a system you don't own or without
authorization, are illegal in most jurisdictions and unethical regardless of jurisdiction. Treat every attack
on this site the way you'd treat a fire drill: valuable to practice, dangerous if run somewhere it doesn't
belong.

If you're ever unsure whether a target is "your own Hackergram instance," stop and check before proceeding.
