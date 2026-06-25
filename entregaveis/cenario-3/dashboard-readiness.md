# Painel de Acompanhamento do Go-Live — Assistente de IA NovaTech

> **Referência:** 14 dias antes da demonstração para a diretoria.
> Os status refletem o estado real do projeto com base nos problemas já identificados.

| Área | O que precisa estar pronto | Classificação | Status | Responsável | Prazo |
|------|---------------------------|---------------|--------|-------------|-------|
| **Coordenação do Assistente** | Documentos da base registram data de atualização; trechos com mais de 180 dias são excluídos automaticamente | BLOQUEANTE | 🔴 Não iniciado | Tech Lead | D+5 |
| **Coordenação do Assistente** | Toda resposta entregue obrigatoriamente inclui fonte do documento, nível de confiança e trechos utilizados | BLOQUEANTE | 🔴 Não iniciado | Tech Lead | D+6 |
| **Coordenação do Assistente** | Quando não há conteúdo relevante na base, o assistente informa "não encontrado" em vez de inventar uma resposta | BLOQUEANTE | 🟡 Em andamento | Tech Lead | D+7 |
| **Coordenação do Assistente** | Falhas temporárias de conexão acionam até 3 novas tentativas automáticas antes de retornar erro | DESEJÁVEL | 🟡 Em andamento | Tech Lead | D+10 |
| **Verificação de Respostas** | Respostas sem campo de fonte são bloqueadas automaticamente antes de chegar ao atendente | BLOQUEANTE | 🔴 Não iniciado | Tech Lead | D+7 |
| **Verificação de Respostas** | Taxa de respostas bloqueadas por esse filtro está abaixo de 5% no ambiente de testes (últimas 24h) | BLOQUEANTE | 🔴 Não iniciado | QA | D+10 |
| **Verificação de Respostas** | As 10 perguntas mais frequentes têm resposta de referência definida; assistente acerta pelo menos 85% | BLOQUEANTE | 🟡 Em andamento | QA + Product Specialist | D+9 |
| **Verificação de Respostas** | Para perguntas críticas, o sistema gera duas respostas e verifica se concordam antes de entregar | DESEJÁVEL | 🔴 Não iniciado | Tech Lead | Pós-demo |
| **Base de Conhecimento** | Atualização de documentos na origem reflete na base em até 24 horas | BLOQUEANTE | 🟡 Em andamento | Tech Lead | D+5 |
| **Base de Conhecimento** | Todos os trechos desatualizados foram revisados e corrigidos ou removidos da base | BLOQUEANTE | 🔴 Não iniciado | Tech Lead + Product Specialist | D+8 |
| **Base de Conhecimento** | Respostas indicam a data do documento utilizado | DESEJÁVEL | 🔴 Não iniciado | Tech Lead | Pós-demo |
| **Proteções e Limites** | Respostas com confiança abaixo de 70% vão para fila de revisão humana com prazo de 30 minutos | BLOQUEANTE | 🔴 Não iniciado | Tech Lead + Delivery Manager | D+8 |
| **Proteções e Limites** | Perguntas fora do escopo do assistente recebem mensagem de encaminhamento automático para o time humano | BLOQUEANTE | 🟡 Em andamento | Tech Lead + Product Specialist | D+7 |
| **Proteções e Limites** | Módulo de feedback corrigido: validação de dados e proteção de informações pessoais dos atendentes | BLOQUEANTE | 🔴 Não iniciado | Tech Lead | D+4 |
| **Proteções e Limites** | Limite de 20 perguntas por hora por atendente para evitar sobrecarga do serviço | DESEJÁVEL | 🔴 Não iniciado | Tech Lead | D+12 |
| **Monitoramento** | Painel de monitoramento em tempo real ativo antes do go-live | BLOQUEANTE | 🟡 Em andamento | Tech Lead + Delivery Manager | D+9 |
| **Monitoramento** | Registros de interação auditados pelo QA: nenhum dado pessoal de atendente exposto | BLOQUEANTE | 🔴 Não iniciado | QA | D+8 |
| **Monitoramento** | Alertas automáticos configurados: time recebe notificação no Teams sem precisar olhar o painel | BLOQUEANTE | 🟡 Em andamento | Tech Lead | D+10 |
| **Monitoramento** | Rastreamento completo do caminho de cada interação para diagnóstico rápido de incidentes | DESEJÁVEL | 🔴 Não iniciado | Tech Lead | Pós-demo |

---

## Resumo para a Diretoria

| Classificação | Total | 🔴 Não iniciado | 🟡 Em andamento | 🟢 Concluído |
|---------------|-------|-----------------|-----------------|--------------|
| BLOQUEANTE | 13 | 9 | 4 | 0 |
| DESEJÁVEL | 6 | 4 | 2 | 0 |

> **Situação atual:** Nenhum critério bloqueante está concluído. Há 9 itens críticos ainda não iniciados, com prazo máximo de 10 dias para serem resolvidos. O item mais urgente é a **correção do módulo de feedback** (prazo D+4), pois envolve risco de privacidade de dados. O segundo item mais urgente é a **garantia de que toda resposta inclui a fonte do documento** (prazo D+6), que resolve diretamente o problema dos 12% de respostas incorretas identificado nos testes.
