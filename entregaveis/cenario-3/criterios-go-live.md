# Critérios de Go-Live — Assistente de IA NovaTech

> **Como ler este documento:** Cada critério está classificado como **[BLOQUEANTE]** ou **[DESEJÁVEL]**.
> - **BLOQUEANTE** = o assistente não pode ir ao ar para a diretoria sem que esse item esteja concluído.
> - **DESEJÁVEL** = pode ir ao ar, mas o item vira uma melhoria planejada para as semanas seguintes.

---

## Área 1 — Coordenação do Assistente
*Como o assistente busca informações e entrega respostas*

**Por que isso importa:** Durante os testes internos, 12% das respostas estavam erradas. Uma parte desses erros vem de o assistente buscar trechos de documentos errados ou desatualizados. Estes critérios garantem que a busca seja confiável antes do go-live.

| # | O que precisa estar funcionando | Classificação |
|---|--------------------------------|--------------|
| 1.1 | Cada documento da base de conhecimento deve registrar a data da última atualização. Documentos sem atualização há mais de 180 dias devem ser automaticamente excluídos das buscas, evitando que o assistente cite informações desatualizadas. | **[BLOQUEANTE]** |
| 1.2 | Toda resposta do assistente deve seguir um formato fixo e obrigatório, incluindo sempre: a resposta em si, o documento de origem, o nível de confiança da resposta e os trechos utilizados. Respostas que não tenham todos esses campos são bloqueadas automaticamente — nunca chegam ao atendente. | **[BLOQUEANTE]** |
| 1.3 | Quando o assistente não encontrar nenhum conteúdo relevante na base para responder uma pergunta, ele deve informar claramente "informação não encontrada" — nunca inventar uma resposta. | **[BLOQUEANTE]** |
| 1.4 | Se a conexão com a base de conhecimento falhar temporariamente, o sistema deve tentar novamente até 3 vezes antes de retornar um erro ao atendente. | **[DESEJÁVEL]** |

---

## Área 2 — Verificação Automática das Respostas
*Garantia de que nenhuma resposta sem fonte chega ao atendente*

**Por que isso importa:** Um dos problemas identificados é que o assistente pode gerar respostas sem citar a fonte do documento. Estes critérios criam uma camada automática que descarta qualquer resposta incompleta antes de ela ser exibida.

| # | O que precisa estar funcionando | Classificação |
|---|--------------------------------|--------------|
| 2.1 | Cada resposta gerada passa por uma verificação automática antes de ser entregue. Se o campo de documento de origem estiver vazio ou ausente, a resposta é descartada e o atendente recebe uma mensagem padrão de "não foi possível responder". Sem exceções. | **[BLOQUEANTE]** |
| 2.2 | No ambiente de testes, a taxa de respostas descartadas por essa verificação deve ser menor que 5% ao longo de 24 horas antes do go-live. Se estiver acima disso, o go-live não ocorre. | **[BLOQUEANTE]** |
| 2.3 | A equipe deve criar um conjunto de referência com as 10 perguntas mais frequentes dos atendentes e as respostas corretas esperadas. O assistente deve acertar pelo menos 85% desse conjunto antes de ir ao ar. | **[BLOQUEANTE]** |
| 2.4 | Para perguntas sobre processos críticos do negócio, o sistema gera duas respostas independentes e verifica se elas concordam antes de entregar. A ser implementado após a demo. | **[DESEJÁVEL]** |

---

## Área 3 — Atualização da Base de Conhecimento
*Garantia de que o assistente sempre consulta documentos vigentes*

**Por que isso importa:** Parte dos erros nos testes veio de o assistente citar versões antigas de documentos. Estes critérios garantem que a base de conhecimento esteja sempre atualizada.

| # | O que precisa estar funcionando | Classificação |
|---|--------------------------------|--------------|
| 3.1 | Sempre que um documento for atualizado na origem (SharePoint, repositório), a base de conhecimento do assistente deve refletir essa mudança em até 24 horas. Todos os trechos desatualizados devem ser revisados e corrigidos ou removidos antes do go-live. | **[BLOQUEANTE]** |
| 3.2 | As respostas do assistente devem indicar a data do documento utilizado, para que o atendente saiba se está consultando uma informação recente ou antiga. | **[DESEJÁVEL]** |
| 3.3 | Dentro de uma mesma conversa no Teams, o assistente deve lembrar o que disse anteriormente para não se contradizer na mesma thread. | **[DESEJÁVEL]** |

---

## Área 4 — Proteções e Limites do Assistente
*Controles que evitam respostas incorretas e protegem os dados dos atendentes*

**Por que isso importa:** Sem proteções, o assistente pode entregar respostas de baixa qualidade sem nenhuma revisão humana, responder sobre assuntos fora do seu escopo e expor dados dos atendentes indevidamente. Estes são os critérios de maior risco do projeto.

| # | O que precisa estar funcionando | Classificação |
|---|--------------------------------|--------------|
| 4.1 | **Ponto de revisão humana obrigatório:** Quando o assistente gerar uma resposta com nível de confiança abaixo de 70%, essa resposta não é entregue diretamente ao atendente. Ela vai para uma fila de revisão visível aos 5 atendentes-piloto, que têm 30 minutos para aprovar, editar ou descartar. Enquanto aguarda, o usuário vê a mensagem: *"Consultando um especialista, aguarde."* Se nenhum atendente revisar dentro de 30 minutos, o sistema envia automaticamente uma mensagem de encaminhamento para o time humano. | **[BLOQUEANTE]** |
| 4.2 | O assistente deve responder apenas dentro do escopo para o qual foi treinado. Perguntas sobre assuntos fora desse escopo (como orientação jurídica, médica ou financeira de alta complexidade) devem receber automaticamente uma mensagem de encaminhamento para o time humano — o assistente nunca tenta responder esses tópicos. Essa regra não pode ser alterada pelo usuário durante a conversa. | **[BLOQUEANTE]** |
| 4.3 | Um módulo de registro de feedbacks foi desenvolvido com auxílio de IA e, durante a revisão, foram identificados dois problemas graves: ele não validava os dados recebidos e armazenava informações pessoais dos atendentes de forma exposta. Esse módulo deve ser corrigido e aprovado pelo responsável técnico antes do go-live. | **[BLOQUEANTE]** |
| 4.4 | Cada atendente pode fazer no máximo 20 perguntas por hora ao assistente. Isso evita que um uso intenso por uma pessoa prejudique o serviço para os demais atendentes-piloto. | **[DESEJÁVEL]** |

---

## Área 5 — Monitoramento em Produção
*Visibilidade para saber em tempo real se o assistente está funcionando bem*

**Por que isso importa:** Sem monitoramento, qualquer problema em produção só será descoberto quando um atendente reclamar. Estes critérios garantem que o time técnico veja os problemas antes dos usuários.

| # | O que precisa estar funcionando | Classificação |
|---|--------------------------------|--------------|
| 5.1 | Antes do go-live, deve estar ativo um painel de monitoramento em tempo real mostrando: volume de perguntas, nível médio de confiança das respostas, fila de revisão humana, taxa de respostas bloqueadas, tempo de resposta e taxa de erros. Visível para o Delivery Manager e o responsável técnico. | **[BLOQUEANTE]** |
| 5.2 | Todos os registros de interação devem seguir as regras de privacidade: nenhum dado pessoal do atendente (nome, e-mail, texto da pergunta sem tratamento) pode ser armazenado de forma exposta. O time de QA deve auditar esse ponto antes do go-live. | **[BLOQUEANTE]** |
| 5.3 | Alertas automáticos devem ser configurados: se a taxa de respostas bloqueadas ultrapassar 10% ou a taxa de erros ultrapassar 15% em qualquer janela de 15 minutos, o time técnico recebe uma notificação automática no canal do Teams. Isso não pode depender de alguém estar olhando o painel manualmente. | **[BLOQUEANTE]** |
| 5.4 | O sistema deve registrar o caminho completo de cada interação — da pergunta do atendente até a resposta entregue — para que em caso de incidente a equipe consiga identificar exatamente onde ocorreu o problema. A ser implementado após a demo. | **[DESEJÁVEL]** |
