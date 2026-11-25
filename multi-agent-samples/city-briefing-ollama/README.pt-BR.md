# Demo: Metagente com Ollama (modelo local)

🇬🇧 [Read in English](README.md)

Esta demo roda o mesmo agente Metagente duas vezes: primeiro com um modelo rodando na sua própria máquina, através do Ollama, depois com o Claude na nuvem. O arquivo do agente é idêntico nas duas execuções. Só o `metagente.toml` muda.

Ela usa apenas recursos que já existem na versão 0.1.4. Não precisa de nenhuma alteração no código do Metagente.

A demo é baseada no agente `researcher.ag`, que lê uma página da Wikipedia com uma ferramenta MCP (`fetch`) e a resume. É um bom teste, porque exige **tool calling**, e é exatamente aí que os modelos locais pequenos têm dificuldade. Há uma seção sobre isso mais abaixo, com um prompt reescrito para modelos pequenos.

## O que você vai ver

1. O agente `Researcher` usando um modelo local para ler uma página da web por uma ferramenta MCP e escrever três frases sobre uma cidade.

2. O mesmo agente, sem mudar nada no `.ag`, rodando no Claude.

3. O que dá errado quando um modelo pequeno precisa chamar uma ferramenta, e como resolver escolhendo um modelo melhor ou reescrevendo o prompt.

Tempo estimado: 15 minutos, mais o download do modelo.

## Pré-requisitos

* Metagente 0.1.4 instalado (zip da [última release](https://github.com/cleuton/MetaAgent/releases/latest) ou build com `cargo build --release`).

* Cerca de 9 GB livres em disco e 16 GB de RAM para o modelo de 14 bilhões de parâmetros. O modelo de 7 bilhões precisa de uns 5 GB de disco e 8 GB de RAM.

* O [uv](https://docs.astral.sh/uv/), porque o agente inicia sua ferramenta com `uvx mcp-server-fetch`.

* Uma chave da Anthropic (`ANTHROPIC_API_KEY`), apenas para o passo 7.

Nos comandos abaixo, `metagente` significa o binário. Se ele não estiver no PATH, use `./bin/metagente` (Linux e macOS) ou `.\bin\metagente.exe` (Windows).

## Passo 1: instalar o Ollama

Linux:

```text
curl -fsSL https://ollama.com/install.sh | sh
```

macOS: instale o aplicativo em https://ollama.com/download ou use `brew install ollama`.

Windows: baixe o instalador em https://ollama.com/download.

## Passo 2: instalar o uv

Linux e macOS:

```text
curl -LsSf https://astral.sh/uv/install.sh | sh
```

Windows (PowerShell):

```text
powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"
```

Confira se funciona. A primeira execução baixa a ferramenta, então pode demorar um pouco:

```text
uvx mcp-server-fetch --help
```

## Passo 3: baixar um modelo que suporta ferramentas

O modelo precisa saber chamar ferramentas (tool calling). Um modelo sem suporte a ferramentas é recusado pelo Ollama quando o agente as pede. Dois modelos da família Qwen2.5 são um bom começo:

```text
ollama pull qwen2.5:14b
```

ou, se você tiver menos memória:

```text
ollama pull qwen2.5:7b
```

O resto deste README usa `qwen2.5:14b`. Se escolher o outro, troque o nome do modelo no `metagente.toml`.

O que encontramos nos nossos testes:

* O `llama3.1:8b` não chamou ferramentas de forma confiável pelo Ollama. Em vez de uma chamada de ferramenta, ele escreveu o próprio formato de ferramenta (`<|python_tag|>`) como texto puro, e o Metagente o mostrou como resposta.

* O `deepseek-r1:7b` não aceita ferramentas.

* Modelos muito pequenos (1B a 3B) respondem perguntas simples, mas falham com frequência quando precisam usar ferramentas.

O suporte muda a cada versão do Ollama e dos modelos. A biblioteca de modelos do Ollama permite filtrar pela capacidade "tools".

## Passo 4: conferir se o Ollama está respondendo

O Ollama escuta na porta 11434 e oferece uma API compatível com OpenAI em `/v1`. No Linux e no macOS (app) ele já roda como serviço. Se não estiver rodando, abra outro terminal e execute `ollama serve`.

Liste os modelos:

```text
curl http://127.0.0.1:11434/v1/models
```

Deve aparecer um JSON com `qwen2.5:14b`. Depois aqueça o modelo, para que a primeira chamada do Metagente não gaste tempo carregando-o na memória:

```text
ollama run qwen2.5:14b "diga apenas: pronto"
```

Use `127.0.0.1` nos endereços, e não `localhost`. Em alguns sistemas `localhost` resolve para IPv6 (`::1`) e o Ollama escuta apenas em IPv4.

## Passo 5: criar o projeto

Crie uma pasta com duas subpastas, uma para cada modelo:

```text
mkdir -p demo-ollama/local demo-ollama/cloud
cd demo-ollama
```

### 5.1 O agente

Crie `local/researcher.ag`:

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

Um agente só pode usar o que declara. Aqui ele declara a ferramenta `fetch` (um servidor MCP iniciado com `uvx`) e a ferramenta `clock`. Durante um `think`, o modelo pode chamar qualquer uma das ferramentas declaradas.

### 5.2 A configuração do modelo local

Crie `local/metagente.toml`:

```toml
[llm]
provider = "openai-compatible"
base_url = "http://127.0.0.1:11434/v1"
model = "qwen2.5:14b"
api_key_env = "OLLAMA_KEY"
```

O provider `openai-compatible` fala o protocolo da OpenAI no endereço que você indicar. O Metagente monta a chamada como `http://127.0.0.1:11434/v1/chat/completions`, que é exatamente o que o Ollama espera.

O Ollama não exige chave, mas o Metagente lê a chave de uma variável de ambiente a cada chamada. Por isso defina um valor qualquer:

Linux e macOS:

```text
export OLLAMA_KEY=ollama
```

Windows (PowerShell):

```text
$env:OLLAMA_KEY = "ollama"
```

## Passo 6: rodar com o modelo local

Primeiro valide o arquivo, sem chamar modelo nenhum:

```text
metagente check local/researcher.ag
```

Depois rode:

```text
metagente run local/researcher.ag research city="Lisbon"
```

O que esperar: três frases curtas sobre Lisboa, seguidas de uma linha como `(checked 2026-11-25T17:17:58+00:00)`. A hora vem da ferramenta `clock`, e os fatos vêm da página que o modelo leu com o `fetch`. O texto exato muda a cada execução, porque depende do modelo.

Para uma cidade com mais de uma palavra, use sublinhados, como no endereço da Wikipedia: `city="Rio_de_Janeiro"`.

Para medir o tempo:

```text
time metagente run local/researcher.ag research city="Lisbon"
```

No Windows, use `Measure-Command { metagente run local/researcher.ag research city="Lisbon" }`.

## Passo 7: o mesmo agente com o Claude

Copie o agente para a pasta `cloud` e crie outro `metagente.toml` nela:

```text
cp local/researcher.ag cloud/
```

Crie `cloud/metagente.toml`:

```toml
[llm]
provider = "anthropic"
model = "claude-sonnet-5-5"
```

Defina a chave:

```text
export ANTHROPIC_API_KEY=<sua chave>
```

No Windows (PowerShell): `$env:ANTHROPIC_API_KEY = "<sua chave>"`.

Rode o mesmo comando na outra pasta:

```text
metagente run cloud/researcher.ag research city="Lisbon"
```

Compare as respostas e os tempos. O arquivo `.ag` é o mesmo nas duas pastas (confira com `diff local/researcher.ag cloud/researcher.ag`). O agente diz o que fazer, e o `metagente.toml` decide quem executa. Essa é a ideia central da demo.

## Modelos pequenos e ferramentas: reescreva o seu prompt

Os modelos que você pode baixar e rodar localmente são pequenos, e muitas vezes têm problemas ao executar ferramentas. Chamar uma ferramenta exige que o modelo decida chamá-la, escreva a chamada no formato exato que o servidor espera e espere o resultado antes de responder. Modelos pequenos pulam ou quebram um desses passos.

Como aparece quando dá errado:

| O que você vê | O que aconteceu |
|---------------|-----------------|
| A resposta é `<\|python_tag\|>` seguido de código ou de uma chamada estranha | O modelo escreveu o próprio formato de ferramenta como texto, e o Ollama não o transformou em uma chamada real |
| A resposta é um script Python que baixa a página | O modelo escreveu código em vez de chamar a ferramenta. Nada foi executado |
| Fatos que parecem inventados, sem leitura da página | O modelo respondeu sem chamar o `fetch` |
| Uma análise de HTML ("este é o código-fonte de uma página...") | O modelo leu a página crua e a descreveu em vez de resumi-la |
| Um erro dizendo que o modelo não suporta ferramentas | O modelo não tem tool calling, como no `deepseek-r1` |

O prompt original foi escrito para um modelo forte. Ele é curto e deixa os detalhes por conta do modelo:

```text
Use the fetch tool to read https://en.wikipedia.org/wiki/{city} and write three short sentences of facts about {city}. If the page cannot be read, say that no facts were found.
```

Para um modelo pequeno, seja explícito sobre o nome da ferramenta, o que passar a ela, a ordem dos passos e o que não fazer. Troque a linha do `think` em `local/researcher.ag` por esta:

```text
    facts = think "You have a tool called fetch. Step 1: call the fetch tool with the url https://en.wikipedia.org/wiki/{city}. Do not write code and do not answer before the tool returns. Step 2: using only the text the tool returned, write exactly three short sentences of facts about {city}. Do not describe the page or its HTML. If the tool fails or returns nothing useful, answer only: no facts were found."
```

Regras práticas para esses prompts:

* Dê o nome da ferramenta e diga o que entregar a ela (`call the fetch tool with the url ...`).

* Numere os passos e diga que a resposta só vem depois que a ferramenta retornar.

* Diga o que não fazer (`do not write code`, `do not describe the page`). Modelos pequenos precisam dos exemplos negativos.

* Diga o que usar na resposta (`using only the text the tool returned`), para o modelo não preencher as lacunas de memória.

* Dê uma resposta fixa para o caso de falha, para que uma chamada que falhou não fique escondida por uma resposta inventada.

* Seja curto. Um prompt muito longo atrapalha mais o modelo pequeno do que ajuda.

Se ainda falhar, em ordem de esforço:

1. Rode o mesmo comando de novo. Modelos pequenos não são consistentes, e uma falha isolada prova pouco.

2. Use um modelo maior (`qwen2.5:14b` no lugar do `qwen2.5:7b`).

3. Rode `ollama list` e confira que o nome no `metagente.toml` é exatamente o que você baixou.

4. Use um modelo da nuvem para os passos que precisam de ferramentas, como no passo 7.

## Problemas comuns

| Sintoma | Causa provável | O que fazer |
|---------|----------------|-------------|
| Não conecta ao modelo | O Ollama não está rodando, ou a porta está errada | Rode `curl http://127.0.0.1:11434/v1/models`. Se falhar, inicie com `ollama serve` |
| Erro 404 ou "model not found" | O nome no `metagente.toml` não é igual ao do Ollama | Veja os nomes com `ollama list` e copie exatamente, incluindo a parte depois dos dois pontos |
| Erro de chave não definida | `OLLAMA_KEY` não está no ambiente do terminal atual | Rode o `export` (ou `$env:`) no mesmo terminal em que roda o Metagente |
| Primeira chamada muito lenta | O modelo está sendo carregado na memória | Aqueça com `ollama run qwen2.5:14b "oi"` antes da demo |
| Conexão recusada mesmo com o Ollama no ar | Há `HTTP_PROXY` definido e o Metagente está mandando o tráfego local pelo proxy | Defina `NO_PROXY=127.0.0.1,localhost` no ambiente |
| Erro ao iniciar a ferramenta `fetch` | O `uv` não está instalado ou não está no PATH | Instale o `uv` (passo 2) e abra um novo terminal |
| Erro dizendo que o endereço do modelo usa http | Você ligou `[secure] llm = true` | Essa opção exige https. Para a demo local, deixe `[secure]` desligado |
| O agente responde sem usar a ferramenta, ou não consegue chamá-la | O modelo local é pequeno demais para tool calling | Veja a seção sobre modelos pequenos acima |

## Limites desta demo (versão 0.1.4)

* Cada projeto usa um único modelo, o do `[llm]`. Por isso a demo usa duas pastas. Misturar Ollama e Claude no mesmo agente (por exemplo, `think "..." using local` e `think "..." using claude`) depende da spec de vários modelos, que ainda não está implementada, e que hoje não prevê endereço próprio para modelos nomeados.

* O Ollama em `http://` não passa na regra `[secure] llm = true`. O tráfego fica na própria máquina, mas a regra não distingue isso.

* A qualidade das respostas e do uso de ferramentas depende do modelo local escolhido.

## Sugestão de roteiro para uma apresentação ao vivo

1. Mostrar o `researcher.ag` (dez linhas) e o `metagente.toml` local.

2. Rodar com o Ollama e mostrar, com `ollama ps` em outro terminal, que o modelo está rodando na sua máquina. O agente ainda precisa da internet, porque a ferramenta `fetch` lê a página da Wikipedia.

3. Mostrar uma execução em que um modelo pequeno falha com o texto `<|python_tag|>` e explicar por quê.

4. Trocar para o prompt reescrito, ou para o `qwen2.5:14b`, e rodar de novo.

5. Ir para a pasta `cloud` e rodar o mesmo agente com o Claude.

6. Fechar com o `diff` dos dois `.ag`, mostrando que são idênticos.
