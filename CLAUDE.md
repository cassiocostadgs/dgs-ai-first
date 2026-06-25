# CLAUDE.md — dgs-ai-first

## O que é este projeto

Projeto de treinamento da iniciativa **AI-First do Grupo DB1** (Semana 4 - DGS). Não é um codebase de produção — é um framework de aprendizado estruturado em cenários, onde o participante age como Delivery Manager sênior de uma equipe que está construindo um assistente de IA para a empresa fictícia **NovaTech**.

Cada cenário produz entregáveis documentais (Markdown) que simulam artefatos reais de projetos de IA: especificações SDD, matrizes de risco, critérios de go-live, planos de rollback, dashboards de observabilidade.

---

## Caso de Uso: NovaTech

- **Empresa:** NovaTech — logística, 1.200 funcionários, 45 atendentes de SAC.
- **Problema:** busca média de 12 minutos por chamada em 3 fontes documentais fragmentadas (~1.250 docs).
- **Solução em construção:** assistente de IA integrado ao Microsoft Teams + SharePoint para reduzir busca para < 2 minutos.
- **Stack definida:** TypeScript/Node, React, Bicep, Azure OpenAI, Azure AI Search, Azure Bot Framework SDK.
- **Ferramentas do time:** Claude (chat), GitHub Copilot, Claude Cowork, Claude Design.
- **Orçamento/plataforma:** Azure AI Services + Microsoft 365 E3.

O assistente usa pipeline RAG com 847 documentos indexados no Azure AI Search. Respostas seguem Structured Output (JSON estrito) com campos obrigatórios: `answer`, `source_document`, `confidence_score`, `chunk_ids`.

---

## Estrutura do Repositório

```
dgs-ai-first/
├── CLAUDE.md                        # este arquivo
├── cenario.md                       # âncora narrativa da fase 1 — leia antes de qualquer tarefa
├── anexos/
│   ├── anexo-a-documentacao-simulada-novatech.md   # 5 docs simulados da NovaTech (POL-001, PROC-042, SLA-2024...)
│   ├── anexo-b-chunks-referencia-rag.md            # chunks de referência do pipeline RAG
│   ├── anexo-c-estrutura-repositorio.md            # blueprint do repositório de implementação futuro
│   └── grupo-db1-Contexto-knowledge-v1.md          # contexto institucional do Grupo DB1
└── entregaveis/
    ├── cenario-1/   # Fase 1: entendimento, matriz de riscos, alinhamento de stakeholders
    ├── cenario-2/   # Fase 2: workflow AI-first, AGENTS.md, governança SDD
    └── cenario-3/   # Fase 3: go-live, observabilidade, rollback, relatório semanal
```

Cada pasta de cenário contém apenas arquivos `.md`. Nunca criar subpastas dentro de cenários sem instrução explícita.

---

## AGENTS.md — Regras de Governança

O arquivo principal de governança está em `entregaveis/cenario-2/AGENTS.md`. É a "constituição" do projeto — define o que agentes de IA (Claude Code, GitHub Copilot) podem e não podem fazer.

**Regras críticas (nunca violar):**
- Specs vivem exclusivamente em `/docs/specs/<module-id>/` (no repositório de implementação futuro).
- Toda spec segue estrutura SDD: `requirements.md` → `plan.md` → `tasks.md`.
- Modificações pós-gate exigem RFC formal aprovado pelo Tech Lead.
- Sem secrets hardcoded. Sem dependências não documentadas.
- Validação de inputs obrigatória via **Zod** (TypeScript). Sem exceções.
- Logs nunca devem conter PII de atendentes (`user_id`, `agent_name`, `query_content` devem ser mascarados).
- System prompts: máximo 4K tokens. Histórico de conversa: máximo 3 turnos. Chunks por busca: máximo 5.

**Gates de validação obrigatórios:**
1. `requirements.md` → `plan.md` — aprovação: Product Specialist + Tech Lead
2. `plan.md` → implementação — aprovação: Tech Lead
3. Merge de PR — aprovação: Tech Lead + 1 peer
4. Deploy — aprovação: QA + Tech Lead

---

## Convenções de Trabalho

**Idioma:**
- Código, comentários técnicos, commits, nomes de variáveis: **inglês**.
- Comunicação de negócio, entregáveis para stakeholders, documentação executiva: **português (BR)**.

**Nomenclatura de arquivos:**
- Kebab-case, sempre em português descritivo: `criterios-go-live.md`, `plano-rollback.md`.
- Sem prefixos numéricos a não ser que já existam na pasta (cenário 1 usa `01-`, `02-`...).

**Commits:**
- Mensagens curtas em português descrevendo o entregável: `up ex 2.3`, `up evid`.

**Novos entregáveis:**
- Sempre criar dentro da pasta de cenário correspondente (`entregaveis/cenario-X/`).
- Um arquivo por entregável. Sem consolidar múltiplos entregáveis em um único arquivo.

---

## Framework de Harness (5 Camadas)

Referência conceitual central do projeto. Todo critério de qualidade e governança se organiza por essas camadas:

1. **Tool Orchestration** — coordenação de ferramentas e agentes (pipeline RAG, endpoint de query).
2. **Verification Loops** — validação automática de outputs (Zod schema, golden dataset, accuracy ≥ 85%).
3. **Context & Memory** — manutenção de contexto e frescor da base documental (re-indexação, `is_stale`).
4. **Guardrails** — limites do sistema e pontos de HITL (confidence_score < 0.70 → fila de revisão, topic boundaries).
5. **Observability** — visibilidade em produção (métricas, alertas, logs sem PII, traces distribuídos).

---

## Estado Atual do Projeto (referência pós-go-live)

- Bot em staging com 5 atendentes-piloto no Teams.
- Linha de base histórica: 12% de respostas incorretas (alucinações + chunks desatualizados).
- Structured Output implementado com campos obrigatórios.
- HITL ativo: respostas com `confidence_score < 0.70` vão para fila de revisão (SLA 30 min).
- Testes de integração cobrem ~75% do código.
- Módulo de feedback gerado via Copilot precisou de remediação (violou AGENTS.md — sem Zod, logava PII).

---

## Documentos de Referência Prioritários

Antes de gerar qualquer entregável novo, leia nesta ordem:
1. `cenario.md` — contexto narrativo completo da NovaTech.
2. `entregaveis/cenario-2/AGENTS.md` — regras de governança do projeto.
3. `anexos/anexo-a-documentacao-simulada-novatech.md` — base documental real do assistente.
4. O entregável mais recente do cenário em que se está trabalhando.
