# Plano de Observabilidade e Ciclo de Melhoria — NovaTech AI Assistant

---

## A. O que vamos acompanhar

### 1. Uso — O assistente está sendo utilizado?

| Indicador | O que mede | Frequência |
|-----------|-----------|------------|
| Perguntas por dia | Total de perguntas feitas ao assistente no dia | Diária |
| Atendentes ativos por dia | Quantos dos 5 atendentes-piloto usaram o assistente no dia | Diária |
| Duração média da conversa | Quanto tempo em média dura uma sessão do atendente com o bot | Diária |
| Perguntas por atendente | Média de perguntas feitas por cada atendente ativo | Semanal |
| Horário de pico | Em qual horário do dia o assistente é mais utilizado | Diária |

### 2. Qualidade — O assistente está acertando?

| Indicador | O que mede | Frequência |
|-----------|-----------|------------|
| Taxa de avaliações negativas | % de respostas que o atendente marcou como "incorreta" ou "ruim" | Diária |
| Taxa de respostas enviadas para revisão humana | % de respostas que o assistente não teve confiança suficiente para entregar sozinho e foram para revisão de um atendente | Diária |
| Taxa de aprovação nas revisões | Das respostas que foram para revisão humana, quantas % foram aprovadas sem alteração | Semanal |
| Taxa de "não sei responder" | % de perguntas em que o assistente não encontrou nenhuma informação relevante na base e respondeu "não encontrado" | Diária |
| Nível médio de confiança | Média do grau de certeza que o assistente atribui às suas próprias respostas (escala de 0 a 1) | Diária |

### 3. Desempenho técnico — O assistente está respondendo rápido e sem erros?

| Indicador | O que mede | Frequência |
|-----------|-----------|------------|
| Tempo de resposta (P95) | O tempo que 95% das perguntas levam para ser respondidas, do início ao fim | Contínua |
| Tempo de busca na base de documentos | Quanto tempo leva só a parte de buscar a informação nos documentos | Contínua |
| Tempo de geração da resposta | Quanto tempo leva só a parte de o assistente redigir a resposta | Contínua |
| Taxa de respostas bloqueadas por formato inválido | % de respostas que o sistema descartou automaticamente por não virem com todos os campos obrigatórios (fonte, nível de confiança etc.) | Contínua |
| Taxa de erros do sistema | % de perguntas que resultaram em falha técnica do sistema, sem resposta nenhuma | Contínua |
| Taxa de novas tentativas automáticas | % de chamadas em que o sistema precisou tentar novamente por instabilidade de conexão | Diária |

### 4. Conteúdo — A base de conhecimento está cobrindo o que os atendentes precisam?

| Indicador | O que mede | Frequência |
|-----------|-----------|------------|
| Documentos mais consultados | Quais os 10 documentos da base que mais geraram respostas na semana | Semanal |
| Perguntas sem nenhuma resposta na base | % de perguntas em que o assistente não encontrou nenhum documento relevante — indica lacunas de conteúdo | Diária |
| Respostas com documentos desatualizados | % de respostas que usaram trechos de documentos já marcados como desatualizados | Diária |
| Principais temas sem cobertura | Agrupamento dos assuntos mais perguntados que a base não consegue responder | Semanal |
| Aproveitamento da base | % dos 847 documentos que foram consultados pelo menos uma vez na semana | Semanal |

---

## B. Quando o time é acionado automaticamente

### Alerta 1 — Queda na qualidade percebida pelos atendentes

**Quando dispara:**
Se mais de 15% das respostas entregues em um dia receberem avaliação negativa dos atendentes (considerando ao menos 20 perguntas no período).

**O que acontece:**
- Notificação automática enviada ao canal de incidentes do time no Teams
- Tech Lead, QA e Delivery Manager são avisados
- O time tem até 2 horas para dar a primeira resposta sobre o que está causando o problema

**Por que 15%:** nos testes internos já identificamos 12% de respostas incorretas. Um índice acima de 15% em produção indica que o assistente está piorando em relação ao que já era o pior cenário conhecido.

---

### Alerta 2 — Falha técnica do sistema

**Quando dispara:**
Se mais de 10% das perguntas resultarem em erro do sistema, **ou** se mais de 8% das respostas forem descartadas automaticamente por virem incompletas (sem fonte, sem nível de confiança) — dentro de qualquer janela de 15 minutos.

**O que acontece:**
- Notificação urgente enviada ao canal de incidentes no Teams
- Tech Lead e Delivery Manager são avisados imediatamente
- O time tem 30 minutos para resolver; se não resolver em 60 minutos, o Plano de Rollback é acionado (ver [plano-rollback.md](plano-rollback.md))

**Por que isso é grave:** quando as respostas chegam incompletas, todo o sistema de revisão humana e de citação de fonte para de funcionar — o atendente fica sem saber se pode confiar na resposta.

---

### Alerta 3 — Fila de revisão humana sobrecarregada

**Quando dispara:**
Se houver mais de 20 respostas aguardando revisão de atendente ao mesmo tempo, e a mais antiga estiver há mais de 45 minutos na fila sem ser avaliada.

**O que acontece:**
- Aviso enviado no canal dos atendentes-piloto no Teams pedindo que priorizem a revisão da fila
- Se a fila chegar a 40 itens, o Delivery Manager é avisado com status de urgência

**Por que isso importa:** cada resposta na fila significa um atendente esperando. Com uma fila travada, o bot na prática para de funcionar para uma parte dos usuários.

---

## C. Como um erro vira uma melhoria

O fluxo abaixo mostra o caminho completo desde o momento em que um atendente avalia uma resposta como incorreta até a correção ser publicada no assistente.

```
┌─────────────────────────────────────────────────────────────────────┐
│  ATENDENTE no Teams                                                 │
│  Recebe a resposta → clica em "👎 Resposta incorreta"               │
│  Pode deixar um comentário explicando o que estava errado           │
└────────────────────────┬────────────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────────────┐
│  REGISTRO AUTOMÁTICO                                                │
│  O sistema salva automaticamente: qual foi a pergunta, qual foi     │
│  a resposta, qual documento foi citado, qual o nível de confiança   │
│  e o comentário do atendente — sem guardar dados pessoais           │
└────────────────────────┬────────────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────────────┐
│  TRIAGEM DIÁRIA (QA — toda manhã às 9h)                            │
│  O time de QA revisa os erros do dia anterior e classifica          │
│  cada um em um dos 3 tipos:                                         │
│                                                                     │
│  [A] Resposta inventada — o assistente criou algo que não existe    │
│      nos documentos                                                 │
│  [B] Documento desatualizado — a informação existe, mas é de uma   │
│      versão antiga do documento                                     │
│  [C] Trecho errado — o documento certo existe, mas o assistente    │
│      usou a parte errada dele                                       │
└────────────────────────┬────────────────────────────────────────────┘
                         │
          ┌──────────────┴──────────────┬──────────────┐
          ▼                             ▼               ▼
   [A] Resposta               [B] Documento      [C] Trecho
     inventada                desatualizado         errado
          │                             │               │
          ▼                             ▼               ▼
  Responsável técnico         Product Specialist   Responsável
  ajusta as instruções        atualiza o           técnico ajusta
  que guiam o assistente.     documento na base    como o assistente
  O assistente passa a        de origem.           faz a busca dentro
  responder somente com       Responsável técnico  dos documentos.
  o que está nos docs,        atualiza a base      Testa se o trecho
  nunca inventando.           de conhecimento.     certo é encontrado.
  Testa em ambiente de        Testa resultado.     Testa resultado.
  homologação antes de
  publicar.
          │                             │               │
          └──────────────┬──────────────┘──────────────┘
                         ▼
┌─────────────────────────────────────────────────────────────────────┐
│  VALIDAÇÃO (QA — antes de publicar a correção)                     │
│  • Testa as 10 perguntas de referência: o assistente deve acertar  │
│    pelo menos 85%                                                   │
│  • Confirma que a pergunta que gerou o erro original agora é        │
│    respondida corretamente                                          │
└────────────────────────┬────────────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────────────┐
│  PUBLICAÇÃO E REGISTRO                                              │
│  • Correção publicada primeiro em homologação, validada pelos       │
│    atendentes-piloto, depois publicada em produção                  │
│  • O erro e a correção são registrados no Relatório Semanal        │
│    da semana seguinte como evidência de melhoria                   │
└─────────────────────────────────────────────────────────────────────┘
```

**Tempo médio esperado — do erro à correção publicada:**
- Documento desatualizado ou trecho errado: **3 a 5 dias úteis**
- Resposta inventada (requer ajuste nas instruções do assistente): **até 7 dias** — esse tipo de correção precisa de mais testes para garantir que não quebra outras respostas
