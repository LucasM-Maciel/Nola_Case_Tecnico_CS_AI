# Nola CS AI — Chatbot de Suporte com IA

Desafio prático para a vaga de Estagiário de Customer Success na **Nola**, plataforma de gestão de restaurantes.

## O que foi construído

Um chatbot de suporte ao cliente funcional, com IA, integrado a uma base de conhecimento no Google Sheets e disponível em dois canais:

- **N8N Chat** — widget de chat embarcado
- **Telegram** — bot (@nola_suporte_bot)

## Arquitetura

```
[Cliente envia mensagem]
         ↓
[Telegram Trigger / N8N Chat Trigger]
         ↓
[IF: /start?] → Sim → [Mensagem de boas-vindas]
         ↓ Não
[Google Sheets — buscar base de conhecimento]
         ↓
[Code JS — detectar bug, montar prompt + contexto]
         ↓
[Groq API — Llama 3 gera resposta]
         ↓
[Code JS — extrair resposta, detectar fallback]
         ↓
[IF: não sabe responder?]
   → Sim → [Google Sheets — logar em "Dúvidas sem resposta"] → [Enviar resposta]
   → Não → [Enviar resposta]
```

## Funcionalidades

| Funcionalidade | Descrição |
|---|---|
| Respostas com IA | Entende linguagem natural, não exige pergunta exata |
| Base editável | Qualquer atualização no Sheets reflete imediatamente |
| Detecção de bugs | Identifica relatos técnicos e trata com fluxo especial |
| Fallback inteligente | Quando não sabe, pede contato para acionar CS humano |
| Log de dúvidas | Registra perguntas sem resposta para melhoria contínua |
| Boas-vindas | Mensagem de apresentação no primeiro acesso (/start) |
| Multi-canal | Funciona no chat web e no Telegram simultaneamente |

## Stack

- **N8N Cloud** — orquestração do fluxo
- **Google Sheets** — base de conhecimento
- **Groq API** — inferência do modelo Llama 3 8B
- **Telegram Bot API** — canal de mensagens
- **Cursor** — IDE com IA para arquitetura, desenvolvimento e documentação

## Base de conhecimento

A planilha possui 35+ registros em categorias:
- Cardápio
- Pedidos e Comandas
- Financeiro
- Mesas e Reservas
- Delivery e Integrações
- Estoque
- Usuários
- Relatórios
- Suporte Técnico
- Novidades

[Acessar planilha](https://docs.google.com/spreadsheets/d/1GAhyWnlgJ-8nI-Msbh5Veebc-Z665g--Chzuue5BBqg)

## Como usar o bot

### Canal 1 — Telegram

1. Abra o Telegram e busque pelo bot **@nola_suporte_bot**
2. Clique em **"Iniciar"** ou mande `/start` para ver a mensagem de boas-vindas
3. Digite sua dúvida em linguagem natural — não precisa ser a pergunta exata
4. O bot responde em segundos com base na base de conhecimento

**Exemplos de perguntas:**
- `Como abro uma comanda?`
- `quero adicionar um produto novo no cardápio`
- `o sistema travou na tela de pedidos` ← ativa fluxo especial de bug
- `qual o horário do suporte?` ← fallback com pedido de contato

### Canal 2 — N8N Chat (widget web)

1. Acesse o workflow no N8N Cloud
2. Clique em **"Open chat"** no rodapé da tela
3. Interaja diretamente pelo widget

### Como atualizar a base de conhecimento

1. Abra a [planilha no Google Sheets](https://docs.google.com/spreadsheets/d/1GAhyWnlgJ-8nI-Msbh5Veebc-Z665g--Chzuue5BBqg)
2. Adicione uma nova linha com `categoria`, `pergunta` e `resposta`
3. Salve — o bot já usa as novas informações na próxima mensagem (sem precisar reiniciar nada)

### Como visualizar dúvidas não respondidas

Na mesma planilha, acesse a aba **"Dúvidas sem resposta"** — lá ficam registradas automaticamente todas as perguntas que o bot não conseguiu responder, com data, canal e texto da pergunta.

---

## Importar o workflow no N8N

O arquivo `Nola_Workflow.json` neste repositório contém o workflow completo exportado do N8N.

Para importar:
1. Acesse seu N8N Cloud
2. Clique em **"+"** para novo workflow
3. Clique nos **três pontos (⋯)** → **"Import from file"**
4. Selecione o arquivo `Nola_Workflow.json`
5. Configure as credenciais (Google Sheets, Telegram, Groq) com seus próprios tokens

---

## Estrutura do repositório

```
├── Nola_Workflow.json          # Workflow exportado do N8N (importável)
├── Nola_Case_Técnico.xlsx      # Base de conhecimento exportada do Google Sheets
├── desafio.md                  # Descrição do desafio proposto pela Nola
├── como_o_bot_funciona.md      # Documentação técnica detalhada do bot
├── documento_entrega.md        # Documentação completa para entrega do desafio
└── README.md                   # Este arquivo
```

---

## Documentação completa

- [documento_entrega.md](./documento_entrega.md) — prompt, processo de construção, impacto para CS e ferramentas utilizadas
- [como_o_bot_funciona.md](./como_o_bot_funciona.md) — documentação técnica detalhada: comportamentos, decisões de arquitetura, cuidados e limitações
