# Hackergram Lab — Companion Website

This repository contains the source of the **Hackergram Lab companion website**, published at
**<https://netexperiments.github.io/websecurity/>**.

Hackergram is a deliberately vulnerable, open-source social-networking application for classical and
LLM-driven web security experimentation. The application itself lives in a separate repository,
[netexperiments/hackergramlab](https://github.com/netexperiments/hackergramlab). This repository holds only
the documentation: setup guides, experiment pages, and the material that supports the accompanying paper.

> **Warning:** Hackergram is intentionally vulnerable. Deploy it only in isolated, controlled environments
> and never expose it to the Internet. See the site's
> [Safety and Ethical Use](https://netexperiments.github.io/websecurity/safety/) page.

## What's on the website

| Section | Contents |
|---|---|
| Hackergram Overview | Features, default users, and the list of covered experiments |
| Architecture | Components, data flow, and the source-code layout of `hackergramlab` |
| Laboratory Setup | Quick Start, deployment options (Simple, Full, GNS3), GNS3 topology and automation, LLM setup |
| Web Security Experiments | One page per experiment, each with the same structure (objective, prerequisites, experiment, expected result, reset, inspect and modify, exercise, hint) |
| Reproducibility | Artifacts, Docker image digests, LLM configuration, and the tested environment |
| Extending Hackergram | How to add routes, experiments and countermeasures, plus a contribution template |
| Safety and Ethical Use | Rules for running the lab responsibly |
| Software Release, License and Citation | Version, authors and how to cite Hackergram |

## Repository layout

| Path | Purpose |
|---|---|
| `docs/` | Markdown source of every page |
| `docs/labs/attacks/` | Lab setup and experiment pages |
| `docs/assets/` | Images (favicon, architecture figure) |
| `docs/css/` | Custom styles |
| `mkdocs.yml` | Site configuration and navigation menu |
| `.github/workflows/ci.yml` | Builds and publishes the site to GitHub Pages |

## Building the site locally

The site is built with [MkDocs](https://www.mkdocs.org/) and the
[Material for MkDocs](https://squidfunk.github.io/mkdocs-material/) theme (tested with MkDocs 1.6.1 and
Material 9.7.7).

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install mkdocs mkdocs-material
mkdocs serve
```

Then open <http://127.0.0.1:8000/websecurity/>. Pages reload automatically when you save a file under
`docs/`.

To check for broken links before publishing:

```bash
mkdocs build --strict
```

## Editing the site

- **Change a page:** edit its `.md` file under `docs/`.
- **Add a page:** create the `.md` file under `docs/` and add it to the `nav:` section of `mkdocs.yml`.
- **Add an experiment:** follow the page structure and contribution template described on the site's
  [Extending Hackergram](https://netexperiments.github.io/websecurity/extending/) page.

## Publishing

Every push to `main` triggers the GitHub Actions workflow in `.github/workflows/ci.yml`, which runs
`mkdocs gh-deploy` and publishes the site to GitHub Pages. There is no separate manual deploy step.

## Authors

- João Pimentel, Instituto Superior Técnico, Universidade de Lisboa
- Rui Valadas, Instituto Superior Técnico, Universidade de Lisboa
- Tiago Domingues, LastPass

## Citation

If you use Hackergram in your research, please cite it ([doi:10.5281/zenodo.22966304](https://doi.org/10.5281/zenodo.22966304)):

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

## Disclaimer

Hackergram and this lab are intended for educational and research purposes only. Use the experiments only
against your own Hackergram instance.
