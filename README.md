# Chatbot de Clima no Telegram (n8n + OpenWeather)

Chatbot no Telegram, construído em **n8n**, que informa a **temperatura atual** de qualquer
cidade do Brasil. O usuário envia o nome da cidade e o estado; o fluxo consulta a **API
gratuita da OpenWeather**, processa a resposta e devolve uma mensagem curta, clara e amigável.

Bot: **@FelipeGMastrangelo_WeatherBot**

Exemplo de retorno:
> 🌤️ A temperatura em Belo Horizonte é de 25°C.

Em caso de cidade não encontrada:
> ❌ Cidade não encontrada. Use o formato Cidade,UF,BR (ex.: São Paulo,SP,BR).

> **Observação sobre o nome do arquivo:** o material do curso cita o export com dois nomes
> diferentes (`workflow-chatbot-telegram.json` e `workflow-telegram-chatbot.json`). Para não
> haver ambiguidade, **os dois arquivos idênticos estão neste repositório**.

---

## Estrutura do workflow

`Telegram Trigger → Capturar Entrada (Set) → Normalizar Entrada (Code) → OpenWeather (HTTP) → Resposta válida? (IF) → Montar Mensagem → Enviar (Telegram)`

| Nó | Tipo | Função |
|----|------|--------|
| **Telegram Trigger** | Telegram Trigger | Recebe mensagens de texto do bot. |
| **Capturar Entrada (Set)** | Set | Captura o texto recebido na variável **`queue`**. |
| **Normalizar Entrada** | Code | Normaliza a `queue`: `trim()`, remove espaços extras, tira acentuação, `toLowerCase()` e garante o sufixo `,br`. |
| **OpenWeather** | HTTP Request (GET) | Consulta `https://api.openweathermap.org/data/2.5/weather` com **Send Query Parameters** ativado: `q={{ $json.queue }}`, `units=metric`, `lang=pt_br`, `appid={{ $env.OPENWEATHER_API_KEY }}`. |
| **Resposta válida?** | IF | Valida o **status** (`cod = 200`) e o **campo esperado** (`main.temp`). |
| **Montar Mensagem** | Code | Extrai `main.temp`, arredonda e monta a string final da resposta. |
| **Enviar Sucesso / Enviar Erro** | Telegram | Send Message com a resposta ou a mensagem de erro. |

**Tratamento de erros:** o nó HTTP usa *Continue (using error output)* — qualquer falha de
rede ou HTTP (ex.: cidade inexistente → 404) é roteada direto ao nó **Enviar Erro**. Respostas
que voltam 200 mas sem `main.temp` também caem no erro pelo nó **Resposta válida?**.

---

## Pré-requisitos

- Uma instância do **n8n** (nuvem ou local — ver `docker-compose.yml`).
- Um **bot do Telegram** criado no [@BotFather](https://t.me/BotFather) (comando `/newbot`).
- Uma **conta gratuita na OpenWeather** e uma **API Key**
  (https://home.openweathermap.org/api_keys).

---

## Como importar o workflow no n8n

1. No n8n, clique em **Workflows → Import from File**.
2. Selecione o arquivo **`workflow-chatbot-telegram.json`** (ou o idêntico `workflow-telegram-chatbot.json`).
3. O fluxo será criado com os 8 nós já conectados.

## Como inserir as credenciais

Veja também o arquivo **`.env.example`** com as variáveis esperadas.

### 1. Telegram (`TELEGRAM_BOT_TOKEN`)
- No n8n, vá em **Credentials → New → Telegram API**.
- Cole o **token** do seu bot (fornecido pelo BotFather).
- Abra os nós **Telegram Trigger**, **Enviar Sucesso** e **Enviar Erro** e selecione
  essa credencial no campo *Credential to connect with*.

### 2. OpenWeather (`OPENWEATHER_API_KEY`)
A chave da OpenWeather é lida como **variável de ambiente**, não fica no JSON.
Defina a variável no ambiente onde o n8n roda:

```bash
# Linux/Mac
export OPENWEATHER_API_KEY="sua_chave_aqui"

# Docker (docker-compose.yml)
environment:
  - OPENWEATHER_API_KEY=sua_chave_aqui
```

> ⚠️ Por padrão o n8n só libera variáveis de ambiente para expressões se
> `N8N_BLOCK_ENV_ACCESS_IN_NODE=false` (já incluído no `docker-compose.yml`).

### Variáveis esperadas
| Variável | Onde |
|----------|------|
| `TELEGRAM_BOT_TOKEN` | Credencial Telegram API no n8n |
| `OPENWEATHER_API_KEY` | Variável de ambiente do n8n |

---

## Como executar e testar

1. Ative o workflow (toggle **Active**) — ou clique em **Execute Workflow** para testes.
2. No Telegram, abra o seu bot e envie o nome de uma cidade.

Testes sugeridos (checklist do desafio):

| Envie | Retorno esperado |
|-------|------------------|
| `São Paulo,SP,BR` | 🌤️ A temperatura em São Paulo é de X°C. |
| `Belo Horizonte,MG,BR` | 🌤️ A temperatura em Belo Horizonte é de X°C. |
| `Recife,PE,BR` | 🌤️ A temperatura em Recife é de X°C. |
| `Cidadeinexistente123` | ❌ Cidade não encontrada. Use o formato Cidade,UF,BR (ex.: São Paulo,SP,BR). |

O bot também aceita só o nome da cidade (ex.: `Bauru`) — o fluxo completa com `,br`.

---

## Comportamento esperado (por design)

Este chatbot é **focado**: responde à **temperatura de cidades do Brasil** quando recebe o
nome da cidade (idealmente no formato `Cidade,UF,BR`). Qualquer mensagem que **não seja uma
cidade reconhecível** (saudações, frases livres, outro idioma ou cidade fora do Brasil)
recebe a orientação:

> ❌ Cidade não encontrada. Use o formato Cidade,UF,BR (ex.: São Paulo,SP,BR).

Isso é **intencional** e segue o enunciado do desafio: a mensagem serve para **orientar** o
usuário a informar a cidade no formato correto — não é uma falha do bot.

---

## (Opcional) Rodar o n8n com Docker

Use o `docker-compose.yml` incluído:

```bash
export OPENWEATHER_API_KEY="sua_chave_aqui"
docker compose up -d
# n8n disponível em http://localhost:5678
```

---

## Segurança (verificação obrigatória antes de subir)

- ✅ O `workflow-chatbot-telegram.json` **não contém tokens nem chaves hardcoded** — a chave
  da OpenWeather entra por `={{ $env.OPENWEATHER_API_KEY }}` e o token do Telegram fica na
  credencial do n8n.
- ✅ O `.env.example` documenta as variáveis **sem** valores reais.
- ✅ Antes do commit, confira que nenhum arquivo tem segredo real.
- ✅ O repositório deve ser **público** (`chatbot-telegram`) para a avaliação automática.

---

_Projeto acadêmico de pós-graduação em Automação — Felipe Mastrangelo._
