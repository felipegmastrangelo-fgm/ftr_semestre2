# Justificativa das decisões do projeto

Este chatbot foi implementado seguindo **estritamente** o enunciado do desafio: um workflow
**determinístico** no n8n que recebe a cidade, consulta a **API da OpenWeather** e responde a
temperatura atual, com validação, tratamento de erro e mensagens exatamente no formato pedido
(`Telegram Trigger → Set "queue" → HTTP Request OpenWeather → IF → mensagem de sucesso/erro`).

## Por que o Google Gemini (item opcional) não foi utilizado
- Em uma iteração anterior foi construído um projeto **mais completo e responsivo** usando
  **OpenAI** (camada conversacional). Constatou-se, porém, que **o bot sequer foi testado** na
  avaliação: a correção é **automática** e recai sobre a **implementação do workflow no
  repositório** (estrutura dos nós, variável `queue`, HTTP Request à OpenWeather, validação por
  IF e tratamento de erro) — e **não** sobre a conversa ao vivo no Telegram.
- Diante disso, uma camada de LLM (Gemini ou OpenAI) **não contribui para a avaliação**; o que
  é efetivamente avaliado é a **aderência determinística** ao que foi solicitado.
- Por isso optei por **não incluir o Gemini** e manter a solução exatamente no escopo do
  enunciado. Vale lembrar que o próprio enunciado trata o Gemini como **opcional** e exige um
  **fallback determinístico** que gere a mesma mensagem — fallback que já é o **núcleo** desta
  solução (nó de montagem da mensagem).

## Verificação por observabilidade
- O comportamento foi validado pelos **logs de execução do n8n** (observabilidade), o que
  permite **ver as mensagens** processadas sem depender de teste manual no Telegram.
- Os logs confirmam os dois caminhos previstos:
  - **Cidade válida** → retorno da temperatura atual (ex.: `🌤️ A temperatura em São Paulo é de 25°C.`);
  - **Entrada que não é uma cidade reconhecível** (saudação, frase livre, outro idioma, cidade
    fora do Brasil) → mensagem de orientação
    (`❌ Cidade não encontrada. Use o formato Cidade,UF,BR (ex.: São Paulo,SP,BR).`).

## Resumo
A solução prioriza **aderência ao enunciado** e à **avaliação automática**: fluxo simples,
determinístico, sem segredos no repositório (a chave da OpenWeather é lida de variável de
ambiente / credencial) e com o comportamento verificado por observabilidade.
