# Plano de Rollback Simplificado — NovaTech AI Assistant

> Princípio: o rollback não é uma decisão de engenharia — é uma decisão de negócio com execução técnica padronizada. Toda ambiguidade sobre "quando acionar" elimina tempo precioso durante um incidente.

---

## Triggers de Alerta (Nível 1 — Monitoramento Contínuo)

Condições que acionam **alerta automático** e notificação imediata ao Tech Lead e Delivery Manager:

| ID | Trigger | Métrica | Threshold |
|----|---------|---------|-----------|
| T1 | Taxa de erro no endpoint | `error_rate` (HTTP 5xx em `/query`) | > 15% em janela de 15 min |
| T2 | Falha de validação de schema | `schema_validation_failure_rate` | > 10% em janela de 15 min |
| T3 | Fila HITL saturada | `hitl_queue_depth` (itens pendentes sem revisão) | > 30 itens acumulados |
| T4 | Queda de qualidade generalizada | `avg_confidence_score` | < 0.55 em janela de 30 min |
| T5 | Quebra de Structured Output | Bot retornando respostas sem campo `source_document` | Qualquer ocorrência confirmada |

Qualquer trigger ativo por mais de **10 minutos sem resolução** escala automaticamente para decisão de rollback.

---

## Dono da Decisão de Rollback

| Situação | Autoriza o rollback |
|----------|---------------------|
| Demo para diretoria em andamento | **Delivery Manager** (decisão imediata, sem necessidade de consulta) |
| Ambiente de staging/piloto (5 atendentes) | **Tech Lead** (pode acionar sem aprovação prévia do DM) |
| Produção ampliada (futuro) | **Delivery Manager** + sign-off do **Product Owner** |

A autorização é verbal ou via mensagem no canal do Teams do time. Não exige cerimônia. O critério é: *"o bot está causando mais dano do que a ausência do bot causaria?"*

---

## Ações de Rollback (Execução Técnica)

### Passo 1 — Desativar o bot no Teams (< 2 minutos)

```
Teams Admin Center → Manage apps → DGS AI Assistant → Status: Blocked
```

Efeito imediato: bot some do Teams para todos os atendentes-piloto. Nenhuma query nova é processada. Nenhum dado é perdido.

---

### Passo 2 — Reverter o endpoint para a última tag estável (< 5 minutos)

```bash
# Identificar última tag estável
git tag --list "stable-*"

# Opção A — revert via Git
git checkout stable-<versão>

# Opção B — revert via Azure CLI (preferencial em produção)
az webapp deployment source config-zip \
  --resource-group novatech-rg \
  --name dgs-api \
  --src ./releases/stable-<versão>.zip
```

> Alternativa: redeployar a partir da última release tag aprovada diretamente pelo portal Azure DevOps, sem necessidade de acesso manual ao servidor.

---

### Passo 3 — Confirmar estabilização (aguardar 3 minutos após o revert)

Verificar no dashboard de observability:
- `error_rate` < 5%
- `schema_validation_failure_rate` < 5%
- Endpoint respondendo com HTTP 200 em health check

---

### Passo 4 — Comunicar stakeholders

**Delivery Manager** envia mensagem padrão no canal de stakeholders:

> *"O assistente AI está temporariamente indisponível para manutenção preventiva. Previsão de retorno: [X horas]. Os atendentes devem utilizar o fluxo manual de consulta. Atualizações a cada 30 minutos."*

---

### Passo 5 — Post-mortem (dentro de 24h após o incidente)

Tech Lead documenta no repositório:
- Trigger acionado
- Root cause identificado
- Fix aplicado
- Critério objetivo de volta ao ar

O documento é a evidência para que a diretoria saiba que o time tem controle do processo.
