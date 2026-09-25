# Hackergram — An Open-Source Platform for Classical and LLM-Driven Web Security Experimentation

Hackergram is a deliberately vulnerable social-networking application, released fully open source so every part of it can be read and inspected. It is built for both classical web-security experiments (SQL injection, XSS, CSRF, and more) and attacks that involve LLM integration, such as prompt injection and LLM-mediated SQL injection. It can be deployed locally, packaged as Docker containers, or run inside a full GNS3 network topology for reproducible, isolated experimentation.

**Current stable release:** Hackergram v1.0, the version described in the accompanying paper.

!!! warning "Intentionally vulnerable"
    Hackergram is intentionally vulnerable and must only be deployed in isolated, controlled environments. It must never be exposed as a production service. Read the [Safety and Ethical Use](safety.md) page before running any experiment.

## What is Hackergram?

Hackergram is the target application used throughout this lab: a small, fully open-source social network with authentication, posts, friends, messaging, search, and LLM-assisted features, each shipped with a corresponding set of vulnerabilities. Because the codebase stays small enough to read end to end, it doubles as a platform for studying both offense (running the provided attacks) and defense (patching the vulnerable code and verifying the fix). Read more on the [Hackergram Overview](labs/hackergram.md) page, or see the [Architecture](architecture.md) page for how its components fit together.

## Labs

Hackergram currently has two versions:

**Hackergram-simple** -  [A version without the LLM related attacks](https://hub.docker.com/r/pimz23/hackergram-simple){:target="_blank"}

**Hackergram** - [The full version](https://hub.docker.com/r/pimz23/hackergram30){:target="_blank"}

This guide provides practical exercises for various attack scenarios. To start, open [Laboratory Setup](labs/attacks/lab-setup.md) to learn how to set up the laboratory. Enjoy!

## Citation

If you use Hackergram in your research, please cite it
([doi:10.5281/zenodo.22966304](https://doi.org/10.5281/zenodo.22966304){:target="_blank"}):

```bibtex
@software{hackergram2026,
  author    = {Pimentel, João and Valadas, Rui and Domingues, Tiago},
  title     = {Hackergram: An Open-Source Platform for Classical and LLM-Driven Web Security Experimentation},
  year      = {2026},
  version   = {1.0},
  publisher = {Zenodo},
  doi       = {10.5281/zenodo.22966304},
  url       = {https://doi.org/10.5281/zenodo.22966304}
}
```

A citation for the accompanying paper will be added here once it is published.

## Contributors

* João Pimentel, [Instituto Superior Técnico, Universidade de Lisboa](https://tecnico.ulisboa.pt/pt/){:target="_blank"}
* Rui Valadas, [Instituto Superior Técnico, Universidade de Lisboa](https://tecnico.ulisboa.pt/pt/){:target="_blank"}
* Tiago Domingues, [LastPass](https://www.lastpass.com/){:target="_blank"}
