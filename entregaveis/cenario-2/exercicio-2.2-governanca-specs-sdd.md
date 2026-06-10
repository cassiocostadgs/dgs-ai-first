# Exercício 2.2 — Governança de Specs como Contratos Executáveis
**NovaTech Assistant | Spec Driven Development | Entregável do Delivery Manager**
**Projeto:** novatech-assistant | **Data:** 2026-06-10 | **Autor:** Delivery Manager

---

## PARTE 1 — Documento de Governança de Specs

### 1.1 Princípio Fundacional

No projeto NovaTech, os arquivos `requirements.md`, `plan.md` e `tasks.md` são **contratos executáveis**, não documentação auxiliar. Eles têm o mesmo rigor de versionamento e controle de qualidade que o código de produção. Nenhum agente de IA inicia trabalho em um artefato cujo contrato antecedente não tenha sido aprovado por um humano responsável.

---

### 1.2 Matriz de Responsabilidade (RACI)

| Artefato | Product Specialist | Tech Lead | Dev Pleno | Dev Sênior | QA | Delivery Manager |
|---|---|---|---|---|---|---|
| `requirements.md` — Criação | **R** | C | I | I | C | A |
| `requirements.md` — Aprovação | A | **R** | I | I | C | A |
| `plan.md` — Criação | C | **R** | C | C | I | I |
| `plan.md` — Aprovação | **R** | A | I | I | I | I |
| `tasks.md` — Criação | I | C | **R** | **R** | I | I |
| `tasks.md` — Aprovação | I | **A/R** | C | C | I | I |
| Registro de mudança (RFC) | C | C | C | C | C | **R** |
| Aprovação final de RFC | C | **R** | I | I | C | A |

> **Legenda:** R = Responsável pela execução · A = Accountable (dono da decisão) · C = Consultado · I = Informado

**Regras complementares:**
- O `requirements.md` nunca é criado por agente de IA sem revisão humana prévia. Claude (chat) pode **rascunhar**, mas o Product Specialist assina antes de qualquer avanço.
- O `plan.md` é de autoria exclusiva do Tech Lead; GitHub Copilot e Claude (chat) podem sugerir estrutura, mas o conteúdo técnico é responsabilidade humana.
- O `tasks.md` pode ser gerado com apoio do GitHub Copilot, desde que o Tech Lead valide cada task antes do início da implementação (Gate 2 do workflow).

---

### 1.3 Convenção de Nomenclatura e Localização no Repositório

#### Estrutura de diretórios

```
novatech-assistant/
└── docs/
    └── specs/
        ├── 01-pipeline-ingestao/
        │   ├── requirements.md
        │   ├── plan.md
        │   └── tasks.md
        ├── 02-api-busca/
        │   ├── requirements.md
        │   ├── plan.md
        │   └── tasks.md
        ├── 03-api-feedback/
        │   ├── requirements.md
        │   ├── plan.md
        │   └── tasks.md
        ├── 04-bot-teams/
        │   ├── requirements.md
        │   ├── plan.md
        │   └── tasks.md
        └── 05-painel-web/
            ├── requirements.md
            ├── plan.md
            └── tasks.md
```

#### Convenções de nomenclatura

| Regra | Exemplo válido | Exemplo inválido |
|---|---|---|
| Prefixo numérico de 2 dígitos para ordenação | `01-pipeline-ingestao/` | `pipeline/` |
| Nomes em kebab-case minúsculo | `02-api-busca/` | `APIBusca/` |
| Três arquivos fixos por módulo | `requirements.md`, `plan.md`, `tasks.md` | `spec-v2.md`, `tasks-final.md` |
| Sem sufixo de versão no nome do arquivo | `requirements.md` | `requirements-v3.md` |
| Versão controlada exclusivamente pelo Git | — | Arquivos como `requirements_old.md` |

> **Princípio:** o nome do arquivo nunca muda. A versão vive no histórico do Git, não no nome.

---

### 1.4 Versionamento Local com Git

#### Estratégia de branches para specs

```
main
└── spec/[modulo]-[artefato]        ← branch de trabalho para cada spec
    Exemplos:
    spec/01-pipeline-ingestao-requirements
    spec/02-api-busca-plan
    spec/04-bot-teams-tasks
```

Após aprovação humana, a branch é mergeada em `main` via Pull Request local (ou merge direto com mensagem de commit padronizada).

#### Padrão de mensagens de commit

```
[spec][modulo] acao: descricao curta

Formato:
  [spec][<id-modulo>] <acao>: <descricao em até 72 caracteres>

Ações válidas:
  create    → primeira versão do artefato
  update    → alteração de conteúdo (requer RFC se em andamento)
  approve   → commit de aprovação humana (assinatura de gate)
  rfc       → registro de solicitação de mudança

Exemplos:
  [spec][01-pipeline-ingestao] create: requirements inicial do pipeline
  [spec][02-api-busca] approve: plan aprovado pelo Tech Lead - gate 1 ok
  [spec][04-bot-teams] rfc: RFC-004 ajuste no limite de histórico de turnos
  [spec][03-api-feedback] update: tasks revisadas após RFC-002 aprovado
```

#### Tags de marcos de spec

Ao final de cada fase de aprovação de um módulo, o Tech Lead cria uma tag leve:

```
git tag spec/01-pipeline-ingestao/requirements-aprovado
git tag spec/01-pipeline-ingestao/plan-aprovado
git tag spec/01-pipeline-ingestao/tasks-aprovado
```

Isso cria pontos de restauração imutáveis no histórico local, permitindo que qualquer membro do time saiba exatamente o estado do contrato no momento em que a implementação foi autorizada.

---

## PARTE 2 — Board de Tracking de Specs (Kanban)

### Status Atual do Pipeline de Specs — Sprint Corrente

| # | Módulo | Rascunho | Em Revisão | Aprovada | Em Implementação | Validada |
|---|---|:---:|:---:|:---:|:---:|:---:|
| 01 | Pipeline de Ingestão | — | — | — | **✦** | — |
| 02 | API de Busca | — | — | **✦** | — | — |
| 03 | API de Feedback | — | **✦** | — | — | — |
| 04 | Bot do Teams | — | — | — | — | **✦** |
| 05 | Painel Web | **✦** | — | — | — | — |

---

### Detalhamento por Módulo

| Módulo | Coluna Atual | Artefato Vigente | Nota de Status |
|---|---|---|---|
| **01 — Pipeline de Ingestão** | Em Implementação | `tasks.md` | `tasks.md` aprovado (tag: `spec/01/tasks-aprovado`). Devs executando tasks 3–7 de 12. Gargalo identificado: task #5 (chunking de tabelas) aguarda decisão do Tech Lead sobre estratégia de splitting. **Ação do DM:** bloquear task #5 no Azure DevOps até resolução. |
| **02 — API de Busca** | Aprovada | `plan.md` | `requirements.md` e `plan.md` aprovados. `tasks.md` ainda não gerado — Dev Sênior inicia decomposição hoje com suporte do Copilot. Gate 2 agendado para amanhã. **Ação do DM:** confirmar disponibilidade do Tech Lead para Gate 2. |
| **03 — API de Feedback** | Em Revisão | `requirements.md` | Product Specialist entregou `requirements.md` v1. Tech Lead em revisão técnica — prazo do Gate 1: 24h a partir de ontem. **Ação do DM:** acionar Tech Lead; gate vence hoje às 17h. |
| **04 — Bot do Teams** | Validada | `tasks.md` | Todos os três artefatos aprovados e implementação concluída. QA validou cobertura de testes (Gate 4 aprovado). Módulo encerrado nesta sprint. **Ação do DM:** arquivar branch de spec, criar tag de módulo completo. |
| **05 — Painel Web** | Rascunho | `requirements.md` | Product Specialist iniciou rascunho do `requirements.md` com apoio do Claude Design para protótipos de interface. Sem prazo de entrega para Gate 1 ainda definido. **Ação do DM:** alinhar data de entrega na próxima daily. |

---

### Leitura do Gargalo pelo DM

```
Distribuição atual:
  Validada          ████░░░░░░  1 módulo  (20%)
  Em Implementação  ████░░░░░░  1 módulo  (20%)
  Aprovada          ████░░░░░░  1 módulo  (20%)
  Em Revisão        ████░░░░░░  1 módulo  (20%)
  Rascunho          ████░░░░░░  1 módulo  (20%)

Ponto crítico: Gate 1 do módulo 03 vence hoje.
               Task #5 do módulo 01 bloqueada — requer decisão técnica.
```

> **Ação prioritária do DM:** desbloqueio do Gate 1 (módulo 03) é o item com maior risco de cascata — um atraso aqui empurra o início de `plan.md` e potencialmente conflita com a disponibilidade do Tech Lead quando o módulo 02 entrar em Gate 2 amanhã.

---

## PARTE 3 — Processo de Gestão de Mudanças (Change Management)

### 3.1 Princípio de Operação

Specs aprovadas são **imutáveis sem processo formal**. Qualquer alteração após o início da implementação (pós-Gate 2) dispara automaticamente um RFC (*Request for Change*). O objetivo não é criar burocracia, mas proteger o trabalho já realizado pelos agentes de IA e evitar código órfão que ninguém consegue rastrear.

---

### 3.2 Gatilho — Quem pode solicitar e como documentar

**Qualquer membro do time** pode identificar a necessidade de mudança. A solicitação é formalizada da seguinte forma:

1. O solicitante abre um item no Azure DevOps do tipo **RFC** com o template abaixo:
2. O item recebe um identificador sequencial: `RFC-NNN`.
3. O Delivery Manager é notificado automaticamente (via regra de alerta no board).

```markdown
## RFC-NNN — [Título da Mudança]

**Módulo afetado:** [nome do módulo]
**Artefato alvo:** [ ] requirements.md  [ ] plan.md  [ ] tasks.md
**Solicitante:** [papel + nome]
**Data:** ____/____/______
**Status atual da implementação:** [ex: tasks 4–7 de 12 concluídas]

### Descrição da mudança solicitada
[O que precisa mudar e por quê — foco no impacto de produto ou técnico]

### Alternativa de não fazer a mudança
[O que acontece se a mudança for rejeitada — contexto para a decisão]
```

---

### 3.3 Análise de Impacto

Antes de qualquer aprovação, o Tech Lead conduz a análise de impacto em até **4 horas**. A análise cobre obrigatoriamente os três eixos abaixo:

| Eixo | Perguntas-chave | Responsável |
|---|---|---|
| **Impacto no código já gerado** | Quantas tasks já concluídas são invalidadas? Existe código órfão? Há risco de regressão em módulos integrados? | Tech Lead |
| **Impacto nas ferramentas de IA** | O contexto que o Copilot/Claude recebeu nas tasks anteriores está desatualizado? As tasks geradas por IA assumiram premissas que mudarão? | Dev Sênior (com apoio do Tech Lead) |
| **Impacto no prazo e capacidade** | Qual o delta de esforço estimado? Compromete a sprint atual? Afeta módulos dependentes no board? | Delivery Manager |

O resultado da análise é registrado diretamente no item RFC no Azure DevOps antes da reunião de aprovação.

---

### 3.4 Fluxo de Aprovação

```
Solicitante abre RFC no Azure DevOps
          │
          ▼
Tech Lead realiza análise de impacto (≤ 4h)
          │
          ▼
DM convoca reunião de decisão (≤ 1h, síncrona ou assíncrona por comentário no RFC)
          │
    ┌─────┴─────┐
    │           │
APROVADO    REJEITADO
    │           │
    │           └──► DM fecha RFC com justificativa registrada
    │                Trabalho continua com spec original
    ▼
Tech Lead atualiza o(s) artefato(s) afetados no repositório
          │
          ▼
Commit com mensagem padrão:
[spec][NN-modulo] rfc: RFC-NNN aplicado — <resumo da mudança>
          │
          ▼
DM atualiza status do módulo no board de tracking
```

**Quórum mínimo para aprovação de RFC:**
- Mudanças em `requirements.md`: Product Specialist + Tech Lead
- Mudanças em `plan.md`: Tech Lead (sozinho, com ciência do PS)
- Mudanças em `tasks.md`: Tech Lead (sozinho)

---

### 3.5 Reset de Tasks — O que acontece com `tasks.md` e com os agentes de IA

Esta é a etapa mais crítica do processo. Um RFC aprovado que afeta tasks em andamento exige um protocolo explícito para evitar trabalho fantasma.

#### Classificação das tasks impactadas

O Tech Lead classifica cada task do `tasks.md` vigente em uma das três categorias:

| Categoria | Definição | Ação |
|---|---|---|
| **Intacta** | A mudança não afeta esta task. | Nenhuma ação — task continua no fluxo normal. |
| **Revisada** | A task ainda é válida, mas precisa de ajuste de critério ou parâmetro. | Tech Lead edita a task no arquivo. Copilot/Dev revisitam apenas os trechos afetados. |
| **Invalidada** | A task contradiz o novo contrato. O código gerado por ela é incompatível. | Task marcada como `[CANCELADA - RFC-NNN]` no `tasks.md`. Código correspondente é removido ou revertido em commit dedicado antes de gerar a task substituta. |

#### Protocolo de reset para agentes de IA

Quando tasks são invalidadas, o seguinte protocolo previne código órfão:

1. **Parada imediata:** o Dev responsável para todo trabalho nas tasks impactadas antes de aplicar o RFC.
2. **Commit de reversão:** o código das tasks invalidadas é revertido em um commit isolado com mensagem:
   ```
   revert: remove código de tasks invalidadas por RFC-NNN
   ```
3. **Atualização do `tasks.md`:** o arquivo é atualizado com as tasks revisadas e as novas tasks geradas a partir do RFC aprovado.
4. **Novo contexto para IA:** ao retomar o trabalho com Copilot ou Claude, o Dev fornece explicitamente o `tasks.md` atualizado como contexto — **nunca continua uma sessão de agente que foi iniciada com o contrato anterior**. Sessões de geração de código são stateless em relação ao contrato; o contrato correto precisa ser injetado a cada nova sessão.
5. **Registro de lição aprendida:** o Delivery Manager anota no Azure DevOps (campo "Observações" do RFC) se a mudança era evitável com uma spec mais robusta no Gate 1 — insumo para melhoria do processo.

#### Exemplo de `tasks.md` após RFC

```markdown
## Tasks — Módulo 02: API de Busca
_Última atualização: RFC-003 aplicado em 2026-06-10_

| ID | Descrição | Status | Observação |
|----|-----------|--------|------------|
| T-01 | Criar endpoint POST /query | Concluída | Intacta |
| T-02 | Integrar Azure AI Search (top-5 chunks) | Concluída | Intacta |
| T-03 | Implementar ordenação por vigência | Em andamento | Revisada (RFC-003: adicionar log de metadado) |
| T-04 | Retornar campo `source_doc_id` na resposta | Pendente | Nova (gerada pelo RFC-003) |
| ~~T-05~~ | ~~Retornar score de relevância bruto~~ | CANCELADA | Invalidada por RFC-003 — removida do escopo |
```

---

### 3.6 Salvaguardas para o Time Enxuto

Para um time de 6 pessoas, o processo acima foi calibrado para não criar overhead excessivo:

| Salvaguarda | Descrição |
|---|---|
| **RFC assíncrono por padrão** | A aprovação pode ocorrer via comentário no Azure DevOps em até 4h, sem reunião obrigatória — reunião síncrona apenas se houver discordância. |
| **RFC dispensado para typos e clareza** | Correções de texto que não alteram nenhum critério técnico ou de aceite não disparam RFC — o Tech Lead commita diretamente com mensagem `[spec] fix: corrige redação em tasks.md`. |
| **Limite de RFCs por sprint** | Se o número de RFCs abertos superar 3 em uma sprint, o DM convoca uma retrospectiva de spec — o problema está na qualidade do Gate 1, não na implementação. |
| **Template pré-preenchido no DevOps** | O template de RFC fica salvo como item recorrente no Azure DevOps, reduzindo o tempo de abertura para menos de 5 minutos. |

---

*Documento gerado em conformidade com o fluxo SDD adotado pelo time NovaTech. Revisão e aprovação: Delivery Manager + Tech Lead antes da primeira sprint de implementação.*
