# Exercício 2.1 — Workflow de Desenvolvimento AI-First
**NovaTech Assistant | Entregável do Delivery Manager**

---

## 1. Fluxo de Trabalho AI-First por Papel

A tabela abaixo mapeia cada papel às ferramentas de IA permitidas em cada etapa do ciclo. Células sem ferramenta indicam execução humana pura (decisão ou aprovação).

| Etapa | Tech Lead | Dev Pleno | Dev Sênior | QA | Product Specialist | Delivery Manager |
|---|---|---|---|---|---|---|
| **Spec** | Claude (chat) — revisão técnica da spec, validação de ADRs | Claude (chat) — leitura e dúvidas | Claude (chat) — leitura e dúvidas | Claude (chat) — extração de cenários de teste a partir da spec | Claude (chat) + Claude Design (protótipos) + Claude Cowork (estruturação de stories) | Claude Cowork (consolidação, riscos, comunicação com stakeholders) |
| **Plan** | Claude (chat) — geração de plano técnico e breakdown arquitetural | Claude (chat) — refinamento do plano com foco na sua área | Claude (chat) — refinamento do plano com foco na sua área | Claude Cowork — estruturação do plano de testes | Claude Cowork — validação de critérios de aceite e escopo | Claude Cowork — plano de projeto, cronograma, mapa de riscos |
| **Tasks** | Claude (chat) — decomposição técnica; atualização no Azure DevOps | GitHub Copilot — geração de subtarefas e estimativas técnicas | GitHub Copilot — geração de subtarefas e estimativas técnicas | Claude Cowork — geração de casos de teste vinculados às tasks | Claude Cowork — detalhamento de user stories, critérios de aceite | Claude Cowork — priorização, distribuição de carga, Azure DevOps boards |
| **Implement** | GitHub Copilot (review inline) + Claude (chat) — decisões arquiteturais pontuais | GitHub Copilot — geração e completação de código TypeScript/React | GitHub Copilot — geração e completação de código TypeScript/React/Bicep | Claude (chat) — geração de scripts de teste (unit/integration) | Claude Design — refinamento de protótipos e especificações visuais | Claude (chat) — acompanhamento de progresso, comunicados |
| **Review** | GitHub Copilot + Claude (chat) — code review, validação de padrões da stack | GitHub Copilot — revisão de par (peer review) | GitHub Copilot — revisão de par (peer review) | Claude Cowork — análise de cobertura de testes, report de gaps | Claude (chat) — validação de aderência funcional à spec original | Claude Cowork — status report, métricas de velocidade, comunicação |
| **Deploy** | Claude (chat) — revisão dos scripts Bicep, checklist de infra | GitHub Copilot — suporte em scripts de automação | GitHub Copilot — suporte em scripts de automação | Claude Cowork — sign-off de qualidade (gate final) | Claude Cowork — validação dos critérios de aceite pós-deploy | Claude Cowork — comunicado de release, registro no Azure DevOps |

---

## 2. Template de Checklist — Validation Gates

```
╔══════════════════════════════════════════════════════════════════════╗
║         NOVATECH ASSISTANT — VALIDATION GATE CHECKLIST              ║
║         Projeto: novatech-assistant   |   Data: ___/___/______       ║
║         Feature/Sprint: _______________   Responsável: _____________ ║
╚══════════════════════════════════════════════════════════════════════╝

┌──────────────────────────────────────────────────────────────────────┐
│ GATE 1 — SPEC APROVADA (antes de gerar o Plan)                       │
├──────────────────────────────────────────────────────────────────────┤
│ □ Spec alinhada com ADR-0001 (Azure OpenAI GPT-4o como LLM)          │
│ □ Requisitos não ultrapassam o budget de contexto definido           │
│   (system prompt ≤ 4K tokens, chunks ≤ 8K, histórico ≤ 3 turnos)    │
│ □ Casos que envolvem documentos contraditórios explicitam o          │
│   comportamento esperado (priorizar versão mais recente via          │
│   metadado de vigência)                                              │
│ □ Os 12 documentos com contradições pendentes no Compliance NÃO      │
│   estão referenciados como fonte primária nesta feature              │
│ □ Critérios de aceite definidos e mensuráveis                        │
│ □ Product Specialist e Tech Lead assinaram                           │
│                                                                      │
│ Resultado: □ APROVADO  □ REPROVADO    Data: ______  Hora: ______     │
│ Assinatura: _________________________                                 │
└──────────────────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────────────────┐
│ GATE 2 — TASKS VALIDADAS (antes de iniciar a Implementação)          │
├──────────────────────────────────────────────────────────────────────┤
│ □ Todas as tasks registradas no Azure DevOps com responsável e       │
│   estimativa                                                         │
│ □ Tasks de backend especificam TypeScript e target (Azure Functions  │
│   ou pipeline de ingestão)                                           │
│ □ Tasks de frontend especificam React                                │
│ □ Tasks de infra especificam Bicep                                   │
│ □ Tasks que tocam o pipeline de RAG referenciam explicitamente       │
│   a estratégia de chunking (5 chunks de ~1.500 tokens)               │
│ □ Nenhuma task depende de tecnologia fora da stack aprovada          │
│ □ Tech Lead assinou                                                  │
│                                                                      │
│ Resultado: □ APROVADO  □ REPROVADO    Data: ______  Hora: ______     │
│ Assinatura: _________________________                                 │
└──────────────────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────────────────┐
│ GATE 3 — CÓDIGO APROVADO PARA MERGE (após Review, antes do merge)    │
├──────────────────────────────────────────────────────────────────────┤
│ □ Código em TypeScript (backend/bot) ou React (frontend)             │
│   sem desvios de linguagem/framework                                 │
│ □ System prompt final não excede 4.000 tokens (verificação manual    │
│   ou script de contagem)                                             │
│ □ Chamadas ao Azure AI Search retornam no máximo 5 chunks            │
│ □ Lógica de resolução de documentos contraditórios presente:         │
│   ordenação por metadado de vigência e instrução no prompt           │
│ □ Nenhuma chamada direta a documentos marcados como obsoletos        │
│ □ Histórico de conversa truncado a 3 turnos                          │
│ □ Pelo menos 1 aprovação de par (Dev Sênior ou Tech Lead)            │
│ □ Tech Lead assinou como aprovação final                             │
│                                                                      │
│ Resultado: □ APROVADO  □ REPROVADO    Data: ______  Hora: ______     │
│ Assinatura: _________________________                                 │
└──────────────────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────────────────┐
│ GATE 4 — TESTES SUFICIENTES PARA DEPLOY                              │
├──────────────────────────────────────────────────────────────────────┤
│ □ Há testes cobrindo o cenário de documentos contraditórios          │
│   (resposta prioriza documento com vigência mais recente)            │
│ □ Há testes de limite de contexto: request acima do budget           │
│   é tratado corretamente (truncamento ou erro explícito)             │
│ □ Testes de integração com Azure AI Search validam retorno           │
│   de exatamente até 5 chunks                                         │
│ □ Fluxo do Teams bot testado end-to-end (ao menos caminho feliz)     │
│ □ Scripts Bicep validados em ambiente de staging (não apenas lint)   │
│ □ Nenhum teste depende de um dos 12 documentos com contradição       │
│   pendente como oráculo de resposta esperada                         │
│ □ QA assinou o sign-off de qualidade                                 │
│ □ Tech Lead ratificou suficiência técnica dos testes                 │
│                                                                      │
│ Resultado: □ APROVADO  □ REPROVADO    Data: ______  Hora: ______     │
│ Assinatura: _________________________                                 │
└──────────────────────────────────────────────────────────────────────┘
```

---

## 3. Detalhamento dos Gates

### Gate 1 — Spec aprovada (Spec → Plan)

| Dimensão | Definição |
|---|---|
| **Quem aprova** | Product Specialist (titular) + Tech Lead (co-aprovação obrigatória) |
| **O que verifica** | (1) Budget de contexto não é violado conceitualmente pelos requisitos descritos. (2) Qualquer feature que consulte a base documental menciona explicitamente o comportamento esperado para os 12 documentos em disputa no Compliance. (3) Critérios de aceite são mensuráveis e não ambíguos. (4) Nenhuma integração fora do escopo (Teams bot + painel web) é introduzida. |
| **Tempo limite** | **24 horas** após entrega da spec ao time. |
| **Se reprovar** | Product Specialist registra os pontos de falha no Azure DevOps (item de trabalho vinculado à spec). Spec retorna ao autor com comentários. Novo ciclo de revisão com prazo de 8h para correção. Gate 1 é reexecutado antes de qualquer atividade de planejamento. |

**Protocolo de desempate (PS aprova, TL reprova — ou vice-versa):** O Gate 1 exige co-aprovação, portanto divergência entre PS e TL constitui reprovação automática do gate. Quando isso ocorre, o Delivery Manager convoca uma sessão de alinhamento (máximo 1 hora, síncrona) com os dois papéis dentro do mesmo prazo de 24 horas. O PS é titular dos critérios de negócio; o TL é titular dos critérios técnicos — cada um tem veto soberano dentro do seu domínio. Se o PS aprova o escopo funcional mas o TL identifica violação técnica (ex.: requisito incompatível com o budget de tokens), a spec retorna ao PS com as restrições técnicas documentadas para ajuste. Se o TL aprova a viabilidade técnica mas o PS identifica desvio de valor de negócio, a spec retorna ao autor com os critérios de aceite revisados. Em nenhum caso o Delivery Manager desempata sozinho: a decisão final pertence ao dono do domínio em conflito. O histórico da divergência e da resolução é registrado no item do Azure DevOps para auditoria.

---

### Gate 2 — Tasks validadas (Tasks → Implement)

| Dimensão | Definição |
|---|---|
| **Quem aprova** | Tech Lead |
| **O que verifica** | (1) Toda task de código identifica a linguagem/framework correto (TypeScript, React ou Bicep). (2) Tasks que alteram o pipeline de RAG referenciam os parâmetros de chunking aprovados (5 chunks × 1.500 tokens). (3) Não há task com dependência técnica não mapeada (ex: serviço Azure não provisionado). (4) Estimativas presentes e razoáveis para o porte do time (2 devs). (5) Tasks registradas no Azure DevOps com assignee definido. |
| **Tempo limite** | **4 horas** após geração das tasks pela IA. |
| **Se reprovar** | Tech Lead marca tasks problemáticas com a tag `blocked-gate2` no Azure DevOps e descreve o motivo. Dev responsável ajusta e solicita nova validação. Implementação não se inicia até aprovação total. SLA da correção: 2h. |

---

### Gate 3 — Código aprovado para merge (Review → Deploy)

| Dimensão | Definição |
|---|---|
| **Quem aprova** | Tech Lead (aprovação final) + ao menos 1 Dev (revisão de par) |
| **O que verifica** | (1) **Limite de tokens do system prompt:** contagem ≤ 4.000 tokens — verificada por script ou inspeção direta do template de prompt no repositório. (2) **Chunks:** lógica de recuperação solicita no máximo 5 resultados ao Azure AI Search. (3) **Documentos contraditórios:** código aplica ordenação por metadado de vigência antes de montar o contexto; prompt instrui o modelo a priorizar a versão mais recente. (4) **Histórico:** janela de histórico limitada a 3 turnos no código de gerenciamento de sessão. (5) **Stack:** nenhum import de biblioteca fora da stack aprovada (TypeScript/Node, React, Bicep, Azure SDK). |
| **Tempo limite** | **8 horas** após abertura do PR. |
| **Se reprovar** | PR recebe label `needs-revision` e comentários inline apontando exatamente qual critério falhou e o valor encontrado vs. esperado. Dev corrige, força novo push na mesma branch e solicita re-review. Gate 3 é reexecutado integralmente. |

---

### Gate 4 — Testes suficientes para deploy (Testes → Deploy)

| Dimensão | Definição |
|---|---|
| **Quem aprova** | QA (sign-off de qualidade) + Tech Lead (ratificação técnica) |
| **O que verifica** | (1) **Documentos contraditórios:** existe ao menos 1 teste automatizado que simula dois documentos com mesmo tema e vigências diferentes e verifica que a resposta prioriza o mais recente. (2) **Budget de contexto:** existe teste de boundary que envia um payload acima do limite e verifica tratamento adequado (erro explícito ou truncamento controlado). (3) **Chunks:** teste de integração com Azure AI Search confirma que o número de chunks retornados não excede 5. (4) **Teams bot:** fluxo completo (mensagem recebida → consulta RAG → resposta entregue no Teams) executado em staging. (5) **Bicep:** templates executados com sucesso em ambiente de staging, não apenas validados com lint. (6) Os 12 documentos com contradição pendente não são usados como "ground truth" em nenhum teste (risco de falso positivo/negativo). |
| **Tempo limite** | **12 horas** após entrega do relatório de testes pelo QA. |
| **Se reprovar** | QA registra gap no Azure DevOps como bug em um workitem com severity alta, vinculado à task de implementação. Feature retorna para a etapa de Implement. Dev corrige o código ou adiciona o teste faltante. Pipeline de CI deve ser reexecutado integralmente antes de novo Gate 4. Deploy bloqueado até aprovação. |

---

> **Observação de processo:** Os gates não são etapas adicionais de burocracia — são o ponto em que o time humano assume responsabilidade pelo output gerado pela IA. A velocidade de cada gate depende da disciplina do time em manter specs claras (Gate 1) e tasks bem descritas (Gate 2), pois esses dois gates são os que mais frequentemente reprovam quando o input para a IA foi impreciso.

---

## 4. Limitações e Riscos do Modelo AI-First

O workflow descrito neste documento representa uma estrutura ideal. Na prática, times enxutos como o da NovaTech encontrarão pontos de atrito previsíveis. Identificar esses riscos antecipadamente é parte do trabalho do Delivery Manager — negá-los é o caminho mais rápido para que o time abandone o processo em silêncio.

### Risco 1 — Gate 1 não captura ambiguidades que só emergem na implementação

O `requirements.md` é revisado por PS e TL com base no que está escrito. Ambiguidades de domínio — especialmente em regras de negócio sutis, como o comportamento exato do assistente frente a documentos com vigências sobrepostas (não apenas contraditórios) — frequentemente só se tornam visíveis quando o Dev tenta traduzir a spec em código. O Gate 1 filtra requisitos incompletos e incompatibilidades óbvias, mas não substitui a descoberta progressiva que acontece durante a implementação. **Mitigação parcial:** incentivar que Devs anotem dúvidas de domínio como comentários inline no código (`// TODO: comportamento indefinido na spec §3.2`) e que essas anotações disparem uma revisão rápida com o PS antes de se tornarem dívida técnica silenciosa.

### Risco 2 — Gate 2 cria gargalo no Tech Lead em sprints sobrecarregadas

Com 5 módulos em paralelo e um único Tech Lead como aprovador exclusivo do Gate 2, o modelo assume disponibilidade contínua de um papel que também é responsável por code review (Gate 3), decisões arquiteturais pontuais e suporte aos Devs durante a implementação. Em sprints de alta densidade, o TL se torna o único bloqueador de três gates simultâneos. **Mitigação parcial:** o DM deve monitorar proativamente o número de gates pendentes atribuídos ao TL no Azure DevOps e acionar redistribuição de carga (ex.: delegar Gate 2 de módulos de menor risco ao Dev Sênior, com validação posterior do TL) antes que o gargalo se materialize, não depois.

### Risco 3 — IA pode gerar tasks formalmente corretas com dependências ocultas

O GitHub Copilot e o Claude geram `tasks.md` com base na estrutura do `plan.md`, mas não têm visibilidade sobre o estado real do ambiente (serviços Azure não provisionados, bibliotecas com conflito de versão, dados de teste ausentes). Uma task pode ser textualmente correta — "implementar endpoint POST /query integrando Azure AI Search" — mas implicitamente depender de um índice de busca que ainda não existe, de um segredo no Key Vault que não foi criado, ou de uma decisão de schema que o TL não finalizou. O Gate 2 captura dependências técnicas óbvias, mas dependências implícitas de ambiente só aparecem quando o Dev tenta executar a task. **Mitigação parcial:** incluir no `tasks.md` um bloco de pré-requisitos explícitos por task (serviços provisionados, variáveis de ambiente, dados de seed necessários), tornando as dependências ocultas visíveis antes da implementação.

### Risco 4 — Deriva de contexto dos agentes entre sessões de trabalho

Copilot e Claude não persistem contexto entre sessões. Um Dev que retoma uma task no dia seguinte com uma nova sessão de agente está trabalhando com um assistente que não tem memória do raciocínio da sessão anterior — e pode tomar decisões contraditórias com o que foi construído horas antes. Em módulos com alta interdependência (ex.: o bot do Teams consumindo a API de busca), essa deriva silenciosa pode gerar inconsistências que só são detectadas no Gate 3, quando o custo de correção já é alto. **Mitigação parcial:** padronizar que cada nova sessão de agente começa com a injeção do `tasks.md` atualizado, dos ADRs relevantes e do diff recente do módulo como contexto explícito — nunca assumir que o agente "lembra" o que foi feito.

### Risco 5 — Velocidade da IA pressiona os gates a virarem formalidade

O principal benefício do modelo AI-First — geração acelerada de código e artefatos — cria uma pressão cultural implícita para que os gates sejam executados rapidamente e sem profundidade. Quando o Copilot entrega um `tasks.md` em minutos e o Dev está ansioso para começar a implementar, o Tech Lead sente o peso social de "não segurar o time". Gates executados sob essa pressão tendem a se tornar checkboxes sem substância, especialmente o Gate 2 (revisão de tasks) e o Gate 4 (suficiência de testes). O risco não é técnico — é comportamental. **Mitigação parcial:** o DM deve tornar explícito nas dailies que o tempo gasto num gate bem executado é recuperado em débito técnico evitado; e deve monitorar se o tempo médio de revisão dos gates está caindo de forma suspeita ao longo das sprints.
