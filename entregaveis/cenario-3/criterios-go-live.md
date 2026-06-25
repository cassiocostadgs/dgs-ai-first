# Critérios de Go-Live por Camada de Harness — NovaTech AI Assistant

> Escopo de referência: demo para diretoria em D+14. Critérios [BLOQUEANTE] são pré-condição para qualquer exposição ao público executivo.

---

## Camada 1 — Tool Orchestration

**Problema atacado:** recuperação de chunks incorretos e documentos desatualizados contribuindo para os 12% de erro.

| # | Critério | Classificação |
|---|----------|--------------|
| 1.1 | O pipeline de ingestão deve indexar metadados de `last_updated_at` em cada documento. Chunks oriundos de documentos com vigência expirada (threshold: >180 dias sem atualização) devem receber flag `is_stale: true` e ser excluídos do retrieval por padrão. | **[BLOQUEANTE]** |
| 1.2 | O endpoint `POST /query` deve retornar exclusivamente Structured Output em JSON estrito, validado contra schema fixo com os campos obrigatórios: `answer` (string), `source_document` (string, non-null), `confidence_score` (float 0–1), `chunk_ids` (array). Respostas em texto livre devem ser **rejeitadas na camada de orquestração** antes de chegar ao Teams. | **[BLOQUEANTE]** |
| 1.3 | Implementar fallback explícito: se o Azure AI Search retornar zero chunks com score acima do threshold mínimo de relevância (ex: cosine similarity < 0.75), o orquestrador deve retornar uma resposta-padrão de "informação não encontrada na base" — nunca invocar o LLM com contexto vazio. | **[BLOQUEANTE]** |
| 1.4 | Mecanismo de retry com exponential backoff (3 tentativas, delay de 500ms/1s/2s) para falhas transitórias de chamada ao Azure AI Search e ao endpoint do modelo. | **[DESEJÁVEL]** |

---

## Camada 2 — Verification Loops

**Problema atacado:** 12% de erro e ausência de garantia estrutural de que campos obrigatórios existem na resposta.

| # | Critério | Classificação |
|---|----------|--------------|
| 2.1 | Implementar validation loop pós-geração via **Zod schema** (TypeScript) ou equivalente: toda resposta do LLM é parseada e validada antes de ser despachada. Se `source_document` for `null`, `undefined` ou string vazia, a resposta é **descartada automaticamente**, um erro estruturado é logado (`RESPONSE_VALIDATION_FAILED`) e o usuário recebe mensagem-padrão de fallback. Sem exceções. | **[BLOQUEANTE]** |
| 2.2 | A taxa de `RESPONSE_VALIDATION_FAILED` em staging deve ser < 5% em janela de 24h antes do go-live. Acima disso, o critério 1.2 ainda não está funcionando corretamente no prompt/schema e o go-live não pode ocorrer. | **[BLOQUEANTE]** |
| 2.3 | Para as top-10 perguntas mais frequentes identificadas nos testes-piloto, criar golden dataset com respostas esperadas. Rodar avaliação automatizada (ex: LLM-as-judge ou exact match em campos estruturados) e exigir accuracy ≥ 85% nesse conjunto antes do go-live. | **[BLOQUEANTE]** |
| 2.4 | Self-consistency check para queries classificadas como alta sensibilidade (ex: queries sobre processos críticos de negócio): gerar 2 respostas independentes e comparar convergência antes de retornar. Implementar após a demo. | **[DESEJÁVEL]** |

---

## Camada 3 — Context & Memory

**Problema atacado:** recuperação de documentos desatualizados impactando qualidade das respostas.

| # | Critério | Classificação |
|---|----------|--------------|
| 3.1 | O pipeline de ingestão deve ser reexecutado (re-indexação incremental) sempre que um documento-fonte for atualizado no repositório de origem. O índice do Azure AI Search deve refletir o estado atual da base documental com lag máximo de 24h. Chunks com `is_stale: true` devem ser auditados e atualizados ou removidos antes do go-live. | **[BLOQUEANTE]** |
| 3.2 | O contexto enviado ao LLM deve incluir explicitamente o campo `document_date` de cada chunk, para que o modelo seja instruído no system prompt a sinalizar quando estiver utilizando fontes com data anterior a um threshold. | **[DESEJÁVEL]** |
| 3.3 | Implementar memória de curto prazo por sessão no bot do Teams (janela de conversa), para evitar que o assistente contradiga respostas anteriores dentro da mesma thread. | **[DESEJÁVEL]** |

---

## Camada 4 — Guardrails

**Problema atacado:** ausência de HITL, furo de governança no módulo de feedback, falta de limites de tópico.

| # | Critério | Classificação |
|---|----------|--------------|
| 4.1 | **HITL obrigatório — ponto de interceptação por confiança:** Toda resposta com `confidence_score < 0.70` deve ser interceptada antes de ser exibida no Teams. O sistema deve enfileirar a resposta em uma fila de revisão visível aos 5 atendentes-piloto, com SLA de resposta de 30 minutos para validação/descarte. O atendente pode aprovar, editar ou descartar. O usuário final recebe mensagem: *"Consultando um especialista, aguarde."* Respostas sem revisão que excedam o SLA retornam mensagem-padrão de escalação. | **[BLOQUEANTE]** |
| 4.2 | **Topic boundary hardcoded:** O AGENTS.md define escopo. Queries que ativem classificadores de tópico fora do escopo (ex: aconselhamento jurídico, médico, financeiro de alta complexidade) devem retornar resposta-padrão pré-aprovada de escalação humana, sem invocar o LLM de geração. Essa regra não pode ser sobrescrita por instrução do usuário no chat. | **[BLOQUEANTE]** |
| 4.3 | **Remediação do módulo de feedback (furo de governança):** O módulo gerado via Copilot deve ser refatorado e aprovado pelo Tech Lead antes do merge em main. Checklist obrigatório: (a) validação de inputs via Zod, (b) remoção de todos os `console.log` com campos de dados do atendente, (c) substituição por log estruturado com PII masking para campos como `user_id`, `agent_name`, `query_content`. PR deve passar em code review com referência explícita ao AGENTS.md. | **[BLOQUEANTE]** |
| 4.4 | Rate limiting por usuário no Teams: máximo de 20 queries/hora por atendente. Proteção contra abuso acidental que degradaria o serviço para os outros usuários-piloto. | **[DESEJÁVEL]** |

---

## Camada 5 — Observability

**Problema atacado:** sem visibilidade, qualquer problema em produção será descoberto tarde demais.

| # | Critério | Classificação |
|---|----------|--------------|
| 5.1 | Dashboard de métricas em produção ativo **antes** do go-live, com as seguintes métricas em tempo real: `query_volume`, `avg_confidence_score`, `hitl_queue_depth`, `schema_validation_failure_rate`, `latência P95 (ms)`, `error_rate (HTTP 5xx)`. Visível para Delivery Manager e Tech Lead. | **[BLOQUEANTE]** |
| 5.2 | Todos os logs de interação devem estar em conformidade com AGENTS.md: sem PII de atendente (name, email, query raw text sem masking). Auditoria de log a ser executada pelo QA antes do go-live como critério de aceite. | **[BLOQUEANTE]** |
| 5.3 | Alerta automático configurado: se `schema_validation_failure_rate > 10%` ou `error_rate > 15%` em janela deslizante de 15 minutos, notificação automática para o canal do Teams do time técnico + Delivery Manager. Alerta não pode depender de alguém olhar o dashboard manualmente. | **[BLOQUEANTE]** |
| 5.4 | Trace distribuído (ex: Application Insights ou OpenTelemetry) correlacionando `query_id → retrieval → geração → validação → entrega`, para diagnóstico rápido de root cause em caso de incidente. | **[DESEJÁVEL]** |
