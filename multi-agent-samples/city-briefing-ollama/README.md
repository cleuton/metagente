# Demo: Metagente with Ollama (local model)

🇧🇷 [Leia em Português](README.pt-BR.md)

This demo runs the same Metagente agent twice: first with a model running on your own machine through Ollama, then with Claude in the cloud. The agent file is identical in both runs. Only `metagente.toml` changes.

It uses only features that already exist in version 0.1.4. No change to the Metagente code is needed.

The demo is built around the `researcher.ag` agent, which reads a Wikipedia page with an MCP tool (`fetch`) and summarizes it. This is a good test, because it needs **tool calling**, and that is exactly where small local models struggle. There is a section about it below, with a prompt written for small models.

## What you will see

1. The `Researcher` agent using a local model to read a web page through an MCP tool and write three sentences about a city.

2. The same agent, with no change to the `.ag` file, running on Claude.

3. What goes wrong when a small model has to call a tool, and how to fix it by choosing a better model or rewriting the prompt.

Estimated time: 15 minutes, plus the model download.

## Prerequisites

* Metagente 0.1.4 installed (zip from the [latest release](https://github.com/cleuton/MetaAgent/releases/latest), or build with `cargo build --release`).

* About 9 GB of free disk and 16 GB of RAM for the 14 billion parameter model. The 7 billion parameter model needs about 5 GB of disk and 8 GB of RAM.

* [uv](https://docs.astral.sh/uv/), because the agent starts its tool with `uvx mcp-server-fetch`.

* An Anthropic key (`ANTHROPIC_API_KEY`), only for step 6.

In the commands below, `metagente` means the binary. If it is not in your PATH, use `./bin/metagente` (Linux and macOS) or `.\bin\metagente.exe` (Windows).

## Step 1: install Ollama

Linux:

```text
curl -fsSL https://ollama.com/install.sh | sh
```

macOS: install the app from https://ollama.com/download, or use `brew install ollama`.

Windows: download the installer from https://ollama.com/download.

## Step 2: install uv

Linux and macOS:

```text
curl -LsSf https://astral.sh/uv/install.sh | sh
```

Windows (PowerShell):

```text
powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"
```

Check that it works. The first run downloads the tool, so it can take a moment:

```text
uvx mcp-server-fetch --help
```

## Step 3: download a model that supports tools

The model must be able to call tools (tool calling). A model that does not support tools is refused by Ollama when the agent asks for them. Two models of the Qwen2.5 family are a good start:

```text
ollama pull qwen2.5:14b
```

or, if you have less memory:

```text
ollama pull qwen2.5:7b
```

The rest of this README uses `qwen2.5:14b`. If you choose the other one, change the model name in `metagente.toml`.

Things we found in our tests:

* `llama3.1:8b` did not call tools reliably through Ollama. Instead of a tool call, it wrote its own tool format (`<|python_tag|>`) as plain text, and Metagente printed it as the answer.

* `deepseek-r1:7b` does not accept tools.

* Very small models (1B to 3B) answer simple questions but often fail when they have to use tools.

Support changes with each Ollama and model release. The Ollama model library lets you filter by the "tools" capability.

## Step 4: check that Ollama is answering

Ollama listens on port 11434 and offers an OpenAI compatible API under `/v1`. On Linux and macOS (app) it already runs as a service. If it is not running, open another terminal and run `ollama serve`.

List the models:

```text
curl http://127.0.0.1:11434/v1/models
```

You should see a JSON with `qwen2.5:14b`. Then warm the model up, so the first Metagente call does not spend its time loading it into memory:

```text
ollama run qwen2.5:14b "say only: ready"
```

Use `127.0.0.1` in addresses, not `localhost`. On some systems `localhost` resolves to IPv6 (`::1`) and Ollama listens only on IPv4.

## Step 5: create the project

Create a folder with two subfolders, one for each model:

```text
mkdir -p demo-ollama/local demo-ollama/cloud
cd demo-ollama
```

### 5.1 The agent

Create `local/researcher.ag`:

```text
agent Researcher
  goal "Find facts about a city and summarize them"
  tool fetch from mcp "uvx mcp-server-fetch"
  tool clock
  accepts research city  # find facts about a city
  on research
    if not city
      fail "I need a city"
    now = clock.now
    facts = think "Use the fetch tool to read https://en.wikipedia.org/wiki/{city} and write three short sentences of facts about {city}. If the page cannot be read, say that no facts were found."
    reply "{facts}\n(checked {now.text})"
```

An agent can use only what it declares. Here it declares the `fetch` tool (an MCP server started with `uvx`) and the `clock` tool. During a `think`, the model can call any of the declared tools.

### 5.2 The local model configuration

Create `local/metagente.toml`:

```toml
[llm]
provider = "openai-compatible"
base_url = "http://127.0.0.1:11434/v1"
model = "qwen2.5:14b"
api_key_env = "OLLAMA_KEY"
```

The `openai-compatible` provider speaks the OpenAI protocol at the address you give. Metagente builds the call as `http://127.0.0.1:11434/v1/chat/completions`, which is exactly what Ollama expects.

Ollama does not need a key, but Metagente reads the key from an environment variable on every call. So set any value:

Linux and macOS:

```text
export OLLAMA_KEY=ollama
```

Windows (PowerShell):

```text
$env:OLLAMA_KEY = "ollama"
```

## Step 6: run with the local model

First validate the file, without calling any model:

```text
metagente check local/researcher.ag
```

Then run it:

```text
metagente run local/researcher.ag research city="Lisbon"
```

What to expect: three short sentences about Lisbon, followed by a line like `(checked 2026-11-25T17:17:58+00:00)`. The time comes from the `clock` tool, and the facts come from the page the model read with `fetch`. The exact text changes on every run, because it depends on the model.

For a city with more than one word, use underscores, as in the Wikipedia address: `city="Rio_de_Janeiro"`.

To measure the time:

```text
time metagente run local/researcher.ag research city="Lisbon"
```

On Windows, use `Measure-Command { metagente run local/researcher.ag research city="Lisbon" }`.

## Step 7: the same agent with Claude

Copy the agent to the `cloud` folder and create another `metagente.toml` there:

```text
cp local/researcher.ag cloud/
```

Create `cloud/metagente.toml`:

```toml
[llm]
provider = "anthropic"
model = "claude-sonnet-5-5"
```

Set the key:

```text
export ANTHROPIC_API_KEY=<your key>
```

On Windows (PowerShell): `$env:ANTHROPIC_API_KEY = "<your key>"`.

Run the same command in the other folder:

```text
metagente run cloud/researcher.ag research city="Lisbon"
```

Compare the answers and the times. The `.ag` file is the same in both folders (check with `diff local/researcher.ag cloud/researcher.ag`). The agent says what to do, and `metagente.toml` decides who does it. That is the central idea of this demo.

## Small models and tools: rewrite your prompt

Models that you can download and run locally are small, and they often have trouble running tools. Calling a tool needs the model to decide to call it, to write the call in the exact format the server expects, and to wait for the result before answering. Small models skip or break one of these steps.

What it looks like when it goes wrong:

| What you see | What happened |
|--------------|---------------|
| The answer is `<\|python_tag\|>` followed by code or a strange call | The model wrote its own tool format as text, and Ollama did not turn it into a real tool call |
| The answer is a Python script that downloads the page | The model wrote code instead of calling the tool. Nothing was executed |
| Facts that look made up, with no reading of the page | The model answered without calling `fetch` |
| An analysis of HTML ("this is the source code of a page...") | The model read the raw page and described it instead of summarizing it |
| An error saying the model does not support tools | The model has no tool calling at all, as with `deepseek-r1` |

The original prompt is written for a strong model. It is short and leaves the details to the model:

```text
Use the fetch tool to read https://en.wikipedia.org/wiki/{city} and write three short sentences of facts about {city}. If the page cannot be read, say that no facts were found.
```

For a small model, be explicit about the tool name, what to pass to it, the order of the steps, and what not to do. Replace the `think` line in `local/researcher.ag` with this:

```text
    facts = think "You have a tool called fetch. Step 1: call the fetch tool with the url https://en.wikipedia.org/wiki/{city}. Do not write code and do not answer before the tool returns. Step 2: using only the text the tool returned, write exactly three short sentences of facts about {city}. Do not describe the page or its HTML. If the tool fails or returns nothing useful, answer only: no facts were found."
```

Rules of thumb for these prompts:

* Name the tool and say what to give it (`call the fetch tool with the url ...`).

* Number the steps, and say that the answer comes only after the tool returns.

* Say what not to do (`do not write code`, `do not describe the page`). Small models need the negative examples.

* Say what to use for the answer (`using only the text the tool returned`), so the model does not fill the gaps from memory.

* Give a fixed answer for failure, so a failed call is not hidden by an invented answer.

* Keep it short. A very long prompt hurts a small model more than it helps.

If it still fails, in order of effort:

1. Run the same command again. Small models are not consistent, and a single failure does not prove much.

2. Use a bigger model (`qwen2.5:14b` instead of `qwen2.5:7b`).

3. Run `ollama list` and check that the name in `metagente.toml` is exactly the one you downloaded.

4. Use a cloud model for the steps that need tools, as in step 7.

## Common problems

| Symptom | Likely cause | What to do |
|---------|--------------|------------|
| Cannot connect to the model | Ollama is not running, or the port is wrong | Run `curl http://127.0.0.1:11434/v1/models`. If it fails, start it with `ollama serve` |
| 404 or "model not found" | The name in `metagente.toml` differs from Ollama's | See the names with `ollama list` and copy exactly, including the part after the colon |
| Key variable not set | `OLLAMA_KEY` is not in the environment of this terminal | Run the `export` (or `$env:`) in the same terminal where you run Metagente |
| The first call is very slow | The model is being loaded into memory | Warm it up with `ollama run qwen2.5:14b "hi"` before the demo |
| Connection refused with Ollama running | `HTTP_PROXY` is set and Metagente sends local traffic through the proxy | Set `NO_PROXY=127.0.0.1,localhost` in the environment |
| An error starting the `fetch` tool | `uv` is not installed or not in the PATH | Install `uv` (step 2) and open a new terminal |
| An error saying the model address uses http | You turned on `[secure] llm = true` | That option requires https. For the local demo, leave `[secure]` off |
| The agent answers without using the tool, or fails to call it | The local model is too small for tool calling | See the section on small models above |

## Limits of this demo (version 0.1.4)

* Each project uses a single model, the one in `[llm]`. That is why the demo uses two folders. Mixing Ollama and Claude in the same agent (for example `think "..." using local` and `think "..." using claude`) depends on the several models spec, which is not implemented yet, and which today has no address of its own for named models.

* Ollama over `http://` does not pass the `[secure] llm = true` rule. The traffic stays on your own machine, but the rule does not distinguish that.

* The quality of the answers, and of the tool use, depends on the local model you choose.

## Suggested script for a live presentation

1. Show `researcher.ag` (ten lines) and the local `metagente.toml`.

2. Run it with Ollama and show, with `ollama ps` in another terminal, that the model is running on your machine. The agent still needs the internet, because the `fetch` tool reads the Wikipedia page.

3. Show a run where a small model fails with the `<|python_tag|>` text, and explain why.

4. Switch to the rewritten prompt, or to `qwen2.5:14b`, and run again.

5. Go to the `cloud` folder and run the same agent with Claude.

6. Close with the `diff` of the two `.ag` files, showing that they are identical.
