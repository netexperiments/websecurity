# LLM Setup

Hackergram's LLM-driven experiments ([LLM-mediated SQL Injection](labs/attacks/llm-mediated-sqli.md),
[LLM-mediated Stored XSS](labs/attacks/llm-mediated-stored-xss.md), [Indirect Prompt
Injection](labs/attacks/indirect-prompt-injection.md), [System Prompt
Leakage](labs/attacks/system-prompt-leakage.md)) depend on a locally hosted [Ollama](https://ollama.com/)
instance. "Install Ollama" alone isn't enough to reproduce these experiments. The exact model matters,
since different models will follow (or refuse) the same extraction/injection prompts differently.

## Models

Hackergram uses two models, both served by [Ollama](https://ollama.com/). Each LLM-facing endpoint calls a
specific one, so **you need both models pulled** to run every LLM exercise:

| Endpoint | Model (Ollama tag) | Inference options | Exercises |
|---|---|---|---|
| `/leaderboard` | `llama2` (Llama 2, 7B) | temperature `0.1`, seed `42` | [LLM-mediated SQL Injection](labs/attacks/llm-mediated-sqli.md) |
| `/generate_post` | `mistral` | Ollama defaults | [Indirect Prompt Injection](labs/attacks/indirect-prompt-injection.md), [System Prompt Leakage](labs/attacks/system-prompt-leakage.md) |
| `/ai_summarize` | `mistral` | temperature `0.4`, seed `123` | [LLM-mediated Stored XSS](labs/attacks/llm-mediated-stored-xss.md) |

Llama 2 at 7B parameters was chosen to balance functionality against the resource constraints of a lab
running on ordinary user hardware. Mistral was also the model used for the temperature/seed benchmark
summarized at the bottom of this page.

To obtain both models:

```bash
ollama pull llama2
ollama pull mistral
```

Both tags are Ollama's defaults (`latest`), which serve the 7B variant of each model.

## Configuration

- **API endpoint used by Hackergram:** `http://localhost:11434/api/generate` (Ollama's default port,
  `11434`), set as `OLLAMA_API_URL` near the top of
  [`views.py`](https://github.com/netexperiments/hackergramlab/blob/main/views.py){:target="_blank"}. The
  backend sends a `POST` request with a JSON payload containing the model name and prompt, and reads the
  generated text from the response. Some of the *fixed* code samples on the attack pages use `/api/chat`
  instead, with explicit `system`/`user` role separation, as part of the proposed countermeasure. That's
  the target state, not what runs by default.
- **How the model is selected:** there is no configuration file or environment variable. Each endpoint
  names its model directly in `views.py` (`query_llm(..., model="llama2")` for `/leaderboard`, and
  `"model": "mistral"` in the payloads of `/generate_post` and `/ai_summarize`). To change a model, edit
  those strings.
- **Required ports:** `11434` (Ollama API). In the Docker image, `container_start.sh` runs `ollama serve`
  inside the same container as Hackergram, so `localhost:11434` resolves without extra networking.

## Resource requirements

Ollama's default 4-bit quantization typically needs around 4–8 GB of RAM and roughly 4 GB of disk per model, and runs on CPU.

## Verifying Ollama is reachable from Hackergram

```bash
curl http://localhost:11434/api/tags
```

This should list both `llama2` and `mistral`. If Hackergram itself
can't reach Ollama, see the troubleshooting entry on [Lab Setup](labs/attacks/lab-setup.md).

## Changing the model

Swapping a model means editing the model string in `views.py` (see Configuration above). Running the
exercises with other models hasn't been tested. Different models follow or refuse the same injection
prompts differently, so results may not match this site's expected outcomes. If you try a different model,
please report what you observe.

## LLM output is probabilistic

Every LLM experiment on this site can behave differently across runs, model versions, and even inference
parameters. This isn't hand-waving: a controlled evaluation with Mistral, varying the Ollama `temperature`
parameter (0.0 to 1.5, 200 randomized seeds per value), found the model's system-prompt protection rate dropped from 100% at
`temperature=0.0` to 88% at `temperature=1.5`. Lower temperature makes the model more deterministic and
more likely to keep following its safety instructions, while higher temperature makes it more variable in both
directions. If an attack in this lab doesn't reproduce on the first try, try again, and consider whether the
configured temperature affects the outcome before concluding the vulnerability is fixed.
