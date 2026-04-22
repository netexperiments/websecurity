# Hackergram Overview

Hackergram is a purpose-built vulnerable social networking application designed to give students and security practitioners a realistic environment for exploring web security flaws. It is used throughout this lab as the primary target for both classical web attacks and vulnerabilities introduced by LLM integration. The codebase is kept small enough that every component can be read and understood directly, while still covering a broad enough range of functionality — authentication, content management, messaging, search, and AI-assisted features — to make the experiments meaningful.

Because the source code is fully open, users are encouraged to go beyond running the provided attacks: inspect the vulnerable implementation, trace how untrusted input reaches an interpreter, apply the suggested countermeasures directly in the code, and verify that the fix works. This makes Hackergram a flexible platform for experimenting not only with exploits but also with the practical design and testing of defenses.

## Features

Users can create an account by registering a username, password, and name. Once logged in, the following features are available:

- **Posts** — Create, edit, and delete posts that are visible to all users on the homepage
- **Friends** — Send, accept, or decline friendship requests, and remove existing friendships
- **Profiles** — View any user's profile, including their name, username, picture, bio, posts, and friends
- **Settings** — Update your own name, password, picture, and bio
- **Search** — Search for posts by content or find other users by username
- **Messages** — Exchange direct messages with other users
- **AI Features** — AI-assisted post generation and post summarization powered by a locally hosted LLM (Ollama/Mistral)
- **Reset** — The `/reset` endpoint restores the application to its initial state, which is useful after attacks that break or corrupt it

## Default Users

The initial state of Hackergram includes the following pre-configured users. The user `mr_robot` has the most data associated with their account.

| User       | Password       |
|------------|----------------|
| admin      | 1_4m_Th3_4dm1n |
| mr_robot   | elliot123      |
| dpr        | silk-road      |
| satoshi    | bitcoin2009    |
| heisenberg | walter1958     |
| rick       | RickC-137      |
| stark      | winterfell     |
| anon1      | 1              |
| anon2      | 2              |
| anon3      | 3              |

## Covered Attacks

The following attacks are covered in this lab, organized by category:

**Parser-Driven Injections**

- [SQL Injection](attacks/sqli.md) — Error-based, union-based, boolean-based, piggybacked, and authentication bypass
- [NoSQL Injection](attacks/nosqli.md) — Syntax injection and operator injection
- [XXE Injection](attacks/xxe.md) — File retrieval and Billion Laughs denial of service

**Interpreter-Driven Injections**

- [Cross-Site Scripting (XSS)](attacks/xss.md) — Stored XSS, reflected XSS, and XSS worm
- [LLM-mediated SQL Injection](attacks/llm-mediated-sqli.md) — SQL injection via LLM-generated queries at the `/leaderboard` endpoint
- [LLM-mediated Stored XSS](attacks/llm-mediated-stored-xss.md) — Stored XSS via unsanitized LLM output at the `/ai_summarize` endpoint
- [Indirect Prompt Injection](attacks/indirect-prompt-injection.md) — Injecting hidden instructions through attacker-controlled external content
- [System Prompt Leakage](attacks/system-prompt-leakage.md) — Extracting internal LLM configuration through prompt manipulation

**Access & Resource Control**

- [Path Traversal](attacks/path-traversal.md) — Accessing files outside the intended directory via crafted paths

**Request & Interaction Forgery**

- [Cross-Site Request Forgery (CSRF)](attacks/csrf.md) — Forging authenticated requests through a malicious page
- [Clickjacking](attacks/clickjacking.md) — Tricking users into interacting with hidden UI elements
- [Server-Side Request Forgery (SSRF)](attacks/ssrf.md) — Forcing the server to issue requests to unintended internal or external targets
