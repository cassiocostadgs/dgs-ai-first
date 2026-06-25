# Dashboard de Readiness do Go-Live — NovaTech AI Assistant

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

---

## Resumo Executivo

| Classificação | Total | 🔴 Vermelho | 🟡 Amarelo | 🟢 Verde |
|---------------|-------|------------|-----------|---------|
| BLOQUEANTE | 13 | 9 | 4 | 0 |
| DESEJÁVEL | 6 | 4 | 2 | 0 |

> **Alerta:** 9 critérios BLOQUEANTES em Vermelho representam risco crítico de go-live. Prioridade imediata: remediação do módulo de feedback (D+4) e implementação de Structured Output (D+6).
