# Quick Start

This page takes you from nothing to a working Hackergram instance on your own machine, using Docker. You
don't need to read any of the experiment pages first.

!!! warning "Intentionally vulnerable"
    Hackergram is deliberately vulnerable. Run it only on your own machine or an isolated network, and never
    expose it to the Internet. See [Safety and Ethical Use](safety.md).

## 1. Prerequisites

- **Docker** (see the version below).
- A **web browser**.
- Disk space: about 1 GB for the simple version, or about 20 GB for the full version (its image is
  roughly 8.8 GB compressed and includes the LLM models).

## 2. Tested operating system

Windows 11 with WSL 2 (Ubuntu 24.04.4 LTS).

## 3. Docker

Tested with **Docker Desktop 4.63.0 (Docker Engine 29.2.1)**. Check your installation with:

```bash
docker --version
```

## 4. Python

Not required. Everything Hackergram needs (Python, Flask, MySQL, MongoDB and, in the full version, Ollama)
runs inside the container.

## 5. Choose a version and pull the image

| Version | Image | Includes |
|---|---|---|
| Simple | `pimz23/hackergram-simple:latest` | Web application, MySQL, MongoDB |
| Full | `pimz23/hackergram30:latest` | Everything in Simple, plus Ollama with the Llama 2 and Mistral models |

Start with the simple version if you're new to the lab:

```bash
docker pull pimz23/hackergram-simple:latest
```

Or the full version:

```bash
docker pull pimz23/hackergram30:latest
```

## 6. Launch Hackergram

Simple version:

```bash
docker run -d --name hackergram -p 80:80 pimz23/hackergram-simple:latest
```

Full version:

```bash
docker run -d --name hackergram -p 80:80 pimz23/hackergram30:latest
```

The container starts MySQL and MongoDB, loads the initial data, and then starts the web application. Give it
a few seconds before opening the browser (the full version takes a little longer, because it also starts
Ollama).

## 7. Ports

| Port | Used by |
|---|---|
| `80` | The Hackergram web application (published to your machine by `-p 80:80`) |
| `11434` | Ollama (full version only). It runs inside the container, so there's no need to publish it |

If port 80 is already in use on your machine, publish a different one, for example `-p 8080:80`, and use
`http://localhost:8080/` in the next step.

## 8. Open Hackergram

Go to:

```
http://localhost/
```

You should see the Hackergram login page.

## 9. Log in

Hackergram starts with a set of pre-configured users. To begin, log in as:

| Username | Password |
|---|---|
| `mr_robot` | `elliot123` |

`mr_robot` is the account with the most data. The full list of default users is on the [Hackergram
Overview](labs/hackergram.md#default-users) page.

## 10. Verify that Hackergram is working

After logging in you should land on the homepage feed, with posts from the default users. You can also check
the container from a terminal:

```bash
docker ps --filter name=hackergram
```

The `STATUS` column should say `Up`. To see the startup output:

```bash
docker logs hackergram
```

## 11. Reset Hackergram

To restore Hackergram to its initial state at any time (default users, posts and friendships), open:

```
http://localhost/reset
```

You'll be redirected to the homepage with the message "Hackergram was reset". Resetting discards every
change made since the container started.

## 12. Stop and remove

Stop the container:

```bash
docker stop hackergram
```

Start it again later:

```bash
docker start hackergram
```

Remove the container completely:

```bash
docker rm -f hackergram
```

Remove the downloaded image too (frees the disk space):

```bash
docker rmi pimz23/hackergram-simple:latest
```

(or `pimz23/hackergram30:latest` for the full version).

---

**[Next: Run your first experiment →](labs/attacks/clickjacking.md)**
