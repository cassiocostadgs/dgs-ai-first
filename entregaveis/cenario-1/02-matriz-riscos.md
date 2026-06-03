# Matriz de Riscos Técnicos — Assistente de IA NovaTech
**Projeto:** Assistente de IA para Atendimento ao Cliente
**Preparado por:** DB1 Global Software — Gerência de Projetos & Consultoria de IA
**Data:** Junho/2026 | **Fase:** Kickoff — Análise de Riscos

---

## Matriz de Riscos Técnicos

---

**Risco:** O assistente inventa procedimentos que não existem

Quando um atendente faz uma pergunta cuja resposta não está na documentação, o modelo tende a gerar uma resposta plausível em vez de admitir que não sabe. Na base da NovaTech, há lacunas confirmadas: não existe nenhum documento formal sobre carga danificada em trânsito nem sobre frete abaixo de 500kg. O assistente pode inventar regras para esses casos e o atendente não terá como identificar que a informação é falsa.

**Probabilidade:** Alta
**Impacto:** Alto
**Criticidade:** Alta

**Mitigação:** Configurar o assistente para responder com uma mensagem padrão quando não encontrar a informação — ex: *"Não localizei essa informação na documentação. Encaminhe ao setor responsável."* — bloqueando qualquer resposta baseada em inferência.

---

**Risco:** Documentos contraditórios geram respostas erradas

A NovaTech tem duas versões ativas do mesmo procedimento de cálculo de frete (PROC-042 v1 e v2), com valores diferentes para os mesmos campos: o multiplicador da região Norte é 1.6 na versão antiga e 1.8 na nova; o prazo adicional de entrega é +2 dias em uma e +3 na outra. Nenhuma das duas está marcada como obsoleta. Se o assistente encontrar as duas ao mesmo tempo, pode misturar os valores e gerar um cálculo incorreto — apresentado com a mesma confiança de uma resposta correta.

**Probabilidade:** Alta
**Impacto:** Alto
**Criticidade:** Alta

**Mitigação:** Implementar, no processo de ingestão dos documentos, um campo de data de vigência para cada arquivo. Quando dois documentos do mesmo tipo forem encontrados, o sistema sempre prioriza o mais recente e informa ao atendente qual versão está sendo utilizada.

---

**Risco:** Documentos informais tratados como fontes confiáveis

A base da NovaTech inclui documentos com graus de confiabilidade muito diferentes. O FAQ de atendimento, por exemplo, é mantido informalmente pelo próprio time, sem validação de Compliance ou Operações — e contém informações que contradizem ou não têm respaldo nas políticas oficiais. O assistente, sem orientação, trata todas as fontes como igualmente confiáveis e pode embasar respostas críticas em registros informais.

**Probabilidade:** Alta
**Impacto:** Médio
**Criticidade:** Alta

**Mitigação:** Classificar cada documento por nível de confiabilidade (oficial, referência, informal) antes da ingestão. Configurar o assistente para deixar visível para o atendente qual tipo de fonte originou a resposta — e para não usar documentos informais em perguntas classificadas como críticas.

---

**Risco:** A expectativa da diretoria está acima do que a tecnologia entrega hoje

A diretoria espera que o assistente resolva a busca por documentação em menos de 2 minutos de forma ampla e consistente. Para perguntas simples e diretas, essa meta é alcançável. Porém, consultas que cruzam múltiplos temas — como calcular o frete de uma carga perigosa acima de 2.500kg para o Norte, com prazo de devolução e SLA de cliente Gold — exigem que o assistente raciocine sobre várias informações ao mesmo tempo, e a precisão cai. O risco é que o projeto seja avaliado com casos simples durante os testes e falhe nos cenários reais mais comuns.

**Probabilidade:** Média
**Impacto:** Alto
**Criticidade:** Alta

**Mitigação:** Levantar no discovery ao menos 50 perguntas reais feitas pelos atendentes, classificadas por nível de complexidade. Usar esse conjunto para definir, junto à diretoria, o que o assistente vai cobrir no go-live e o que ficará fora do escopo inicial — com metas de precisão documentadas e acordadas antes do desenvolvimento começar.

---

**Risco:** Volume elevado de fontes reduz a qualidade das respostas

Com ~1.250 fontes ingeridas, o assistente recebe um volume grande de trechos de documentos a cada consulta. Modelos de linguagem têm dificuldade de manter atenção em contextos muito longos: quando a informação relevante está no meio de muitos outros trechos, o modelo tende a ignorá-la ou confundi-la com outras. As 50 planilhas de frete são especialmente problemáticas — cada linha vira um fragmento de dado sem contexto, dificultando que o modelo identifique a informação certa.

**Probabilidade:** Alta
**Impacto:** Médio
**Criticidade:** Alta

**Mitigação:** Limitar o número de trechos enviados ao modelo por consulta (máximo de 5 trechos mais relevantes, não todos os recuperados). Converter as planilhas de frete em tabelas de texto estruturado durante a ingestão, para que o sistema consiga recuperar os dados certos com mais precisão.

---

*Documento preparado para uso interno — kickoff NovaTech / DB1 Global Software*
