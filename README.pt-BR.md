# Metagente

🇬🇧 [Read in English](README.md)

**Construa agentes de IA em poucas linhas, não em algumas centenas.**

Metagente é uma pequena linguagem para construir agentes de IA. Você descreve para que serve o agente, quais ferramentas
ele pode usar e o que ele responde; o interpretador faz o resto. Ele fala **MCP** (Model Context Protocol) e **A2A**
(Agent to Agent) nativamente, então seus agentes podem usar qualquer ferramenta MCP e conversar com outros agentes sem
nenhuma configuração extra.

**Vídeo de demonstração**
[![Demo do Metagente](https://img.youtube.com/vi/zrNYqy84Ak8/maxresdefault.jpg)](https://www.youtube.com/watch?v=zrNYqy84Ak8)

Esta demonstração está em [**multi-agent-samples**](./multi-agent-samples/metagente-demo-recrutador/)

O interpretador é um único binário escrito em Rust. Sem ambiente Python, sem framework para aprender.

<!-- DEMO: replace with a GIF of writing weather.ag and running it (10 to 15 seconds). -->
![Demo](docs/demo.gif)

```text
agent Weather
  goal "Answer questions about the weather"
  tool weather from mcp "npx -y weather-mcp"
  accepts ask city
  on ask
    forecast = weather.forecast city: city
    reply "In {city} it will be {forecast.summary}"
```

Esse é um agente completo: um objetivo, uma ferramenta MCP, uma entrada e uma resposta.

## Experimente em 30 segundos

1. Baixe o zip para o seu sistema na [última release](https://github.com/cleuton/MetaAgent/releases/latest):
   `metagente-linux-amd64.zip`, `metagente-macos-arm64.zip` ou `metagente-windows-amd64.zip`.
2. Descompacte e abra um terminal na pasta.
3. Rode o primeiro exemplo:

```text
./bin/metagente run samples/clock.ag now
```

No Windows, use `.\bin\metagente.exe`. Depois crie seu próprio agente:

```text
./bin/metagente new hello
./bin/metagente run hello.ag greet name=World
```

Prefere compilar a partir do código-fonte? Você precisa do [Rust](https://rustup.rs):

```text
git clone https://github.com/cleuton/MetaAgent
cd MetaAgent
cargo build --release
target/release/metagente run examples/clock.ag now
```

## Escreva suas próprias ferramentas em Python, TypeScript, Go e Java

Um agente pode chamar uma **ferramenta**: um pequeno programa, escrito em qualquer linguagem, que faz uma tarefa. O
Metagente não tem banco de dados, nem planilhas, nem aprendizado de máquina; sua ferramenta tem o que a linguagem dela
tiver. Você a conecta com uma linha:

```text
tool calc from mcp @parameters.calc_command
```

**[Leia o guia de interoperabilidade: ferramentas em Python, TypeScript, Go e Java](docs/mcp-tools-guide.md)** (em inglês)

- O mesmo exemplo escrito quatro vezes, e **os mesmos dois agentes** (byte a byte) em todas as linguagens. Mude uma linha
  do `metagente.toml` e o agente passa a usar uma ferramenta em outra linguagem.
- Uma ferramenta que lê um banco de dados.
- `smoke.py`, um programa que testa uma ferramenta antes de qualquer agente usá-la.
- "Quebre de propósito": o que você vê quando algo dá errado, e uma tabela de problemas comuns.
- Todos os comandos e saídas foram executados de verdade. Os arquivos estão em [multi-agent-samples/mcp-tools](multi-agent-samples/mcp-tools/README.md).

## Por que Metagente?

Frameworks como LangChain ou CrewAI são poderosos, mas assumem que você é um programador trabalhando dentro de um
projeto Python. O Metagente faz uma troca diferente:

| | Frameworks de agentes em Python | Metagente |
|---|---|---|
| O que você escreve | Código Python usando uma biblioteca | Um pequeno arquivo declarativo de agente |
| Instalação | Python, ambiente virtual, pacotes | Um binário |
| Ferramentas | Adaptadores específicos de cada framework | Qualquer servidor MCP |
| Agentes conversando com agentes | Em geral dentro do mesmo processo | A2A, entre processos e máquinas |
| Segurança | Depende do seu código | Permissões por agente para pastas, variáveis de ambiente e links |
| Erros | Stack traces | Mensagens amigáveis para iniciantes |

Se você precisa de controle fino em Python, use um framework. Se quer um agente funcionando rápido, ou quer dar a
quem não programa uma forma de construir agentes, o Metagente é para você.

> **Nota (0.1.4): não testado no Microsoft Windows.** O validador que o `[auth]` executa é iniciado com um ambiente limpo
> (apenas `PATH` e `SystemRoot`). Isso ainda não foi testado no Windows. Linux e macOS são os sistemas testados para ele.

## O que funciona hoje

- **Ferramentas em Python, TypeScript, Go e Java:** escreva uma ferramenta na sua linguagem e conecte-a a um agente com uma linha. Veja o [guia de interoperabilidade](docs/mcp-tools-guide.md).
- **Linguagem e interpretador:** objetivos, ferramentas, entradas, respostas, `if`, laços e resultados.
- **Ferramentas embutidas e ferramentas MCP:** conecte qualquer servidor MCP.
- **Agentes seguros:** cada agente declara quais pastas, variáveis de ambiente e outros agentes pode usar.
- **Link dinâmico:** agentes chamam agentes por nome ou caminho, com verificação de interface e detecção de ciclos.
- **Servidor A2A e MCP:** exponha um agente como servidor MCP ou com um Agent Card A2A, e envie ou receba tarefas A2A. Qualquer agente servido com `--mcp` é uma ferramenta que outros agentes (ou qualquer cliente MCP) podem usar: veja [Um agente como ferramenta](docs/tutorial.md#an-agent-as-a-tool).
- **CLI:** `new`, `check`, `run` e `serve`.
- **Texto com várias linhas** (novo na 0.1.2): um prompt com muitas linhas entre `"""` e `"""`, mantido exatamente como digitado.
- **Parâmetros externos** (novo na 0.1.2): endereços e prompts ficam no `metagente.toml` e os agentes os leem como `@parameters.nome`.
- **Conexões seguras** (novo na 0.1.3): endereços `https://` em todo lugar, `[secure]` para exigir https por nome, `metagente serve` respondendo por https (`--tls-cert`, `--tls-key`), certificados autoassinados apenas quando o `metagente.toml` permite, e proxies autenticados.
- **Interoperabilidade com Python** (novo na 0.1.3): testado com o SDK oficial A2A para Python, nos dois sentidos.
- **Vários modelos de linguagem** (novo na 0.1.5): `think "..." using fast` escolhe qual modelo responde; os modelos são sub-tabelas `[llm.NOME]` do `[llm]` no `metagente.toml`, Anthropic ou OpenAI nos endereços oficiais, com o nome do modelo e a chave vindos de variáveis de ambiente.
- **Versão mínima e tokens bearer** (novo na 0.1.4): `[version]` faz um Metagente mais antigo recusar um projeto; `[auth]` protege mensagens com um token que um programa seu valida, e repassa o token para os agentes e ferramentas que você listar.

## Você pode usar modelos locais!

Sim! Você pode usar modelos locais com o **OLLAMA**. Veja [**esta demo!**](./multi-agent-samples/city-briefing-ollama/).

### Novo na 0.1.5: vários modelos de linguagem para o `think`

**Uma palavra opcional na linha do `think`, e nenhuma seção nova.** `think "..." using fast` envia aquela chamada ao modelo
chamado `fast`; um `think` sem `using` continua usando o modelo do `[llm]`, exatamente como antes. Os modelos são
sub-tabelas da seção `[llm]` que você já conhece: `[llm.fast]`, `[llm.strong]`. Só a linha do agente e o `metagente.toml`
mudam. Um exemplo completo e executável é [multi-agent-samples/two-models](multi-agent-samples/two-models/README.md).

```text
agent Assistant
  goal "Answer a question in two steps"
  accepts ask question
  on ask
    short = think "Rewrite as a short search query: {question}" using fast
    reply think "Answer this well: {short}" using strong
```

#### `[llm.NOME]`: as configurações

| Configuração | Obrigatória | O que é | Se você omitir |
|--------------|-------------|---------|----------------|
| `provider` | sim | `anthropic` ou `openai` | um erro |
| `model` | `openai`: esta ou `model_env` | o nome do modelo | `anthropic`: `claude-sonnet-5-5` |
| `model_env` | veja `model` | o **nome de uma variável** que guarda o nome do modelo, lida na primeira vez que o modelo é usado | o `model` do arquivo |
| `api_key_env` | não | o **nome da variável** que guarda a chave, lida a cada chamada | `ANTHROPIC_API_KEY` ou `OPENAI_API_KEY` |

* O nome `NOME` (o apelido) começa com uma letra e tem no máximo 24 letras, dígitos, `_` ou `-`. Não pode ser `provider`,
  `model`, `api_key_env` ou `base_url`, nem uma palavra da linguagem (`reply`, `think`, `using`...).
* Um modelo nomeado sempre vai ao endereço oficial da empresa dele. Não existe `base_url`: para outro endereço (um
  gateway, um serviço compatível com OpenAI) continue usando a seção `[llm]`, que não mudou.
* `using` só é uma palavra especial logo depois do prompt de um `think`, seguida do apelido e do fim da linha. Em qualquer
  outro lugar ela é um nome comum, então um arquivo que tem uma variável chamada `using` continua funcionando.
* O apelido é escrito no arquivo. Ele não pode vir de um parâmetro, de uma mensagem ou de uma variável.

#### Copie e cole: só Anthropic

```toml
[version]
minimum = "0.1.5"

[llm.claude]
provider = "anthropic"
model_env = "CLAUDE_MODEL"        # o NOME de uma variável; você a define no seu terminal
api_key_env = "ANTHROPIC_API_KEY"
```

```text
reply think "Say hello in one sentence" using claude
```

#### Copie e cole: só OpenAI

```toml
[version]
minimum = "0.1.5"

[llm.gpt]
provider = "openai"
model_env = "GPT_MODEL"
api_key_env = "OPENAI_API_KEY"
```

```text
reply think "Say hello in one sentence" using gpt
```

#### Copie e cole: os dois, em um agente

```toml
[version]
minimum = "0.1.5"

[llm.fast]
provider = "anthropic"
model_env = "FAST_MODEL"
api_key_env = "FAST_KEY"

[llm.strong]
provider = "openai"
model_env = "STRONG_MODEL"
api_key_env = "STRONG_KEY"
```

Para manter também um modelo padrão, escreva-o no `[llm]` como antes; então o `think` sem `using` o usa:

```toml
[llm]
provider = "anthropic"
model = "claude-sonnet-5-5"

[llm.gpt]
provider = "openai"
model_env = "OPENAI_MODEL"
```

#### Por que as chaves ficam só em variáveis de ambiente

Uma chave escrita em um arquivo acaba em um repositório, em um backup ou em uma captura de tela. No `metagente.toml` você
escreve apenas o **nome** da variável, e a chave fica no seu shell. Uma configuração chamada `key`, `api_key`, `token`,
`secret` ou `password` em uma sub-tabela é recusada. O Metagente nunca imprime uma chave, nunca deixa um agente ler a
variável (o `tool env` a recusa mesmo quando o agente a declara) e a envia só ao endereço oficial da própria empresa.
Uma mensagem sobre uma sub-tabela não imprime nada do arquivo além de um nome simples e curto, então uma chave colada
onde deveria haver um nome é recusada sem ser mostrada.

#### Use o `[version]`

O Metagente 0.1.4 e os anteriores não conhecem o `[llm.NOME]` (eles o ignoram com uma nota) e param no `using`. Coloque
`[version] minimum = "0.1.5"` no projeto; o Metagente 0.1.5 avisa quando uma sub-tabela está definida sem ele.

#### O que você vê quando algo está errado

Toda mensagem traz a linha e a correção. As mensagens do programa aparecem em inglês, exatamente como o Metagente as
imprime. As principais:

| Situação | Mensagem |
|----------|----------|
| Um nome de modelo que não existe | `There is no model called fats. The models are: fast, strong.` e `did you mean fast?` |
| Uma configuração com erro de digitação | `[llm.fast] has the setting api_key_evn, which I do not know.` e `did you mean api_key_env?` |
| `model` e `model_env` juntos | `[llm.fast] has both model and model_env.` |
| Uma chave escrita no arquivo | `[llm.fast] has a key written in the file. Keys are never kept here.` |
| Um endereço em um modelo nomeado | `[llm.fast] has base_url. A named model always uses the official address.` |
| Um provider diferente dos dois | `The provider anthropik in [llm.fast] is not one a named model accepts. Use anthropic or openai.` |
| Sem provider, ou `openai` sem nome de modelo | `The provider is missing in [llm.fast].` / `The provider openai needs a model name in [llm.fast].` |
| Uma variável que não está definida | `The model fast needs the variable FAST_KEY, and it is not set.` |
| Uma chamada falha | `The model fast (anthropic) answered with status 429.` ou `...could not be reached: timeout.` (nunca o texto do serviço, um nome de modelo ou um endereço) |
| `[llm.fast]` sem `[version]` | `Warning: [llm.fast] is set but [version] is not. Older programs do not know it.` |

O `metagente check` também avisa quando a variável de chave ou de modelo de um modelo em uso não está definida onde você o
executa. Enquanto o `metagente serve` roda, uma edição dos modelos não é aplicada: uma linha pede a reinicialização.

### Novo na 0.1.4: uma versão mínima, e tokens para mensagens servidas

**Nenhuma sintaxe mudou.** As duas seções ficam só no `metagente.toml`, nunca em um arquivo `.ag`. Um exemplo completo e
executável é [multi-agent-samples/auth-chain](multi-agent-samples/auth-chain/README.md) (três agentes, um token, nenhuma
chave de modelo). Leia o [guia de autenticação](docs/authentication-guide.md) antes de usar com usuários reais.

#### `[version]`: diga qual Metagente um projeto precisa

```toml
[version]
minimum = "0.1.4"      # três números inteiros, sem zeros à esquerda; o projeto roda nessa versão e em qualquer mais nova
```

* Um programa mais antigo interrompe `run`, `check` e `serve` com *"This project needs Metagente 0.1.4 or newer, and this is
  0.1.3. Nothing was run."* antes de olhar qualquer outra coisa no arquivo. Nenhuma seção, nenhuma verificação.
* **Sempre defina esta seção quando usar o `[auth]`.** Programas anteriores à 0.1.4 não conhecem o `[auth]`: eles o ignoram
  e servem todas as mensagens **sem proteção**. O `[version]` faz com que eles recusem o projeto. O Metagente 0.1.4 avisa
  quando o `[auth]` está definido sem o `[version]`, e o `metagente new` escreve o `[version]` para você.

#### `[auth]`: proteja mensagens com um token bearer

```toml
[auth]
validator = "python3 validator.py"           # seu programa: o token na entrada padrão, código de saída 0 = aceito
protect = ["Planner.plan", "Worker.*"]       # Agente.mensagem, ou Agente.* para todas as mensagens de um agente
forward = ["Researcher", "kb"]               # agentes remotos e ferramentas http(s) que recebem o token de quem chamou
cache_seconds = 30                           # 0 a 300, padrão 0: lembra um token ACEITO por esse tempo
timeout_seconds = 5                          # 1 a 60, padrão 5: o limite de tempo do validador
```

| Chave | Padrão | Limites | Significado |
|-------|--------|---------|-------------|
| `validator` | nenhum | texto | O comando. Iniciado sem shell, em um ambiente limpo (apenas `PATH`). Obrigatório quando `protect` não está vazio. |
| `protect` | `[]` | lista de `Agente.mensagem` ou `Agente.*` | Mensagens que exigem um token aceito por A2A e por MCP em http. |
| `forward` | `[]` | nomes de agentes `remote` e de ferramentas MCP http(s) | Recebem `Authorization: Bearer <o token de quem chamou>`. Nada mais recebe. |
| `cache_seconds` | `0` | 0 a 300 | Um token que o validador aceitou é lembrado por esse tempo. Uma recusa nunca é. |
| `timeout_seconds` | `5` | 1 a 60 | O tempo de espera, de início e de execução do validador, somados. |

* O Metagente nunca abre um token, verifica uma assinatura ou sabe quem é uma pessoa. Seu validador decide.
* Sem token, com um token malformado, ou com um que o validador recusa: HTTP 401 com `WWW-Authenticate: Bearer`.
  Se o validador está ausente, lento demais ou quebrado: HTTP 503. Nada do agente é executado. Todo erro de digitação no
  `[auth]` interrompe `run`, `serve` e `check`.
* O Agent Card, o `initialize` e o `tools/list` do MCP, e toda mensagem que não está em `protect` continuam públicos.
* Uma mensagem protegida nunca é servida em aberto: http simples em outro endereço (`--public`) e `--mcp stdio` são
  recusados; http simples neste computador inicia com um aviso.
* O `link` não alcança uma mensagem protegida a partir de uma requisição que não trouxe um token aceito, e o `GetTask`
  devolve uma tarefa protegida apenas ao token que a iniciou.
* Experimente: [multi-agent-samples/auth-chain](multi-agent-samples/auth-chain/README.md).

**Não testado no Microsoft Windows.** O validador roda em um ambiente limpo (apenas `PATH` e `SystemRoot`) e isso ainda
não foi testado no Windows.

Mensagens que você verá:

| Situação | Mensagem |
|----------|----------|
| Sem token | `Planner.plan needs a token, and this request had none. Send the header Authorization: Bearer <token>.` |
| Token malformado | `The Authorization header of this request is not a bearer token I can read.` |
| Não aceito | `The token was not accepted for Planner.plan.` |
| Problema no validador | `I could not ask the validator (python3 validator.py): it did not answer in 5 seconds. Nothing was run.` |
| Rede aberta | `Planner.plan is protected, and this address is plain http and open to other computers: 0.0.0.0:8080. A token would travel in the open.` |
| Por stdio | `Planner.plan is protected, and stdio has no place for a token.` |
| Chamada por link | `Admin.delete is protected, and the request that reached Planner did not carry an accepted token.` |
| Tarefa | `This task needs the token that started it.` |
| Chave com erro de digitação | `[auth] has the setting protec, which I do not know. Nothing was served, because a misspelled setting could leave messages open.` |
| Nome que não corresponde a nada | `[auth] protect lists Planner.plann, but no such message is declared.` |
| Não pode receber | `files is a built in tool, so it cannot receive a token.` |
| O destino quer um token | `Researcher refused the call: it needs a token. If it needs the one this agent received, add Researcher to forward in [auth].` |
| Versão antiga demais | `This project needs Metagente 0.1.4 or newer, and this is 0.1.3. Nothing was run.` |
| Formato de versão ruim | `[version] minimum must be three numbers like 0.1.4, but it says ">=0.1.4".` |
| `[version]` ausente | `Warning: [auth] is set but [version] is not. Metagente 0.1.3 and older ignore [auth] and would serve every message without protection.` |

### Novo na 0.1.3: conexões seguras (https nas duas pontas)

Um endereço que começa com `https://` simplesmente funciona: agentes, ferramentas, endereços web e o modelo de linguagem
são alcançados por TLS, e o certificado é verificado contra os certificados do seu computador (o binário também traz um
conjunto próprio, então funciona até em uma máquina sem nada instalado). **Nenhuma sintaxe mudou.** A forma de um
programa pedir uma conexão segura é o próprio endereço: escreva `https://` no lugar de `http://`, no arquivo ou vindo de
`@parameters.nome`.

Duas coisas são novas além disso: você pode **exigir** https para um nome, e o `metagente serve` pode **responder** por
https. Um exemplo completo e executável é [multi-agent-samples/https-chain](multi-agent-samples/https-chain/README.md):
um agente chama um agente, que chama uma ferramenta, tudo por https, com um certificado que você mesmo cria no seu
computador. Novo em certificados? Comece pelo [guia de conexões seguras](docs/secure-connections-guide.md), que os ensina
do zero.

#### Exigir https para um nome: `[secure]`

Opcional. A comunicação segura também é opcional: sem esta seção nada muda. Ela fica só no `metagente.toml`, nunca em um
arquivo `.ag`, então um agente não pode enfraquecê-la.

```toml
[secure]
agents = ["Bob", "Researcher"]   # nomes de agentes, exatamente como nos arquivos .ag (diferencia maiúsculas)
tools  = ["weather", "http"]     # nomes de ferramentas; "http" faz a ferramenta http embutida usar só https
llm    = false                   # true: o endereço do modelo de linguagem deve ser https
```

* Um nome em `agents` é um `remote Bob at "..."` que você chama, **e** um agente que você serve.
  Chamá-lo por `http://` é recusado com uma mensagem simples **antes de abrir qualquer conexão**, e servi-lo exige um
  certificado (veja abaixo).
* Um nome em `tools` é uma ferramenta que você alcança por um endereço `http://` ou `https://`. Uma ferramenta que roda
  como programa local (`stdio`) não tem endereço para proteger, então listá-la é um erro que diz isso. O mesmo vale para
  as ferramentas embutidas que não sejam `http`.
* `llm = true` recusa um endereço de modelo que começa com `http://`, antes de a chave ser enviada. Os endereços
  hospedados padrão são https e continuam funcionando.
* **Um agente em `agents` é seguro em todas as direções**: tudo o que ele serve é só https, e toda chamada de rede feita
  pelos próprios handlers dele (agentes remotos, ferramentas alcançadas por http, a ferramenta `http`, o modelo) deve ser
  https, ou é recusada. Um agente que ele carrega com `link` segue a mesma regra, porque um agente ligado roda no
  contexto do agente que o carregou. Ferramentas locais (`stdio`, `file`, `state`, `env`, `clock`) não são tráfego de
  rede e continuam funcionando.
* Para uma ferramenta de terceiros, o Metagente recusa http e recusa ser enviado de https para http. Isso é tudo o que
  ele pode garantir: o que o outro servidor faz com seus dados está fora do controle dele. Para agentes e ferramentas que
  o próprio Metagente serve, ele garante também o lado do servidor.
* O `metagente check` reporta um endereço `http://` para qualquer item listado, um nome listado que não corresponde a
  nada (um provável erro de digitação, com o nome mais próximo), chaves desconhecidas e tipos errados. Uma mudança no
  `[secure]` vale para a próxima requisição enquanto o servidor roda, sem reinicialização.

#### Servir por https

```text
metagente serve agent.ag --a2a 8443 --mcp 9443 --tls-cert certs/cert.pem --tls-key certs/key.pem
```

ou, uma vez só, no `metagente.toml`:

```toml
[serve]
tls_cert = "certs/dev-cert.pem"   # certificado PEM (caminhos relativos partem da pasta do metagente.toml)
tls_key  = "certs/dev-key.pem"    # chave privada PEM
```

* Com um certificado e uma chave, toda porta do processo responde só por https: não há listener simples nem
  redirecionamento. Sem eles, o `serve` responde por http simples exatamente como na 0.1.2. Para ter os dois, inicie dois
  processos.
* As flags vencem o `metagente.toml` (um caminho relativo dado como flag parte da pasta atual), e o `serve` avisa em uma
  linha quando usou uma flag no lugar de uma configuração. Dar só um dos dois é um erro que nomeia o que falta.
* Um agente em `[secure] agents` **nunca** é servido por http simples: sem certificado e chave o `serve` se recusa a
  iniciar. Se o arquivo tem vários agentes e um deles é seguro, o processo inteiro responde só por https, e avisa.
  Um agente seguro não pode ser servido com `--mcp stdio`.
* Todo endereço no Agent Card é `https://`, qualquer que seja o cabeçalho de uma requisição. (Um agente simples, não
  seguro, atrás de um proxy reverso que termina o TLS e envia `X-Forwarded-Proto: https` também anuncia https.)
* O certificado e a chave são lidos quando o `serve` inicia. Editá-los no `metagente.toml` depois exige reinicialização,
  e o `serve` imprime uma linha dizendo isso. Um certificado expirado é recusado; um que expira em até 30 dias só avisa.
* O serviço fica no seu próprio computador, a menos que você adicione `--public`. O https protege o canal, não quem está
  chamando: ainda não existe login nesta versão.

#### Um certificado para desenvolvimento, e como confiar nele

```text
mkdir certs
openssl req -x509 -newkey rsa:2048 -nodes -days 365 -subj "/CN=localhost" -addext "subjectAltName=DNS:localhost,IP:127.0.0.1" -addext "basicConstraints=critical,CA:FALSE" -keyout certs/dev-key.pem -out certs/dev-cert.pem
```

O certificado deve nomear o host que as pessoas usam no endereço (`localhost` e `127.0.0.1` aqui), e deve dizer
`CA:FALSE`. O computador que o chama precisa então confiar nele, de uma de duas formas, em `[network]`:

1. **Preferida, `ca_file`**: `ca_file = "certs/dev-cert.pem"` confia nesse certificado (ou na autoridade que o assinou)
   e continua verificando todo o resto, inclusive o nome.
2. **Só para desenvolvimento, `allow_self_signed`**: pula as verificações de certificado. Todo `run`, `serve` e `check`
   imprime um aviso dizendo isso, e o arquivo do agente não pode silenciá-lo. `self_signed_hosts = ["127.0.0.1"]` limita
   a opção a esses hosts.

Confiar em um certificado e exigir https são coisas independentes: `allow_self_signed` nunca torna `http://` aceitável
para um nome em `[secure]`, e `[secure]` nunca liga os certificados autoassinados.

#### `[network]` e `[network.proxy]`: todas as chaves

```toml
[network]
allow_self_signed = false        # true: sem verificação de certificado para agentes, ferramentas, a ferramenta http e o proxy (só desenvolvimento)
self_signed_hosts = ["127.0.0.1"] # opcional: limita allow_self_signed a esses hosts
llm_allow_self_signed = false    # true: sem verificação de certificado SOMENTE para o modelo de linguagem (um gateway privado em que você confia)
ca_file = "certs/dev-ca.pem"     # confia em mais uma autoridade, para toda conexão, e continua verificando

[network.proxy]
url = "http://proxy.company.com:3128"   # sem nome de usuário nem senha dentro do endereço
username_env = "PROXY_USER"             # o NOME da variável que guarda o nome de usuário
password_env = "PROXY_PASSWORD"         # o NOME da variável que guarda a senha
no_proxy = ["localhost", "127.0.0.1", ".internal.company.com"]
pass_to_tools = false                   # true: repassa essas configurações aos programas de ferramentas iniciados por stdio
```

Todas as chaves são opcionais e todo padrão é desligado. `allow_self_signed` **não** vale para o modelo de linguagem,
porque a chave do seu modelo viaja nessa conexão: `llm_allow_self_signed` é uma chave separada, e imprime o próprio aviso
(a chave é enviada a um servidor cuja identidade não foi verificada). Um `ca_file` que não existe é um erro que nomeia o
arquivo e a linha.

* **Proxy.** As conexões passam pelo proxy (`CONNECT` para https, proxy simples para http), com login Basic quando
  `username_env` e `password_env` estão definidos. O login é **somente** os nomes das variáveis de ambiente: um
  `username` literal, um `password` literal, ou `user:pass@` dentro de `url` é rejeitado, porque o `metagente.toml`
  costuma ser compartilhado ou versionado. O login é enviado só ao proxy, nunca é impresso, e essas variáveis não podem
  ser lidas pelos agentes (`tool env`), exatamente como a chave do modelo.
* **Proxy pelo ambiente.** Sem `url`, são usadas as variáveis habituais `HTTPS_PROXY`, `HTTP_PROXY` e `NO_PROXY`
  (o `check` avisa uma vez). Com `url`, o toml vence e as variáveis de proxy do ambiente são ignoradas por completo,
  inclusive `NO_PROXY`.
* **`pass_to_tools = true`** entrega `HTTP_PROXY`, `HTTPS_PROXY` e `NO_PROXY` (e suas formas em minúsculas) aos servidores
  MCP iniciados por `stdio`, somente a esses programas filhos, login incluído. O `check` avisa que o login passa a ficar
  visível a esses programas, então declare apenas ferramentas em que você confia. Uma ferramenta `stdio` não faz conexão
  de rede pelo Metagente, então as configurações de proxy e de certificado não valem para esse canal.

#### O que você vê quando algo está errado

Toda mensagem nomeia a coisa, diz o que aconteceu e termina com a correção. Nenhuma mostra texto de Rust ou de
bibliotecas, uma chave ou uma senha. As mensagens aparecem em inglês, como o Metagente as imprime.

| Mensagem (formato) | Correção |
|--------------------|----------|
| `Bob is declared secure in metagente.toml but this address uses http: http://host:8443` | escreva `https://`, ou remova `Bob` de `[secure] agents` |
| `Researcher is declared secure, so it may only call https addresses. This address uses http: ...` | mude o endereço para que comece com `https://` |
| `The model address in [llm] uses http, and [secure] llm is on. Nothing was sent.` | use um endereço `https://` em `[llm]` |
| `Researcher is declared secure, so I will only serve it over https, but no certificate is set.` | defina `tls_cert` e `tls_key` em `[serve]`, ou use `--tls-cert` e `--tls-key` |
| `I have a certificate but no key (tls_key).` (ou o inverso) | adicione a configuração que falta |
| `Facts is declared secure and stdio is not https.` | sirva-o com `--mcp PORTA` e um certificado |
| `files runs as a local program, so it has no address to secure.` | remova-o de `[secure] tools`, ou dê à ferramenta um endereço `https://` |
| `the key in ... does not match the certificate in ...` | use a chave que foi criada com esse certificado |
| `the certificate in ... expired on ...` | crie um novo certificado e uma nova chave |
| `Warning: The certificate in ... expires on ...` | renove quando puder |
| `https://host/... sent me to http://host/..., and I will not follow it: an https address sent me to an http address` | sirva o agente por https, ou use o endereço final |
| `the agent card of Bob points tasks to ..., which is less secure than ...; I will not follow it` | sirva o agente por https, para que o card dele aponte para um endereço https |
| `the agent card of Bob points tasks to ..., a different host than ...; Bob is declared secure, so I will not follow it` | sirva o agente de modo que o card dele aponte para o próprio endereço |
| `I could not complete a secure connection to https://...` | confira a porta e se o servidor responde https; nunca há nova tentativa por http |
| `I could not trust the certificate of ...: it was signed by an authority I do not know, for example a self-signed certificate` | `ca_file` (preferido) ou, só para desenvolvimento, `allow_self_signed` |
| `the certificate of ... has expired` / `... is not valid yet` | renove-o / confira o relógio |
| `the certificate of ... is for X, but the address says Y` | use o nome que está no certificado |
| `the proxy ... refused the login` | confira `username_env` e `password_env`, e se as variáveis estão definidas |
| `I could not reach the proxy ...; ... was never contacted` | confira `url` em `[network.proxy]` |
| `` `password` in [network.proxy] must not hold a real login `` | escreva o nome de uma variável em `password_env` |
| `[secure] lists the agent Bobb, but no agent or remote called Bobb is declared here. Did you mean Bob?` | corrija o nome |

#### Teste com Python

O `cargo test` também verifica que um agente Python e um agente Metagente chamam um ao outro, por http simples, por
https com um certificado autoassinado, e por um proxy autenticado, usando o `a2a-sdk` oficial. Esses testes precisam do
Python 3.11 ou mais novo; sem ele, dizem que foram pulados (na CI, a falta do Python quebra o build):

```text
python3 -m venv .venv
. .venv/bin/activate            # Windows: .venv\Scripts\activate
pip install -r tests/interop/requirements.txt
export METAGENTE_INTEROP_PYTHON="$PWD/.venv/bin/python"   # Windows: set METAGENTE_INTEROP_PYTHON=.venv\Scripts\python.exe
cargo test
```

Para experimentar os dois lados à mão, veja [examples/python_interop](examples/python_interop/README.md).

### Novo na 0.1.2

Um prompt com muitas linhas, escrito como linhas normais. Ele chega ao modelo exatamente como digitado
([examples/multiline.ag](examples/multiline.ag)):

```text
agent Recruiter
  goal "Screen candidate resumes"
  accepts screen resume
  on screen
    reply think """You are a technical recruiter.
Read the resume below and list its three strongest points.

Resume:
{resume}"""
```

Um endereço e um prompt que mudam de um computador para outro, mantidos fora do agente. Eles são entradas de
`[parameters]` no `metagente.toml`, na pasta dos agentes ([examples/parameters.ag](examples/parameters.ag)):

```toml
[parameters]
a2a_leitor = "http://127.0.0.1:8080"
prompt1 = "you are a recruiter..."
```

```text
agent Screener
  goal "Screen a resume with the help of a remote reader"
  remote leitor at @parameters.a2a_leitor
  accepts screen candidate
  on screen
    resume = leitor.read candidate: candidate
    reply think @parameters.prompt1
```

Um agente carregado com `link` lê o `metagente.toml` do agente que o carregou. Os detalhes estão em
[docs/syntax.md](docs/syntax.md#parameters).

## Um exemplo completo

O [City Briefing](samples/city-briefing/README.md) junta tudo: um agente Concierge pergunta a um agente Researcher por
A2A, e o Researcher lê uma página web por uma ferramenta MCP. Os dois usam o Claude. Traga a sua própria chave de API.

## Saiba mais

A documentação em `docs/` e os READMEs dos exemplos estão em inglês.

| Tópico | Onde |
|--------|------|
| Tutorial, passo a passo | [docs/tutorial.md](docs/tutorial.md) |
| Guia de programação (if, laços, resultados) | [docs/guide.md](docs/guide.md) |
| Sintaxe da linguagem, incluindo texto com várias linhas e parâmetros (novo na 0.1.2) | [docs/syntax.md](docs/syntax.md) |
| Exemplos | [samples/](samples/) |
| Conexões seguras de ponta a ponta (novo na 0.1.3) | [multi-agent-samples/https-chain](multi-agent-samples/https-chain/README.md) |
| Guia de conexões seguras para iniciantes: o que é um certificado, como criar um autoassinado, como rodar a cadeia inteira por https (novo na 0.1.3) | [docs/secure-connections-guide.md](docs/secure-connections-guide.md) |
| Dois modelos em um agente: um modelo Anthropic rápido e um modelo OpenAI, quebre de propósito (novo na 0.1.5) | [multi-agent-samples/two-models](multi-agent-samples/two-models/README.md) |
| Tokens bearer de ponta a ponta: três agentes, um token, quebre de propósito (novo na 0.1.4) | [multi-agent-samples/auth-chain](multi-agent-samples/auth-chain/README.md) |
| Guia de autenticação: o que o Metagente faz e não faz, o validador, o cache, a lista de verificação (novo na 0.1.4) | [docs/authentication-guide.md](docs/authentication-guide.md) |
| **Guia de interoperabilidade:** escreva suas próprias ferramentas em Python, TypeScript, Go e Java, e conecte-as a um agente com uma linha; uma ferramenta que lê um banco de dados | [docs/mcp-tools-guide.md](docs/mcp-tools-guide.md) |
| O exemplo de ferramentas: a mesma ferramenta em quatro linguagens, os mesmos dois agentes, uma ferramenta de banco de dados | [multi-agent-samples/mcp-tools](multi-agent-samples/mcp-tools/README.md) |
| Histórico de mudanças | [CHANGELOG.md](CHANGELOG.md) |

## Roadmap

| Etapa | Estado |
|-------|--------|
| Linguagem central, ferramentas embutidas e MCP, CLI, tutorial | Pronto |
| Agentes seguros (permissões por agente) | Pronto |
| Link dinâmico entre agentes | Pronto |
| Texto com várias linhas e parâmetros externos (v0.1.2) | Pronto |
| Conexões seguras: https nas duas pontas, `[secure]`, proxy, interoperabilidade com Python (v0.1.3) | Pronto |
| Versão mínima e tokens bearer para mensagens servidas (v0.1.4) | Pronto |
| Vários modelos de linguagem para o `think`: `using NOME`, `[llm.NOME]` (v0.1.5) | Pronto, verificações ao vivo pendentes |
| v1.0: servidor MCP, A2A Agent Card, tarefas A2A, `serve` | Pronto, release pendente |
| **Projeto Barracuda:** um servidor de agentes invocado por A2A ou por um frontend, com uma fila para disparo em lote | Próximo |
| Tarefas de longa duração, tokens para chamadores sem supervisão, registro de agentes, agendamento, streaming A2A | Ideias |

## Como é construído

O Metagente é desenvolvido com desenvolvimento orientado por especificação, usando o [GitHub Spec Kit](https://github.com/github/spec-kit).
Toda funcionalidade começa como uma spec, então você pode ler por que as coisas são como são:

| Documento | Caminho |
|-----------|---------|
| Constituição (princípios) | `.specify/memory/constitution.md` |
| Spec da funcionalidade | `specs/001-metagente-core/spec.md` |
| Plano de implementação | `specs/001-metagente-core/plan.md` |
| Lista de tarefas | `specs/001-metagente-core/tasks.md` |
| Contratos de linguagem, CLI, configuração e protocolos | `specs/001-metagente-core/contracts/` |
| Guia de validação | `specs/001-metagente-core/quickstart.md` |
| Spec da v0.1.2 (texto com várias linhas e parâmetros) | `specs/004-multiline-text-and-parameters/spec.md` |
| Plano, tarefas e contratos da v0.1.2 | `specs/004-multiline-text-and-parameters/` |
| Spec da v0.1.3 (comunicação segura por declaração) | `specs/005-secure-by-declaration/spec.md` |
| Plano, tarefas, pesquisa e contratos da v0.1.3 | `specs/005-secure-by-declaration/` |
| Spec da v0.1.4 (versão mínima e autenticação de mensagens servidas) | `specs/006-version-and-served-auth/spec.md` |
| Plano, tarefas, pesquisa e contratos da v0.1.4 | `specs/006-version-and-served-auth/` |
| Spec da v0.1.5 (vários modelos de linguagem para o `think`) | `specs/007-multiple-llm-models/spec.md` |
| Plano, tarefas, pesquisa, contratos e as verificações ao vivo da v0.1.5 | `specs/007-multiple-llm-models/` |

## Contribuindo

Issues, ideias e pull requests são bem-vindos. Se você construir um agente com o Metagente, abra uma issue e mostre;
os bons viram exemplos.

Se o Metagente for útil para você, uma estrela no repositório ajuda outras pessoas a encontrá-lo.

## Versão

**v0.1.5.** Versionamento semântico. A versão é mantida neste arquivo, no `CHANGELOG.md` e no `Cargo.toml`.
