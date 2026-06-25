# Plano de Observabilidade e Ciclo de Melhoria — NovaTech AI Assistant

---

## A. Matriz de Métricas

### Dimensão 1 — Uso

| Métrica | Descrição | Frequência de Coleta | Fonte |
|---------|-----------|----------------------|-------|
| `queries_per_day` | Total de perguntas submetidas ao bot por dia | Diária | Teams Bot / API Gateway |
| `active_users_per_day` | Atendentes únicos que interagiram com o bot no dia | Diária | Teams Bot |
| `avg_session_length` | Tempo médio de uma conversa (primeira query → última mensagem) | Diária | Teams Bot |
| `queries_per_user` | Média de perguntas por atendente ativo | Semanal | Teams Bot |
| `peak_hour_volume` | Horário de maior concentração de queries (ex: 9h–11h) | Diária | API Gateway |

### Dimensão 2 — Qualidade

| Métrica | Descrição | Frequência de Coleta | Fonte |
|---------|-----------|----------------------|-------|
| `negative_feedback_rate` | % de respostas que receberam thumbs down ou marcação explícita de "resposta incorreta" pelo atendente | Diária | Teams Bot (feedback widget) |
| `hitl_escalation_rate` | % de respostas interceptadas pelo HITL (confidence_score < 0.70) sobre o total de queries | Diária | Middleware HITL |
| `hitl_approval_rate` | % de respostas na fila HITL que foram aprovadas (vs. descartadas/editadas) pelos atendentes-piloto | Semanal | Middleware HITL |
| `fallback_rate` | % de queries que retornaram resposta-padrão de "informação não encontrada" (retrieval abaixo do threshold) | Diária | API Gateway |
| `avg_confidence_score` | Média do campo `confidence_score` de todas as respostas entregues no período | Diária | Structured Output log |

### Dimensão 3 — Técnica

| Métrica | Descrição | Frequência de Coleta | Fonte |
|---------|-----------|----------------------|-------|
| `p95_latency_ms` | Latência P95 do fluxo completo (query → resposta no Teams), em milissegundos | Contínua (alertável) | Application Insights |
| `azure_search_latency_ms` | Latência P95 exclusiva da chamada ao Azure AI Search | Contínua | Application Insights |
| `llm_latency_ms` | Latência P95 exclusiva da chamada ao LLM de geração | Contínua | Application Insights |
| `schema_validation_failure_rate` | % de respostas do LLM rejeitadas pelo Zod schema (campos ausentes/inválidos) | Contínua (alertável) | Validation middleware |
| `api_error_rate_5xx` | Taxa de erros HTTP 5xx no endpoint `/query` | Contínua (alertável) | API Gateway |
| `retry_trigger_rate` | % de chamadas que acionaram o mecanismo de retry (exponential backoff) | Diária | API Gateway |

### Dimensão 4 — Conteúdo

| Métrica | Descrição | Frequência de Coleta | Fonte |
|---------|-----------|----------------------|-------|
| `top_10_documents_retrieved` | Ranking dos 10 documentos mais acionados via retrieval | Semanal | Azure AI Search logs |
| `unanswered_query_rate` | % de queries onde nenhum chunk com score ≥ threshold foi retornado (gap de cobertura da base) | Diária | Azure AI Search logs |
| `stale_chunk_hit_rate` | % de respostas que utilizaram chunks com flag `is_stale: true` (documentos desatualizados) | Diária | Pipeline de ingestão |
| `top_5_unanswered_themes` | Agrupamento temático das queries sem resposta RAG (identificar gaps de conteúdo) | Semanal | NLP clustering sobre query logs |
| `document_coverage_ratio` | % dos 847 documentos que foram consultados ao menos uma vez na semana | Semanal | Azure AI Search logs |

---

## B. Alertas Críticos com Limiares Concretos

### Alerta 1 — Degradação de Qualidade Percebida pelo Usuário

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

### Alerta 2 — Instabilidade Técnica do Endpoint

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

**Contexto:** `schema_validation_failure_rate` acima de 8% indica que o LLM está quebrando o contrato de Structured Output com frequência, comprometendo toda a camada de HITL e de citação de fonte.

---

### Alerta 3 — Sobrecarga da Fila HITL

**Regra:**
```
SE hitl_queue_depth > 20 itens pendentes sem revisão
E tempo_mais_antigo_na_fila > 45 minutos
→ Disparar aviso no canal #novatech-ai-atendentes (Teams)
→ Notificar os 5 atendentes-piloto para priorizar revisão da fila
→ Se hitl_queue_depth > 40 itens: escalar para Delivery Manager
   com status "HITL inoperante — risco de SLA de 30min quebrado"
```

**Contexto:** Uma fila HITL saturada significa que usuários estão aguardando respostas de baixa confiança sem resolução — na prática equivale ao bot estar offline para um subconjunto de queries.

---

## C. Fluxo de Feedback Loop Simplificado

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
          ┌──────────────┴──────────────┬──────────────┐
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

**Tempo médio esperado do ciclo:** feedback negativo → correção em produção:
- Categorias B e C: **3 a 5 dias úteis**
- Categoria A (ajuste de prompt): até **7 dias** (exige ciclo de avaliação mais cuidadoso)
