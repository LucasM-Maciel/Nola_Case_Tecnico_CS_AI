# Desafio Prático — Estagiário de Customer Success
**Candidata:** Lucia  
**Empresa:** Nola — Gestão de Restaurantes  
**Data de entrega:** 12 de maio de 2026

---

## 1. Prompt utilizado no bot

O bot utiliza dois prompts distintos, selecionados dinamicamente com base na mensagem do cliente:

**Prompt padrão (dúvidas gerais):**
> "Você é a Nola, assistente virtual da plataforma Nola de gestão de restaurantes. Responda de forma clara e amigável. Use APENAS a base de conhecimento abaixo. Se não souber responder, diga: 'Não encontrei essa informação na minha base. Para te ajudar melhor, pode me informar seu nome e e-mail? Nossa equipe de CS entrará em contato em até 2 horas.' BASE: [conteúdo do Google Sheets]"

**Prompt para bugs/erros técnicos** (ativado quando o cliente menciona "erro", "bug", "travou", "não funciona" etc.):
> "Você é a Nola, assistente da plataforma Nola. O cliente está com problema técnico. Responda com empatia, peça qual tela apresentou o erro e informe que encaminhará ao time técnico."

**Parâmetros de geração:**
- Modelo: `llama3-8b-8192` (via Groq API)
- Temperature: `0.3` (respostas mais consistentes e menos criativas)
- Max tokens: `300`

---

## 2. Processo de construção

### Ferramentas utilizadas
- **N8N Cloud** — orquestração do fluxo de automação
- **Google Sheets** — base de conhecimento editável
- **Groq API** (modelo Llama 3 8B) — inteligência artificial para geração de respostas
- **Telegram Bot API** — canal de atendimento via mensagens
- **Claude (Anthropic)** — apoio na arquitetura, escrita de código e resolução de problemas

### Etapas do desenvolvimento

**1. Estruturação da base de conhecimento**  
Criação de uma planilha no Google Sheets com 35+ registros organizados em colunas: `categoria`, `pergunta` e `resposta`. As categorias cobrem: Cardápio, Pedidos, Financeiro, Mesas, Delivery, Estoque, Usuários, Relatórios, Suporte e Novidades.

**2. Configuração do N8N Chat**  
Primeiro workflow com o trigger nativo de chat do N8N, permitindo um chat widget embarcado sem necessidade de contas externas.

**3. Integração com Google Sheets**  
Uso do nó nativo Google Sheets do N8N (OAuth2) para buscar todas as linhas da base de conhecimento a cada mensagem recebida, garantindo que qualquer atualização na planilha seja refletida imediatamente no bot.

**4. Integração com Groq via HTTP Request**  
Chamada à API da Groq com o contexto da base de conhecimento + pergunta do usuário montados dinamicamente via nó Code (JavaScript). O corpo da requisição é serializado com `JSON.stringify` para evitar problemas de caracteres especiais.

**5. Canal Telegram**  
Criação de bot no BotFather e configuração de segundo workflow com Telegram Trigger. Adição de fluxo de boas-vindas com `/start` e limpeza de formatação markdown nas respostas.

**6. Funcionalidades avançadas**
- Detecção automática de relatos de bug por regex nas mensagens
- Fallback inteligente solicitando dados de contato quando o bot não sabe responder
- Log automático de dúvidas não respondidas em aba separada do Google Sheets para análise pelo time de CS

---

## 3. Como essa solução ajuda no dia a dia de CS

### O problema
Times de Customer Success em empresas de tecnologia recebem alto volume de chamados repetitivos — dúvidas sobre funcionalidades básicas que consomem tempo da equipe e atrasam o atendimento de casos mais complexos.

### A solução
O chatbot atua como primeira linha de suporte (nível 1), resolvendo automaticamente as dúvidas mais frequentes 24 horas por dia, 7 dias por semana, sem intervenção humana.

### Impactos diretos no CS

**Redução de esforço operacional**  
Dúvidas recorrentes (cardápio, pedidos, financeiro) são resolvidas instantaneamente, liberando o time para focar em atividades de maior valor: onboarding, relacionamento e sucesso do cliente.

**Escalonamento inteligente**  
Bugs e erros técnicos são identificados automaticamente e tratados com empatia, coletando informações relevantes antes de acionar o time técnico — reduzindo o tempo de diagnóstico.

**Base de conhecimento viva**  
A integração com Google Sheets permite que qualquer membro do time de CS atualize, adicione ou corrija informações sem tocar no código ou no N8N. Adicionar um novo tópico = adicionar uma linha na planilha.

**Inteligência de produto**  
A aba "Dúvidas sem resposta" registra automaticamente perguntas que o bot não conseguiu resolver. Isso vira input direto para:
- Melhorar a base de conhecimento
- Identificar gaps no produto
- Priorizar treinamentos e materiais de apoio

**Experiência do cliente**  
A IA entende linguagem natural — o cliente pode perguntar de qualquer forma, com erros de digitação ou palavras diferentes, e receber a resposta correta. Isso elimina a frustração do "não entendi sua mensagem" dos chatbots tradicionais.

### Aplicações além do suporte
- **Onboarding:** Bot guia novos clientes nas primeiras configurações
- **Lançamento de features:** Inserir novidades na planilha e o bot já comunica aos clientes
- **Treinamento interno:** Nova equipe pode consultar o bot para aprender sobre o produto

---

## 4. Ferramentas de IA utilizadas como apoio

**Claude (Anthropic) — principal**  
Utilizado como mentor técnico ao longo de todo o processo. Apoiou em: arquitetura da solução, escrita e depuração de código JavaScript, resolução de erros do N8N em tempo real, elaboração dos prompts do bot e estruturação da base de conhecimento.

**Groq + Llama 3 (IA do produto)**  
Modelo de linguagem que alimenta o chatbot. Escolhido pela alta velocidade de inferência (respostas em menos de 1 segundo) e disponibilidade gratuita via API — ideal para prototipagem e demonstração.

**Cursor (IDE com IA integrada)**  
Utilizado como ambiente de desenvolvimento e para ter uma visão geral do projeto de forma estruturada. O agente de IA do Cursor ajudou a organizar o raciocínio sobre a arquitetura da solução, documentar o projeto e manter clareza sobre as etapas e decisões tomadas ao longo do desenvolvimento.
