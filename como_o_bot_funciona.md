# Como o bot funciona — Documentação Técnica

---

## Visão geral do fluxo

Cada mensagem recebida percorre exatamente esta sequência:

```
1. Trigger (Telegram ou N8N Chat)
2. IF: é /start? → sim: envia boas-vindas e encerra
3. Buscar todas as linhas da base no Google Sheets
4. Montar contexto: detectar bug + construir prompt + serializar body
5. Chamar Groq API (Llama 3)
6. Extrair resposta + detectar fallback
7. IF: é fallback? → sim: logar no Sheets antes de responder
8. Enviar resposta ao cliente
```

---

## Comportamentos do bot

### 1. Resposta baseada em base de conhecimento

O bot **não tem conhecimento próprio sobre a Nola** — ele só sabe o que está na planilha. Isso é intencional: garante que as respostas sejam sempre precisas e controladas pelo time de CS.

A cada mensagem, o bot busca **todas as linhas** do Google Sheets e monta um contexto assim:

```
[Cardápio] Como cadastro um novo item? -> Acesse Cardápio > Produtos...
[Pedidos] Como cancelo um item? -> Dentro da comanda aberta...
```

Esse contexto é injetado no prompt do sistema antes de chamar a IA.

**Vantagem:** adicionar uma nova dúvida à base = adicionar uma linha na planilha. O bot já usa na próxima mensagem, sem restart, sem código.

---

### 2. Compreensão de linguagem natural

O bot usa um modelo de linguagem (Llama 3), não um sistema de palavras-chave. Isso significa que ele entende a **intenção** da pergunta, não o texto literal.

Exemplos de perguntas que mapeiam para a mesma resposta:
- `Como cadastro um produto?`
- `quero adicionar um prato novo`
- `komo boto um item no cardapio` (com erro de digitação)
- `meu cliente pediu uma sobremesa nova, como faço?`

Chatbots tradicionais exigiriam todas essas variações cadastradas. Aqui, uma única entrada na base é suficiente.

---

### 3. Detecção automática de bugs

Antes de chamar a IA, o código analisa a mensagem com uma expressão regular:

```javascript
const isBug = /erro|bug|não funciona|nao funciona|travou|travando|quebrou|caiu|parou/i.test(userMessage);
```

Se detectar qualquer uma dessas palavras, o bot **troca o prompt automaticamente** — em vez de buscar na base de conhecimento, ele responde com empatia, pede detalhes sobre qual tela apresentou o problema e informa que vai acionar o time técnico.

**Por que isso importa para CS:** o bot coleta informações úteis antes de escalonar, reduzindo o tempo de diagnóstico do time técnico.

---

### 4. Fallback inteligente com solicitação de contato

Quando o bot não encontra a resposta na base, o prompt instrui a IA a responder com uma frase específica contendo "não encontrei" ou "entrar em contato". O nó de extração detecta isso via regex:

```javascript
const isFallback = /não encontrei|nao encontrei|entrar em contato/i.test(answer);
```

Se `isFallback` for `true`, o bot:
1. Responde ao cliente pedindo nome e e-mail
2. Registra automaticamente a pergunta na aba "Dúvidas sem resposta" com data e canal

**Por que isso importa para CS:** nenhuma dúvida se perde. O time tem visibilidade total do que o bot não conseguiu resolver e pode usar isso para melhorar a base ou abrir tarefas.

---

### 5. Mensagem de boas-vindas (/start)

No Telegram, quando o usuário envia `/start`, um nó IF intercepta a mensagem **antes** de chegar à IA e responde com uma mensagem fixa de apresentação — listando o que o bot pode ajudar.

Isso não consome créditos de API e responde instantaneamente.

---

### 6. Log automático de dúvidas não respondidas

A aba "Dúvidas sem resposta" na planilha é alimentada automaticamente. Cada linha registra:

| data | canal | pergunta |
|---|---|---|
| 12/05/2026, 00:10:32 | Telegram | Qual o telefone do suporte? |

O time de CS pode revisar essa aba periodicamente para:
- Adicionar novas entradas à base de conhecimento
- Identificar dúvidas recorrentes que viram materiais de apoio
- Sinalizar gaps no produto para o time de tecnologia

---

## Decisões técnicas e cuidados

### Serialização do body com JSON.stringify

O maior desafio técnico foi enviar o contexto da base de conhecimento para a API do Groq. Como o contexto contém quebras de linha, aspas e caracteres especiais, colocá-lo diretamente em um JSON causava erros de parsing.

**Solução:** o body da requisição é montado como objeto JavaScript no nó Code e serializado com `JSON.stringify($json)` no nó HTTP Request. Isso garante que todos os caracteres especiais sejam escapados corretamente.

### Referência direta ao nó anterior após bifurcação

Quando o fluxo se bifurca (IF de fallback), o nó de envio recebe dados de branches diferentes. Para garantir que `chatId` e `answer` sempre estejam disponíveis, o nó Telegram referencia diretamente o nó de extração pelo nome:

```
{{ $('Extrair resposta (Telegram)').first().json.chatId }}
{{ $('Extrair resposta (Telegram)').first().json.answer }}
```

### Limpeza de markdown nas respostas

O modelo Llama 3 formata respostas com markdown (`**negrito**`, `*itálico*`). O Telegram interpreta esses caracteres de forma restrita e retorna erro se a formatação estiver incompleta. A solução foi remover o markdown antes de enviar:

```javascript
const answer = rawAnswer
  .replace(/\*\*/g, '')
  .replace(/\*/g, '')
  .replace(/_{1,2}/g, '')
  .replace(/#{1,6} /g, '');
```

### Escolha do modelo

Foram testados dois modelos via Groq:
- `llama-3.3-70b-versatile` — qualidade alta, ~3-5 segundos de resposta
- `llama3-8b-8192` — qualidade suficiente para FAQ, ~0.5-1 segundo de resposta

Para suporte baseado em base de conhecimento, o modelo menor tem qualidade equivalente com velocidade muito superior. O `llama3-8b-8192` foi mantido como padrão.

### Temperature 0.3

A temperature controla a criatividade do modelo. Valor baixo (0.3) significa respostas mais consistentes e previsíveis — ideal para suporte técnico onde precisão é mais importante que criatividade.

---

## Limitações conhecidas

- **Sem memória de conversa:** cada mensagem é tratada de forma independente. O bot não lembra o que foi dito anteriormente na mesma conversa.
- **Base carregada a cada mensagem:** todas as linhas do Sheets são buscadas a cada pergunta. Para bases muito grandes (500+ linhas), pode haver lentidão.
- **Detecção de fallback por texto:** a detecção de "não sei responder" depende de a IA usar as frases previstas no prompt. Em raros casos pode não detectar corretamente.
