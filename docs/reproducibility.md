# Reproducibility

This page lists every artifact associated with the Hackergram publication, and the environment it was
tested in, so the experiments on this site can be reproduced even after Hackergram continues to evolve.

!!! note "Documentation version: v1.0"
    This documentation corresponds to Hackergram v1.0, the version associated with the Computer Networks
    software article. The `v1.0` release tag has not been created yet; until it is, the source code is pinned
    to a specific commit below.

## Artifacts

| Artifact | Location / identifier |
|---|---|
| Source code | [netexperiments/hackergramlab](https://github.com/netexperiments/hackergramlab){:target="_blank"}, commit [`b723a88`](https://github.com/netexperiments/hackergramlab/commit/b723a88b7930ff9b848ba6d3a2883e5874f7587f){:target="_blank"} (2026-04-01) |
| Software version / tag | v1.0 (_tag to be created in `hackergramlab` before submission_) |
| Docker images | See [Docker images](#docker-images) below (tag and exact content digest for each) |
| Full / LLM configuration | The full image (`pimz23/hackergram30`) bundles Ollama and both models; see [LLM configuration](#llm-configuration) |
| Attack scripts | Embedded in each attack page of this site (e.g. [SQL Injection](labs/attacks/sqli.md)) |
| GNS3 topology | Topology figure, node names and addressing on the [Lab Setup](labs/attacks/lab-setup.md#gns3-deployment) page |
| GNS3 automation | `HackergramLabTopology.py`, on the [Lab Setup](labs/attacks/lab-setup.md#gns3-deployment) page |
| Initial dataset | [`start.sql`](https://github.com/netexperiments/hackergramlab/blob/main/start.sql){:target="_blank"}: MySQL schema and seed data (default users, posts, friendships) |
| Reset mechanism | The `/reset` route in [`views.py`](https://github.com/netexperiments/hackergramlab/blob/main/views.py){:target="_blank"}, which calls `models.reset()` to restore the initial state |
| Companion website | [netexperiments.github.io/websecurity](https://netexperiments.github.io/websecurity/){:target="_blank"}, source in [netexperiments/websecurity](https://github.com/netexperiments/websecurity){:target="_blank"} |

### Docker images

Tags such as `latest` can be moved to a new build at any time. The content digest identifies the exact image
used; pull by digest (e.g. `docker pull pimz23/hackergram30@sha256:b15dc8…`) to get a byte-identical copy.

| Image | Tag | Built | Digest |
|---|---|---|---|
| [`pimz23/hackergram-simple`](https://hub.docker.com/r/pimz23/hackergram-simple){:target="_blank"} | `latest` | 2026-04-03 | `sha256:822ff3f7259ebc2f8d810036eb70ac19c05b0eb31f7fe3d7aebff6627e86d093` |
| [`pimz23/hackergram30`](https://hub.docker.com/r/pimz23/hackergram30){:target="_blank"} | `latest` | 2025-08-30 | `sha256:b15dc8be87c6da4000e0bb0e69ebbf3b117b1bbdebf847eda2a5feb47e9d56dd` |
| [`pimz23/my-gns3-attacker`](https://hub.docker.com/r/pimz23/my-gns3-attacker){:target="_blank"} | `1.5` | 2024-10-09 | `sha256:390b9750102f4ccecddc0a4aab6fca99a410c8f0da8f9fc462b2e6ec62d90863` |
| [`pimz23/zap-desktop-novnc`](https://hub.docker.com/r/pimz23/zap-desktop-novnc){:target="_blank"} | `latest` | 2025-09-01 | `sha256:f15db4f8c4a2d57b9aaa0139960b99d2f4b557d72b864d2d19e267043c55091e` |
| [`gns3/webterm`](https://hub.docker.com/r/gns3/webterm){:target="_blank"} | `latest` | unknown | `sha256:041bf51a02603e229eeeb0019f5141587048417188b44f45708b968ab61cca09` |

The two Hackergram images are built on Ubuntu 20.04 (`linux/amd64`).

### LLM configuration

Served by Ollama 0.17.7 through `POST http://localhost:11434/api/generate`. The model for each endpoint is
set directly in `views.py`:

| Endpoint | Model | Inference options |
|---|---|---|
| `/leaderboard` | `llama2` (Llama 2, 7B) | temperature `0.1`, seed `42` |
| `/generate_post` | `mistral` (Mistral, 7B) | Ollama defaults |
| `/ai_summarize` | `mistral` (Mistral, 7B) | temperature `0.4`, seed `123` |

See [LLM Setup](llm-setup.md) for installation details.

## Tested environment

| Component | Version used / tested |
|---|---|
| Hackergram | v1.0 (code at `hackergramlab` commit `b723a88`) |
| Operating system | Ubuntu 24.04.4 LTS (WSL 2 on Windows 11) |
| Python | 3.12.3 |
| Flask | 3.1.3 (Werkzeug 3.1.6, Jinja2 3.1.6) |
| MySQL | 8.0.46 |
| MongoDB | 8.0.19 |
| Docker | Docker Desktop 4.63.0 (Engine 29.2.1) |
| GNS3 | 2.2 |
| Ollama | 0.17.7 |
| LLM models | `llama2` (7B) and `mistral` (7B), Ollama default tags |

!!! note "Test environment"
    These versions come from the WSL test installation. The distributed Docker images are built on
    Ubuntu 20.04, so package versions inside the containers may differ.

## Reproduce the paper examples

_These links will be finalized once the paper's section numbers are stable._

| Paper section | Experiment |
|---|---|
| _Section TBD_: XSS worm | [Run this experiment](labs/attacks/xss.md#xss-worm) |
| _Section TBD_: LLM-mediated SQL injection | [Run this experiment](labs/attacks/llm-mediated-sqli.md) |
| _Section TBD_: GNS3 deployment | [Build this topology](labs/attacks/lab-setup.md#gns3-deployment) |
