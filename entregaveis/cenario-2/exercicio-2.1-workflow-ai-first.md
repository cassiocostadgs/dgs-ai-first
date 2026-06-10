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
| **Se reprovar** | QA registra gap no Azure DevOps como bug com severity alta, vinculado à task de implementação. Feature retorna para a etapa de Implement. Dev corrige o código ou adiciona o teste faltante. Pipeline de CI deve ser reexecutado integralmente antes de novo Gate 4. Deploy bloqueado até aprovação. |

---

> **Observação de processo:** Os gates não são etapas adicionais de burocracia — são o ponto em que o time humano assume responsabilidade pelo output gerado pela IA. A velocidade de cada gate depende da disciplina do time em manter specs claras (Gate 1) e tasks bem descritas (Gate 2), pois esses dois gates são os que mais frequentemente reprovam quando o input para a IA foi impreciso.
