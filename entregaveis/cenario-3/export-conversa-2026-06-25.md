# Exportação da Conversa — NovaTech AI Assistant (Cenário 3)
**Data:** 25/06/2026  |  **Sessão:** Governança de IA, Go-Live e Observabilidade

---

## Usuário
Aja como um Delivery Manager sênior especializado em Harness Engineering e Governança de IA. Seu objetivo é estruturar os critérios de go-live, o dashboard de acompanhamento e o plano de rollback com base no cenário real do nosso projeto na NovaTech.

---

### CONTEXTO DO PROJETO: NOVATECH (FASE DE GOVERNANÇA E VALIDAÇÃO)

O assistente de IA da NovaTech está em desenvolvimento e temos uma demonstração crucial para a diretoria em exatamente 2 semanas. 

O que temos construído até agora:
- Pipeline de ingestão processa 847 documentos indexados no Azure AI Search.
- Endpoint de query recebe perguntas via POST, busca chunks e retorna respostas com citação de fonte.
- Bot do Teams funcional em staging, acessível por 5 atendentes-piloto.
- Artefatos no repositório: AGENTS.md, especificações SDD, skills e guardrails de produto (formalizados em DEVE / NÃO DEVE / QUANDO EM DÚVIDA).
- Testes de integração cobrem cerca de 75% do código.

Problemas e Descobertas Recentes (Devem ser mitigados pelos critérios de Go-Live):
1. Alinhamento de Qualidade: Em testes internos, 12% das respostas estavam incorretas (alucinação, documentos desatualizados e recuperação de chunks incorretos).
2. Falta de Padronização: As respostas do assistente são retornadas em texto livre. Não há formato estruturado (JSON) que garanta campos obrigatórios (fonte, score de confiança). Se o modelo "esquecer" a fonte, a resposta é enviada assim mesmo.
3. Quebra de Governança no Código: Um desenvolvedor gerou um módulo de feedback usando Copilot que ignorou as regras do AGENTS.md (não utilizou Zod para validação e logou dados sensíveis do atendente).

---

### CONCEITOS-CHAVE E FERRAMENTAS DISPONÍVEIS
- Framework de Harness (5 camadas): 
  (1) Tool orchestration (coordenação de ferramentas/agentes)
  (2) Verification loops (verificação automática de outputs)
  (3) Context & memory (manutenção de contexto)
  (4) Guardrails (limites do sistema e pontos de HITL)
  (5) Observability (visibilidade em produção)
- Structured Outputs: Uso de JSON estrito com campos obrigatórios (ex: answer, source_document, confidence_score) para rejeitar programaticamente respostas fora do padrão.
- Human-in-the-Loop (HITL): Validação humana obrigatória baseada no risco da decisão.
- Ferramentas do Time: Claude (chat), GitHub Copilot, Claude Cowork, Claude Design.

---

### TAREFA: GERAR OS TRÊS ENTREGÁVEIS

Com base nessas informações, gere os seguintes entregáveis detalhados:

#### 1. Critérios de Go-Live por Camada de Harness
Defina critérios específicos organizados estritamente pelas 5 camadas do framework de Harness, atacando diretamente os problemas descobertos (os 12% de erro, o problema do texto livre e o furo de governança no código).
- Classifique cada critério como [BLOQUEANTE] (sem isso não vai ao ar para a diretoria) ou [DESEJÁVEL] (pode ir ao ar, virando débito técnico pós-demo). Demonstre pragmatismo técnico.
- Obrigatório: Na camada de Guardrails, inclua ao menos um ponto de Human-in-the-Loop (HITL) concreto (ex: respostas com score de confiança abaixo de X ou sobre cargas perigosas passam por revisão dos atendentes-piloto antes de serem exibidas).

#### 2. Dashboard de Readiness do Go-Live (Simulação Claude Cowork)
Crie uma tabela estruturada em Markdown para servir de visão executiva no Claude Cowork. A tabela deve conter as colunas:
- Camada do Harness
- Critério de Go-Live
- Classificação (Bloqueante / Desejável)
- Status Atual (Verde / Amarelo / Vermelho) -> Distribua os status de forma realista considerando que faltam 2 semanas e temos problemas a resolver.
- Responsável (ex: Tech Lead, QA, Delivery Manager, Product Specialist)
- Data-Alvo (dentro da janela de 2 semanas)

#### 3. Plano de Rollback Simplificado
Defina um plano de contingência direto, sem termos vagos como "analisar a situação". Foque em comandos e ações objetivas:
- Gatilho (Trigger): Qual o sinal claro, métrica ou comportamento do bot no Teams que dispara o alerta de que algo deu errado.
- Dono da Decisão: Quem tem o papel final para autorizar o desligamento/reversão.
- Ação Imediata: O passo técnico exato para reverter o sistema ou desativar o bot de forma segura.

Diretriz de Resposta: Seja extremamente profissional, use a terminologia correta de Engenharia de IA e evite respostas genéricas. Foque no cenário da NovaTech.

---

## Assistente
# Entregáveis de Go-Live: NovaTech AI Assistant

---

## 1. Critérios de Go-Live por Camada de Harness

> Escopo de referência: demo para diretoria em D+14. Critérios [BLOQUEANTE] são pré-condição para qualquer exposição ao público executivo.

---

### Camada 1 — Tool Orchestration

**Problema atacado:** recuperação de chunks incorretos e documentos desatualizados contribuindo para os 12% de erro.

| # | Critério | Classificação |
|---|----------|--------------|
| 1.1 | O pipeline de ingestão deve indexar metadados de `last_updated_at` em cada documento. Chunks oriundos de documentos com vigência expirada (definir threshold, ex: >180 dias sem atualização) devem receber flag `is_stale: true` e ser excluídos do retrieval por padrão. | **[BLOQUEANTE]** |
| 1.2 | O endpoint `POST /query` deve retornar exclusivamente Structured Output em JSON estrito, validado contra schema fixo com os campos obrigatórios: `answer` (string), `source_document` (string, non-null), `confidence_score` (float 0–1), `chunk_ids` (array). Respostas em texto livre devem ser **rejeitadas na camada de orquestração** antes de chegar ao Teams. | **[BLOQUEANTE]** |
| 1.3 | Implementar fallback explícito: se o Azure AI Search retornar zero chunks com score acima do threshold mínimo de relevância (ex: cosine similarity < 0.75), o orquestrador deve retornar uma resposta-padrão de "informação não encontrada na base" — nunca invocar o LLM com contexto vazio. | **[BLOQUEANTE]** |
| 1.4 | Mecanismo de retry com exponential backoff (3 tentativas, delay de 500ms/1s/2s) para falhas transitórias de chamada ao Azure AI Search e ao endpoint do modelo. | **[DESEJÁVEL]** |

---

### Camada 2 — Verification Loops

**Problema atacado:** 12% de erro e ausência de garantia estrutural de que campos obrigatórios existem na resposta.

| # | Critério | Classificação |
|---|----------|--------------|
| 2.1 | Implementar validation loop pós-geração via **Zod schema** (TypeScript) ou equivalente: toda resposta do LLM é parseada e validada antes de ser despachada. Se `source_document` for `null`, `undefined` ou string vazia, a resposta é **descartada automaticamente**, um erro estruturado é logado (`RESPONSE_VALIDATION_FAILED`) e o usuário recebe mensagem-padrão de fallback. Sem exceções. | **[BLOQUEANTE]** |
| 2.2 | A taxa de `RESPONSE_VALIDATION_FAILED` em staging deve ser < 5% em janela de 24h antes do go-live. Acima disso, o critério 1.2 ainda não está funcionando corretamente no prompt/schema e o go-live não pode ocorrer. | **[BLOQUEANTE]** |
| 2.3 | Para as top-10 perguntas mais frequentes identificadas nos testes-piloto, criar golden dataset com respostas esperadas. Rodar avaliação automatizada (ex: LLM-as-judge ou exact match em campos estruturados) e exigir accuracy ≥ 85% nesse conjunto antes do go-live. | **[BLOQUEANTE]** |
| 2.4 | Self-consistency check para queries classificadas como alta sensibilidade (ex: queries sobre processos críticos de negócio): gerar 2 respostas independentes e comparar convergência antes de retornar. Implementar após a demo. | **[DESEJÁVEL]** |

---

### Camada 3 — Context & Memory

**Problema atacado:** recuperação de documentos desatualizados impactando qualidade das respostas.

| # | Critério | Classificação |
|---|----------|--------------|
| 3.1 | O pipeline de ingestão deve ser reexecutado (re-indexação incremental) sempre que um documento-fonte for atualizado no repositório de origem. O índice do Azure AI Search deve refletir o estado atual da base documental com lag máximo de 24h. Chunks com `is_stale: true` devem ser auditados e atualizados ou removidos antes do go-live. | **[BLOQUEANTE]** |
| 3.2 | O contexto enviado ao LLM deve incluir explicitamente o campo `document_date` de cada chunk, para que o modelo seja instruído no system prompt a sinalizar quando estiver utilizando fontes com data anterior a um threshold. | **[DESEJÁVEL]** |
| 3.3 | Implementar memória de curto prazo por sessão no bot do Teams (janela de conversa), para evitar que o assistente contradiga respostas anteriores dentro da mesma thread. | **[DESEJÁVEL]** |

---

### Camada 4 — Guardrails

**Problema atacado:** ausência de HITL, furo de governança no módulo de feedback, falta de limites de tópico.

| # | Critério | Classificação |
|---|----------|--------------|
| 4.1 | **HITL obrigatório — ponto de interceptação por confiança:** Toda resposta com `confidence_score < 0.70` deve ser interceptada antes de ser exibida no Teams. O sistema deve enfileirar a resposta em uma fila de revisão visível aos 5 atendentes-piloto, com SLA de resposta de 30 minutos. O atendente pode aprovar, editar ou descartar. O usuário final recebe mensagem: *"Consultando um especialista, aguarde."* Respostas sem revisão que excedam o SLA retornam mensagem-padrão de escalação. | **[BLOQUEANTE]** |
| 4.2 | **Topic boundary hardcoded:** O AGENTS.md define escopo. Queries que ativem classificadores de tópico fora do escopo (ex: aconselhamento jurídico, médico, financeiro de alta complexidade) devem retornar resposta-padrão pré-aprovada de escalação humana, sem invocar o LLM de geração. Essa regra não pode ser sobrescrita por instrução do usuário no chat. | **[BLOQUEANTE]** |
| 4.3 | **Remediação do módulo de feedback (furo de governança):** O módulo gerado via Copilot deve ser refatorado e aprovado pelo Tech Lead antes do merge em main. Checklist obrigatório: (a) validação de inputs via Zod, (b) remoção de todos os `console.log` com campos de dados do atendente, (c) substituição por log estruturado com PII masking para campos como `user_id`, `agent_name`, `query_content`. PR deve passar em code review com referência explícita ao AGENTS.md. | **[BLOQUEANTE]** |
| 4.4 | Rate limiting por usuário no Teams: máximo de 20 queries/hora por atendente. Proteção contra abuso acidental que degradaria o serviço para os outros usuários-piloto. | **[DESEJÁVEL]** |

---

### Camada 5 — Observability

**Problema atacado:** sem visibilidade, qualquer problema em produção será descoberto tarde demais.

| # | Critério | Classificação |
|---|----------|--------------|
| 5.1 | Dashboard de métricas em produção ativo **antes** do go-live, com as seguintes métricas em tempo real: `query_volume`, `avg_confidence_score`, `hitl_queue_depth`, `schema_validation_failure_rate`, `latência P95 (ms)`, `error_rate (HTTP 5xx)`. Visível para Delivery Manager e Tech Lead. | **[BLOQUEANTE]** |
| 5.2 | Todos os logs de interação devem estar em conformidade com AGENTS.md: sem PII de atendente (name, email, query raw text sem masking). Auditoria de log a ser executada pelo QA antes do go-live como critério de aceite. | **[BLOQUEANTE]** |
| 5.3 | Alerta automático configurado: se `schema_validation_failure_rate > 10%` ou `error_rate > 15%` em janela deslizante de 15 minutos, notificação automática para o canal do Teams do time técnico + Delivery Manager. Alerta não pode depender de alguém olhar o dashboard manualmente. | **[BLOQUEANTE]** |
| 5.4 | Trace distribuído (ex: Application Insights ou OpenTelemetry) correlacionando query_id → retrieval → geração → validação → entrega, para diagnóstico rápido de root cause em caso de incidente. | **[DESEJÁVEL]** |

---

## 2. Dashboard de Readiness do Go-Live

> Snapshot: D-14 antes da demo. Status distribuídos com base nos problemas reportados e no estado atual do projeto.

| Camada do Harness | Critério de Go-Live | Classificação | Status Atual | Responsável | Data-Alvo |
|-------------------|---------------------|---------------|--------------|-------------|-----------|
| **Tool Orchestration** | Metadados `last_updated_at` indexados + flag `is_stale` no pipeline | BLOQUEANTE | 🔴 Vermelho | Tech Lead | D+5 |
| **Tool Orchestration** | Endpoint retorna Structured Output JSON com schema fixo obrigatório | BLOQUEANTE | 🔴 Vermelho | Tech Lead | D+6 |
| **Tool Orchestration** | Fallback explícito quando retrieval retorna zero chunks relevantes | BLOQUEANTE | 🟡 Amarelo | Tech Lead | D+7 |
| **Tool Orchestration** | Retry com exponential backoff para falhas transitórias | DESEJÁVEL | 🟡 Amarelo | Tech Lead | D+10 |
| **Verification Loops** | Validation loop via Zod pós-geração, descarte de respostas sem `source_document` | BLOQUEANTE | 🔴 Vermelho | Tech Lead | D+7 |
| **Verification Loops** | Taxa `RESPONSE_VALIDATION_FAILED` < 5% em staging (24h window) | BLOQUEANTE | 🔴 Vermelho | QA | D+10 |
| **Verification Loops** | Golden dataset (top-10 queries) com accuracy ≥ 85% | BLOQUEANTE | 🟡 Amarelo | QA + Product Specialist | D+9 |
| **Verification Loops** | Self-consistency check para queries de alta sensibilidade | DESEJÁVEL | 🔴 Vermelho | Tech Lead | Pós-demo |
| **Context & Memory** | Re-indexação incremental configurada, lag máximo 24h | BLOQUEANTE | 🟡 Amarelo | Tech Lead | D+5 |
| **Context & Memory** | Auditoria e remoção/atualização de chunks `is_stale` | BLOQUEANTE | 🔴 Vermelho | Tech Lead + Product Specialist | D+8 |
| **Context & Memory** | `document_date` no contexto enviado ao LLM | DESEJÁVEL | 🔴 Vermelho | Tech Lead | Pós-demo |
| **Guardrails** | HITL ativo: respostas com `confidence_score < 0.70` vão para fila de revisão (SLA 30min) | BLOQUEANTE | 🔴 Vermelho | Tech Lead + Delivery Manager | D+8 |
| **Guardrails** | Topic boundary hardcoded para tópicos fora de escopo | BLOQUEANTE | 🟡 Amarelo | Tech Lead + Product Specialist | D+7 |
| **Guardrails** | Remediação do módulo de feedback (Zod + PII masking + AGENTS.md compliance) | BLOQUEANTE | 🔴 Vermelho | Tech Lead | D+4 |
| **Guardrails** | Rate limiting 20 queries/hora por atendente no Teams | DESEJÁVEL | 🔴 Vermelho | Tech Lead | D+12 |
| **Observability** | Dashboard de métricas em produção ativo antes do go-live | BLOQUEANTE | 🟡 Amarelo | Tech Lead + Delivery Manager | D+9 |
| **Observability** | Logs em conformidade com AGENTS.md (sem PII, auditado pelo QA) | BLOQUEANTE | 🔴 Vermelho | QA | D+8 |
| **Observability** | Alerta automático configurado (Teams channel) para anomalias | BLOQUEANTE | 🟡 Amarelo | Tech Lead | D+10 |
| **Observability** | Trace distribuído (Application Insights / OpenTelemetry) | DESEJÁVEL | 🔴 Vermelho | Tech Lead | Pós-demo |

**Legenda de Risco:** 9 itens BLOQUEANTES em Vermelho — risco crítico de go-live se nenhuma ação ocorrer hoje. Prioridade imediata: remediação do módulo de feedback (D+4) e Structured Output (D+6).

---

## 3. Plano de Rollback Simplificado

> Princípio: o rollback não é uma decisão de engenharia — é uma decisão de negócio com execução técnica padronizada. Toda ambiguidade sobre "quando acionar" elimina tempo precioso durante um incidente.

---

### Triggers de Alerta (Nível 1 — Monitoramento)

Condições que acionam **alerta automático** e notificação imediata ao Tech Lead e Delivery Manager:

| Trigger | Métrica | Threshold |
|---------|---------|-----------|
| T1 | `error_rate` (HTTP 5xx no endpoint `/query`) | > 15% em janela de 15 min |
| T2 | `schema_validation_failure_rate` | > 10% em janela de 15 min |
| T3 | `hitl_queue_depth` (itens pendentes sem revisão) | > 30 itens acumulados |
| T4 | `avg_confidence_score` | < 0.55 em janela de 30 min |
| T5 | Bot do Teams retornando respostas em texto livre sem campo `source_document` | Qualquer ocorrência confirmada |

Qualquer trigger ativo por mais de **10 minutos sem resolução** escala automaticamente para decisão de rollback.

---

### Dono da Decisão de Rollback

| Situação | Autoriza o rollback |
|----------|---------------------|
| Demo para diretoria em andamento | **Delivery Manager** (decisão imediata, sem necessidade de consulta) |
| Ambiente de staging/piloto (5 atendentes) | **Tech Lead** (pode acionar sem aprovação prévia do DM) |
| Produção ampliada (futuro) | **Delivery Manager** + sign-off do **Product Owner** |

A autorização é verbal ou via mensagem no canal do Teams do time. Não exige cerimônia. O critério é: *"o bot está causando mais dano do que ausência de bot causaria?"*

---

### Ações de Rollback (Execução Técnica)

**Passo 1 — Desativar o bot no Teams (< 2 minutos)**
```
Teams Admin Center → Manage apps → DGS AI Assistant → Status: Blocked
```
Efeito imediato: bot some do Teams para todos os atendentes-piloto. Nenhuma query nova é processada. Nenhum dado é perdido.

**Passo 2 — Reverter o endpoint para a última tag estável (< 5 minutos)**
```bash
git tag --list "stable-*"          # identificar última tag estável
git checkout stable-<versão>       # ou via pipeline de CD:
az webapp deployment source config-zip \
  --resource-group novatech-rg \
  --name dgs-api \
  --src ./releases/stable-<versão>.zip
```
Alternativa se pipeline de CD estiver disponível: redeployar a partir da última release tag aprovada diretamente pelo portal Azure DevOps, sem necessidade de acesso manual ao servidor.

**Passo 3 — Confirmar estabilização**

Aguardar 3 minutos após o revert e verificar no dashboard:
- `error_rate` < 5%
- `schema_validation_failure_rate` < 5%
- Endpoint respondendo com HTTP 200 em health check

**Passo 4 — Comunicar**

Delivery Manager envia mensagem padrão no canal de stakeholders:
> *"O assistente AI está temporariamente indisponível para manutenção preventiva. Previsão de retorno: [X horas]. Os atendentes devem utilizar o fluxo manual de consulta. Atualizações a cada 30 minutos."*

**Passo 5 — Post-mortem (dentro de 24h após o incidente)**

Tech Lead documenta no repositório: trigger acionado, root cause identificado, fix aplicado, critério de volta ao ar. O documento é a evidência para que a diretoria saiba que o time tem controle do processo.

---

> **Nota para o time:** Os 9 critérios BLOQUEANTES em vermelho representam dívida operacional que precisa ser eliminada nos próximos 8 dias para viabilizar a demo com segurança. A recomendação é começar hoje pela remediação do módulo de feedback (D+4) — é o item com maior risco reputacional e o menor esforço relativo de resolução.

---

## Usuário
salve os .md na pasta "cenario-3"

---

## Assistente
Vou verificar a estrutura de pastas antes de salvar.

---

## Assistente
A pasta existe e está vazia. Vou criar os três arquivos .md.

---

## Assistente
Três arquivos salvos em [entregaveis/cenario-3/](entregaveis/cenario-3/):

- [criterios-go-live.md](entregaveis/cenario-3/criterios-go-live.md) — critérios por camada de harness com classificação BLOQUEANTE/DESEJÁVEL
- [dashboard-readiness.md](entregaveis/cenario-3/dashboard-readiness.md) — tabela executiva com status, responsável e data-alvo
- [plano-rollback.md](entregaveis/cenario-3/plano-rollback.md) — triggers, dono da decisão e comandos de execução

---

## Usuário
Aja como um Delivery Manager sênior especializado em Governança de IA e Engenharia de Confiabilidade (MRE/SRE). O bot da NovaTech foi para o ar (go-live), e agora precisamos estruturar o Plano de Observabilidade e Melhoria Contínua para garantir que o assistente evolua de forma segura e os erros sejam corrigidos rapidamente.

Relembrando o nosso Histórico/Contexto da NovaTech:
- O bot está integrado ao Teams com 5 atendentes-piloto.
- Partimos de uma linha de base onde 12% das respostas internas estavam incorretas (alucinações/chunks errados).
- Adotamos Structured Outputs e um fluxo de Human-in-the-Loop (HITL).
- Temos 847 documentos indexados no Azure AI Search.

Gere os seguintes 2 entregáveis detalhados:

---

#### 1. Plano de Observabilidade e Ciclo de Melhoria (Feedback Loop)

A. Matriz de Métricas (Mapeie em formato de tabela as 4 dimensões obrigatórias):
   - Métricas de Uso (Ex: perguntas/dia, tempo de resposta/interação).
   - Métricas de Qualidade (Ex: % de feedback negativo dos atendentes, % de escalações para humanos).
   - Métricas Técnicas (Ex: latência da API do Azure AI Search/LLM, taxa de erro de chamadas).
   - Métricas de Conteúdo (Ex: documentos mais consultados, volumetria de perguntas sem resposta no RAG).

B. Alertas Críticos com Limiares (Thresholds) Concretos:
   Defina de 2 a 3 alertas operacionais automáticos. Não use termos vagos como "se as alucinações aumentarem". Estipule regras numéricas claras (Ex: "Se a taxa de feedback negativo ultrapassar 15% em uma janela móvel de 24 horas -> Disparar notificação no canal de incidentes do time").

C. Fluxo de Feedback Loop Simplificado:
   Desenhe o passo a passo de como o feedback negativo deixado por um dos 5 atendentes-piloto no Teams navega pelo sistema, é analisado pelo time de engenharia e vira uma correção prática no assistente (seja via ajuste de prompt, atualização de documento ou engenharia de dados).

---

#### 2. Template de Relatório Semanal de Prontidão (Simulação Claude Cowork)

A diretoria da NovaTech exigiu um relatório semanal para acompanhar se o assistente está melhorando ou piorando. Crie um template em Markdown de exatamente 1 página (focado em leitura executiva de 2 minutos). O template deve conter:

- Cabeçalho Executivo (Data, Versão do Bot, Status Geral do Período).
- Visão Geral dos KPIs Principais (Uso, Qualidade, Técnica e Conteúdo).
- Indicador de Tendência Clara (Sinalize se o indicador Melhorou [▲], Piorou [▼] ou Estabilizou [▬] em relação à semana anterior).
- Sumário de Conteúdo (Quais as top 3 perguntas sem resposta e os documentos mais acionados).
- Plano de Ação Próxima Semana (Ações planejadas baseadas nos desvios encontrados, ex: re-ingestão de documento X, ajuste de guardrail Y).

---
Diretriz de Resposta: Escreva de forma analítica, corporativa e altamente acionável. O foco principal é converter dados técnicos de IA em visibilidade de negócio para a diretoria da NovaTech.

---

## Assistente
# Observabilidade e Melhoria Contínua — NovaTech AI Assistant (Pós Go-Live)

---

## 1. Plano de Observabilidade e Ciclo de Melhoria (Feedback Loop)

### A. Matriz de Métricas

#### Dimensão 1 — Uso

| Métrica | Descrição | Frequência de Coleta | Fonte |
|---------|-----------|----------------------|-------|
| `queries_per_day` | Total de perguntas submetidas ao bot por dia | Diária | Teams Bot / API Gateway |
| `active_users_per_day` | Atendentes únicos que interagiram com o bot no dia | Diária | Teams Bot |
| `avg_session_length` | Tempo médio de uma conversa (primeira query → última mensagem) | Diária | Teams Bot |
| `queries_per_user` | Média de perguntas por atendente ativo | Semanal | Teams Bot |
| `peak_hour_volume` | Horário de maior concentração de queries (ex: 9h–11h) | Diária | API Gateway |

#### Dimensão 2 — Qualidade

| Métrica | Descrição | Frequência de Coleta | Fonte |
|---------|-----------|----------------------|-------|
| `negative_feedback_rate` | % de respostas que receberam thumbs down ou marcação explícita de "resposta incorreta" pelo atendente | Diária | Teams Bot (feedback widget) |
| `hitl_escalation_rate` | % de respostas interceptadas pelo HITL (confidence_score < 0.70) sobre o total de queries | Diária | Middleware HITL |
| `hitl_approval_rate` | % de respostas na fila HITL que foram aprovadas (vs. descartadas/editadas) pelos atendentes-piloto | Semanal | Middleware HITL |
| `fallback_rate` | % de queries que retornaram resposta-padrão de "informação não encontrada" (retrieval abaixo do threshold) | Diária | API Gateway |
| `avg_confidence_score` | Média do campo `confidence_score` de todas as respostas entregues no período | Diária | Structured Output log |

#### Dimensão 3 — Técnica

| Métrica | Descrição | Frequência de Coleta | Fonte |
|---------|-----------|----------------------|-------|
| `p95_latency_ms` | Latência P95 do fluxo completo (query → resposta no Teams), em milissegundos | Contínua (alertável) | Application Insights |
| `azure_search_latency_ms` | Latência P95 exclusiva da chamada ao Azure AI Search | Contínua | Application Insights |
| `llm_latency_ms` | Latência P95 exclusiva da chamada ao LLM de geração | Contínua | Application Insights |
| `schema_validation_failure_rate` | % de respostas do LLM rejeitadas pelo Zod schema (campos ausentes/inválidos) | Contínua (alertável) | Validation middleware |
| `api_error_rate_5xx` | Taxa de erros HTTP 5xx no endpoint `/query` | Contínua (alertável) | API Gateway |
| `retry_trigger_rate` | % de chamadas que acionaram o mecanismo de retry (exponential backoff) | Diária | API Gateway |

#### Dimensão 4 — Conteúdo

| Métrica | Descrição | Frequência de Coleta | Fonte |
|---------|-----------|----------------------|-------|
| `top_10_documents_retrieved` | Ranking dos 10 documentos mais acionados via retrieval | Semanal | Azure AI Search logs |
| `unanswered_query_rate` | % de queries onde nenhum chunk com score ≥ threshold foi retornado (gap de cobertura da base) | Diária | Azure AI Search logs |
| `stale_chunk_hit_rate` | % de respostas que utilizaram chunks com flag `is_stale: true` (documentos desatualizados) | Diária | Pipeline de ingestão |
| `top_5_unanswered_themes` | Agrupamento temático das queries sem resposta RAG (identificar gaps de conteúdo) | Semanal | NLP clustering sobre query logs |
| `document_coverage_ratio` | % dos 847 documentos que foram consultados ao menos uma vez na semana | Semanal | Azure AI Search logs |

---

### B. Alertas Críticos com Limiares Concretos

#### Alerta 1 — Degradação de Qualidade Percebida pelo Usuário

**Regra:**
```
SE negative_feedback_rate > 15%
EM janela móvel de 24 horas (mínimo 20 queries no período)
→ Disparar P2 no canal #novatech-ai-incidentes (Teams)
→ Notificar: Tech Lead + QA + Delivery Manager
→ SLA de primeira resposta: 2 horas
```

**Contexto:** A linha de base histórica é 12% de respostas incorretas em testes internos. Um threshold de 15% em produção sinaliza regressão acima da baseline, com alta probabilidade de impacto direto na experiência dos atendentes.

---

#### Alerta 2 — Instabilidade Técnica do Endpoint

**Regra:**
```
SE api_error_rate_5xx > 10%
OU schema_validation_failure_rate > 8%
EM janela móvel de 15 minutos
→ Disparar P1 no canal #novatech-ai-incidentes (Teams)
→ Notificar: Tech Lead (obrigatório) + Delivery Manager
→ SLA de primeira resposta: 30 minutos
→ Se não resolvido em 60 minutos: acionar Plano de Rollback (ver plano-rollback.md)
```

**Contexto:** `schema_validation_failure_rate` acima de 8% indica que o LLM está quebrando o contrato de Structured Output com frequência, o que compromete toda a camada de HITL e de citação de fonte.

---

#### Alerta 3 — Sobrecarga da Fila HITL

**Regra:**
```
SE hitl_queue_depth > 20 itens pendentes sem revisão
E tempo_mais_antigo_na_fila > 45 minutos
→ Disparar aviso no canal #novatech-ai-atendentes (Teams)
→ Notificar os 5 atendentes-piloto para priorizar revisão da fila
→ Se hitl_queue_depth > 40 itens: escalar para Delivery Manager
   com status "HITL inoperante — risco de SLA de 30min quebrado"
```

**Contexto:** Uma fila HITL saturada significa que usuários estão aguardando respostas de baixa confiança sem resolução, o que na prática equivale ao bot estar offline para um subconjunto de queries.

---

### C. Fluxo de Feedback Loop Simplificado

```
┌─────────────────────────────────────────────────────────────────────┐
│  ATENDENTE-PILOTO no Teams                                          │
│  Recebe resposta → clica "👎 Resposta incorreta"                    │
│  (widget embutido no card do bot) → campo opcional de comentário    │
└────────────────────────┬────────────────────────────────────────────┘
                         │ evento: feedback_negative_submitted
                         ▼
┌─────────────────────────────────────────────────────────────────────┐
│  CAMADA DE COLETA (automatizada)                                    │
│  • Log estruturado gravado: { query_id, query_text (masked),        │
│    response_text, source_document, confidence_score,                │
│    feedback_type: "negative", comment, timestamp, agent_id (hash) } │
│  • Evento enviado ao dashboard de observabilidade                   │
│  • negative_feedback_rate recalculado em tempo real                 │
└────────────────────────┬────────────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────────────┐
│  TRIAGEM (QA — diária, toda manhã às 9h)                           │
│  • QA revisa lote de feedbacks negativos acumulados                 │
│  • Classifica cada item em uma das 3 categorias:                    │
│    [A] Alucinação do modelo (resposta inventada)                    │
│    [B] Documento desatualizado (fonte existe mas dado está errado)  │
│    [C] Chunk incorreto (documento certo, trecho errado recuperado)  │
│  • Abre ticket no repositório com categoria, query_id e evidência   │
└────────────────────────┬────────────────────────────────────────────┘
                         │
          ┌──────────────┴──────────────┐──────────────┐
          ▼                             ▼               ▼
   [A] Alucinação              [B] Documento    [C] Chunk incorreto
          │                    desatualizado            │
          ▼                             │               ▼
  Tech Lead revisa               Product      Tech Lead revisa
  system prompt e                Specialist   configuração do
  few-shot examples.             atualiza o   retrieval (chunk
  Ajusta instrução de            documento    size, overlap,
  grounding: "responda           na base de   score threshold).
  apenas com base nos            origem →     Reindexação do
  documentos fornecidos,         Tech Lead    documento afetado.
  nunca extrapole."              reindexação  Teste de regressão.
  Teste A/B em staging.          incremental.
          │                             │               │
          └──────────────┬──────────────┘──────────────┘
                         ▼
┌─────────────────────────────────────────────────────────────────────┐
│  VALIDAÇÃO (QA — antes do deploy)                                   │
│  • Rodar golden dataset (top-10 queries) → verificar accuracy ≥ 85% │
│  • Confirmar que a query que gerou o feedback negativo original      │
│    agora retorna resposta correta e com confidence_score ≥ 0.70     │
│  • Aprovar PR com referência ao ticket e ao feedback_id             │
└────────────────────────┬────────────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────────────┐
│  DEPLOY E FECHAMENTO                                                │
│  • Deploy em staging → validação dos 5 atendentes-piloto           │
│  • Deploy em produção com nova tag semântica (ex: v1.2.1)          │
│  • Ticket fechado com link para o commit e métrica de melhoria      │
│  • Resultado registrado no Relatório Semanal da semana seguinte     │
└─────────────────────────────────────────────────────────────────────┘
```

**Tempo médio esperado do ciclo:** feedback negativo → correção em produção: **3 a 5 dias úteis** para categorias B e C; até **7 dias** para categoria A (ajuste de prompt exige ciclo de avaliação mais cuidadoso).

---

## 2. Template de Relatório Semanal de Prontidão

```markdown
# Relatório Semanal de Prontidão — NovaTech AI Assistant
**Data de Referência:** [DD/MM/AAAA] | **Semana:** [Nº] de [ANO]
**Versão do Bot:** v[X.Y.Z] | **Responsável:** [Nome do Delivery Manager]

---

## Status Geral do Período
> [🟢 ESTÁVEL / 🟡 ATENÇÃO / 🔴 CRÍTICO]
> Narrativa executiva em 2 linhas: o que mais chamou atenção na semana e se o assistente
> está dentro ou fora dos critérios de qualidade acordados.

---

## KPIs Principais da Semana

| Dimensão | Métrica | Valor Atual | Meta | vs. Semana Anterior | Tendência |
|----------|---------|-------------|------|---------------------|-----------|
| **Uso** | Queries/dia (média) | [N] | > 30/dia | [+N / -N] | [▲ / ▼ / ▬] |
| **Uso** | Atendentes ativos/dia | [N] / 5 | 5/5 | [+N / -N] | [▲ / ▼ / ▬] |
| **Qualidade** | Taxa de feedback negativo | [X]% | < 10% | [+X% / -X%] | [▲ / ▼ / ▬] |
| **Qualidade** | Taxa de escalação HITL | [X]% | < 20% | [+X% / -X%] | [▲ / ▼ / ▬] |
| **Qualidade** | Avg confidence_score | [0.XX] | ≥ 0.75 | [+0.XX / -0.XX] | [▲ / ▼ / ▬] |
| **Técnica** | Latência P95 (ms) | [N] ms | < 3.000 ms | [+N / -N] | [▲ / ▼ / ▬] |
| **Técnica** | schema_validation_failure | [X]% | < 5% | [+X% / -X%] | [▲ / ▼ / ▬] |
| **Técnica** | API error rate (5xx) | [X]% | < 3% | [+X% / -X%] | [▲ / ▼ / ▬] |
| **Conteúdo** | Taxa de queries sem resposta RAG | [X]% | < 8% | [+X% / -X%] | [▲ / ▼ / ▬] |
| **Conteúdo** | Chunks is_stale acionados | [X]% | 0% | [+X% / -X%] | [▲ / ▼ / ▬] |

> Legenda de tendência: ▲ Melhorou | ▼ Piorou | ▬ Estabilizou (variação < 2%)

---

## Sumário de Conteúdo

### Top 3 Perguntas sem Resposta na Base RAG
| # | Tema da Query (anonimizado) | Volume na Semana | Ação Sugerida |
|---|----------------------------|------------------|---------------|
| 1 | [Tema 1] | [N] queries | [Ex: Criar documento sobre X] |
| 2 | [Tema 2] | [N] queries | [Ex: Revisar cobertura de Y] |
| 3 | [Tema 3] | [N] queries | [Ex: Validar com Product Specialist] |

### Top 3 Documentos Mais Acionados
| # | Documento | Consultas na Semana | Última Atualização | Status |
|---|-----------|--------------------|--------------------|--------|
| 1 | [Nome do doc] | [N] | [DD/MM/AAAA] | [🟢 Vigente / 🔴 Desatualizado] |
| 2 | [Nome do doc] | [N] | [DD/MM/AAAA] | [🟢 Vigente / 🔴 Desatualizado] |
| 3 | [Nome do doc] | [N] | [DD/MM/AAAA] | [🟢 Vigente / 🔴 Desatualizado] |

---

## Plano de Ação — Próxima Semana

| # | Ação | Categoria | Responsável | Prazo | KPI Impactado |
|---|------|-----------|-------------|-------|---------------|
| 1 | [Ex: Re-ingestão do documento "Política de Crédito v3"] | Conteúdo | Tech Lead | [DD/MM] | stale_chunk_hit_rate |
| 2 | [Ex: Ajuste de threshold HITL de 0.70 → 0.75] | Guardrail | Tech Lead | [DD/MM] | hitl_escalation_rate |
| 3 | [Ex: Criação de documento sobre Tema 1 identificado] | Conteúdo | Product Specialist | [DD/MM] | unanswered_query_rate |
| 4 | [Ex: Revisão de few-shot examples para reduzir alucinação] | Prompt Engineering | Tech Lead + QA | [DD/MM] | negative_feedback_rate |

---
*Relatório gerado automaticamente a partir dos logs de observabilidade. Revisado e aprovado por: [Nome] — [DD/MM/AAAA]*
```

---

> **Nota de implementação:** O template acima deve ser preenchido automaticamente por um script de geração toda segunda-feira às 8h, consumindo as métricas do dashboard de observabilidade. O Delivery Manager revisa e adiciona a narrativa executiva e o plano de ação antes de enviar à diretoria até as 10h da mesma segunda-feira.

---

## Usuário
crie o claude.md do projeto

---

## Assistente
Vou explorar a estrutura do projeto antes de escrever o CLAUDE.md.

---

## Assistente
[CLAUDE.md](CLAUDE.md) criado na raiz do projeto. Ele cobre:

- **O que é o projeto** — treinamento AI-First do DB1, não codebase de produção.
- **Contexto NovaTech** — problema, stack, estado atual do bot.
- **Estrutura de pastas** — onde cada tipo de artefato vive.
- **Regras do AGENTS.md** — as restrições que nunca podem ser violadas (Zod, PII, gates, limites de tokens).
- **Convenções** — idioma por contexto, nomenclatura de arquivos, padrão de commits.
- **Framework de Harness** — as 5 camadas como referência conceitual central.
- **Ordem de leitura prioritária** — o que uma sessão nova deve consumir antes de gerar qualquer entregável.

Em sessões futuras, o Claude Code vai carregar esse arquivo automaticamente e terá contexto suficiente para trabalhar sem precisar de briefing repetido.

---

## Usuário
melhora a linguagem dos artefatos do criterio de go live, esta muito técnico, tem que ser claro e para leigo ver e aprovar

---

## Assistente
Vou reescrever ambos os arquivos trocando jargão técnico por linguagem de negócio clara.

---

## Assistente
Os dois arquivos foram reescritos. As principais mudanças:

**Critérios de go-live:**
- Nomes das camadas traduzidos para negócio ("Tool Orchestration" → "Coordenação do Assistente", etc.)
- Cada área tem um parágrafo curto explicando **por que importa** para quem não é técnico
- Jargão removido: sem `confidence_score`, `Zod schema`, `exponential backoff`, `cosine similarity`, `PII masking` — substituídos pelo que cada coisa faz na prática
- Critério 4.1 (HITL) reescrito com o fluxo do ponto de vista do atendente, não da engenharia

**Dashboard:**
- Coluna "Camada do Harness" virou "Área" com nomes em português
- Critérios reescritos como frases de resultado ("o assistente informa X" em vez de "implementar fallback Y")
- Status "Vermelho" virou "Não iniciado" e "Amarelo" virou "Em andamento" — mais claro para leigo
- Resumo executivo com narrativa direta sobre o risco e as prioridades

---

## Usuário
os itens do exercício 2 bem como o plano de observabilidade e template de relatório salver em .md na pasta "cenario-3"

---

## Assistente
Vou criar os dois arquivos restantes do cenário 3.

---

## Assistente
Dois arquivos criados em [entregaveis/cenario-3/](entregaveis/cenario-3/):

- [plano-observabilidade.md](entregaveis/cenario-3/plano-observabilidade.md) — matriz de métricas nas 4 dimensões, 3 alertas com thresholds numéricos e o fluxo completo do feedback loop
- [template-relatorio-semanal.md](entregaveis/cenario-3/template-relatorio-semanal.md) — template executivo de 1 página com KPIs, tendências (▲▼▬), sumário de conteúdo e plano de ação

A pasta cenário-3 agora tem 5 entregáveis completos: critérios de go-live, dashboard de readiness, plano de rollback, plano de observabilidade e template do relatório semanal.

---

## Usuário
o plano de observabilidade ta com palavras muito técnicas, tem que ser pos´sivel leigo ver

---

## Assistente
[plano-observabilidade.md](entregaveis/cenario-3/plano-observabilidade.md) reescrito. O que mudou:

- **Nomes de métricas** em código (`queries_per_day`, `negative_feedback_rate`, `p95_latency_ms`) removidos — substituídos por descrições em português direto
- **Coluna "Fonte"** removida da matriz — irrelevante para um leigo
- **Alertas** saíram do bloco de código técnico e viraram frases "quando dispara / o que acontece / por que importa"
- **Fluxo de feedback** teve os rótulos técnicos trocados: "Log estruturado", "chunk incorreto", "golden dataset", "deploy em staging", "PR" — tudo reescrito como ação de negócio compreensível
- Os 3 tipos de erro ([A], [B], [C]) ganharam explicações em linguagem do dia a dia: "resposta inventada", "documento desatualizado", "trecho errado"

---

## Usuário
revalide se os itens na pasta cenário-3 atendem os seguintes critérios:

Segue os critérios


- Os critérios são organizados pelas 5 camadas do harness (não é lista genérica).

- A distinção bloqueante vs desejável demonstra pragmatismo.

- Há ao menos um ponto de HITL concreto.

- O plano de rollback tem trigger, responsável e ação (não é "avaliar a situação").

- As métricas cobrem as 4 dimensões.

- Os alertas têm thresholds concretos (não "monitorar se piora").

- O feedback loop conecta o atendente à correção no assistente.

- O template é claro para um executivo entender em 2 minutos.

---

## Assistente
Vou ler todos os arquivos da pasta para validar cada critério.

---

## Assistente
Aqui está a validação critério a critério:

---

## Resultado da Validação — Cenário 3

| # | Critério | Arquivo | Resultado | Evidência |
|---|----------|---------|-----------|-----------|
| 1 | Critérios organizados pelas 5 camadas do harness | criterios-go-live.md / dashboard-readiness.md | ✅ Aprovado | 5 áreas nomeadas: Coordenação do Assistente, Verificação de Respostas, Base de Conhecimento, Proteções e Limites, Monitoramento — mapeadas 1:1 com as camadas do framework |
| 2 | Distinção bloqueante vs desejável demonstra pragmatismo | criterios-go-live.md | ✅ Aprovado | 13 itens BLOQUEANTE + 6 DESEJÁVEL com justificativa para cada classificação; itens de difícil prazo (trace distribuído, self-consistency) explicitamente adiados para pós-demo |
| 3 | Ao menos um ponto de HITL concreto | criterios-go-live.md (item 4.1) | ✅ Aprovado | Threshold definido (confiança < 70%), fila visível aos 5 atendentes-piloto, SLA de 30 min, mensagem exibida ao usuário e fallback automático se o prazo vencer |
| 4 | Rollback tem trigger, responsável e ação objetiva | plano-rollback.md | ✅ Aprovado | 5 triggers com threshold numérico (ex: > 15% erros em 15 min), tabela de dono da decisão por situação, 5 passos de execução com caminho exato no Teams Admin Center e comandos Azure CLI |
| 5 | Métricas cobrem as 4 dimensões | plano-observabilidade.md / template-relatorio-semanal.md | ✅ Aprovado | Dimensões Uso, Qualidade, Desempenho Técnico e Conteúdo presentes com indicadores distintos em cada uma |
| 6 | Alertas têm thresholds concretos | plano-observabilidade.md | ✅ Aprovado | Alerta 1: > 15% avaliações negativas em 24h (mín. 20 queries). Alerta 2: > 10% erros OU > 8% respostas bloqueadas em 15 min. Alerta 3: > 20 itens na fila + item mais antigo há > 45 min |
| 7 | Feedback loop conecta atendente à correção | plano-observabilidade.md (seção C) | ✅ Aprovado | Fluxo completo: thumbs down do atendente → registro automático → triagem QA (3 categorias) → caminho de correção por tipo → validação com perguntas de referência → publicação com registro no relatório semanal |
| 8 | Template claro para executivo em 2 minutos | template-relatorio-semanal.md | ✅ Aprovado | Semáforo no topo, KPIs com meta e tendência (▲▼▬), top 3 lacunas de conteúdo, plano de ação com responsável e prazo |

---

**Todos os 8 critérios atendidos.**

Dois pontos menores que não reprovam mas vale observar:

- **plano-rollback.md** ainda tem nomes técnicos na tabela de triggers (`error_rate`, `schema_validation_failure_rate`) — funciona bem como documento de operação para o time técnico, mas se for apresentado para a diretoria vale uma versão simplificada similar ao que foi feito nos critérios de go-live.
- **template-relatorio-semanal.md** mantém "P95 (ms)" na tabela de KPIs — recomendável renomear para "Tempo de resposta (95% das perguntas)" para leitura executiva.

Quer que eu corrija esses dois pontos?

---

## Usuário
sim, corrigir

---

## Assistente
Dois ajustes aplicados:

- [plano-rollback.md](entregaveis/cenario-3/plano-rollback.md) — tabela de triggers reescrita em linguagem de negócio: removidos nomes de variáveis em código, colunas renomeadas de "Métrica/Threshold" para "O que medir/Quando acionar", e a confirmação de estabilização saiu de nomes técnicos para frases descritivas
- [template-relatorio-semanal.md](entregaveis/cenario-3/template-relatorio-semanal.md) — "Tempo de resposta P95 (ms)" renomeado para "Tempo de resposta (95% das perguntas)"

---

## Usuário
exporte a conversa aqui do chat?

---

## Assistente
Não tenho uma ferramenta para exportar diretamente o histórico do chat. Mas o Claude Code salva transcrições automaticamente. Vou verificar onde está o arquivo desta sessão.

---

## Assistente
Encontrei o arquivo da sessão. Vou extrair as mensagens e salvar como Markdown legível.

---

## Assistente
O arquivo ficou quase vazio — o formato do JSONL é diferente do esperado. Vou inspecionar a estrutura real.

---

## Assistente
Agora sei a estrutura exata. Vou gerar o export corretamente.

---

