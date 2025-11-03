## 1. Criação de Épico
  
Todas as demandas deverão estar vinculadas a um **Épico**, seja ele relacionado a **clientes** ou a **demandas internas**.  

### Orientações:

- Cada Épico deve detalhar informações sobre o **produto ou iniciativa**. 
- Para **demandas internas**, indicar claramente o tipo:  
	- Pronto Atendimento  
	- Agendamento  
	- Integração
	- Outros  

- Para **demandas de clientes**, criei um template para adicionar os dados no descritivo
	- 📄 **Template de apoio**: [Modelo de Épico](https://doutorhoje.atlassian.net/wiki/pages/resumedraft.action?draftId=240484353&draftShareId=d6a01177-8189-4ba4-9d63-4711a70baf78)  


🔎 **Importante**: todo acompanhamento de demandas deverá ser realizado a partir do Épico correspondente.  

---

## 2. Criação de Demandas

Para garantir organização e consistência, **toda nova demanda deverá ser criada via formulários**.  
### Benefícios:

- Permite registrar a demanda sem a necessidade de detalhar todas as informações no momento da criação.  

- Evita poluição do backlog com dados incompletos ou mal formatados.  

- Facilita a categorização e priorização posterior.  

📌 **Atenção**:  Ao consultar os formulário selecione o filtro de categoria **FORM**.  
🔗 Formulário oficial: [Adicionar Demanda](https://doutorhoje.atlassian.net/jira/software/projects/DV/form)
![[Captura de Tela 2025-09-26 às 09.04.24.png|640x376]]

---

## 3. Criação de Tarefa

Ao criar uma **tarefa**, é fundamental garantir que todas as informações necessárias estejam preenchidas para que o time tenha clareza sobre o que deve ser feito.

### Requisitos obrigatórios:

- 🔗 **Épico vinculado** → toda tarefa deve estar associada a um épico previamente criado.  
- 📝 **Descrição detalhada** → incluir contexto, objetivo e critérios de aceite (preferencialmente no formato *Given/When/Then*).  
- 📅 **Data limite** → informar a *due date* ou expectativa de entrega.  

### Requisitos recomendados:

- 🎯 **Sprint** → sempre que possível, já alocar a tarefa na sprint atual ou planejada.  

- 👤 **Responsável** → atribuir a tarefa a um desenvolvedor, considerando a natureza da atividade e a capacidade do time.  

- ⏱️ **Estimativa de esforço** → registrar a pontuação (story points) ou horas, de acordo com o método adotado pelo time.  

- 🏷️ **Categoria** → adicionar tags que facilitem filtragem e relatórios.  

- 📎 **Anexos e referências** → incluir links, documentos ou prints que ajudem na execução.  

- ✅ **Subtarefas**→ detalhar etapas menores dentro da tarefa.  

### Boas práticas:

1. **Clareza nos critérios de aceite** → usar Gherkin (Given/When/Then) ou uma lista de condições objetivas que determinem quando a tarefa está concluída.  
2. **Divisão de tarefas grandes** → se a atividade for extensa, quebrar em subtarefas ou tarefas menores.  
3. Divisão de tarefas por tecnologia → se a atividade tiver back e front realizar a divisão em tarefas distintas.
4. **Revisão antes da sprint** → validar em conjunto durante o *Planning* para confirmar entendimento, prioridade e esforço.  

### Fluxo Resumido de Criação de Tarefa:

1. Criar tarefa vinculada a um épico.  
2. Adicionar descrição clara + critérios de aceite.  
3. Definir prazo e estimativa.  
4. Alocar sprint e responsável (quando aplicável).  
5. Validar com o time durante o planejamento.

---
## 4. Planning

O **Sprint Planning** é um momento essencial para alinhar expectativas, priorizar atividades e garantir que toda a equipe esteja comprometida com as entregas da sprint.  

### Objetivos principais:
- ✅ Definir **quais tarefas** entrarão na sprint.  
- ✅ Realizar a **estimativa de esforço** (story points), considerando a complexidade e dependências.  
- ✅ Garantir que **toda a equipe esteja presente** e alinhada.  
- ✅ Distribuir as responsabilidades respeitando a **capacidade (capacity)** de cada membro do time.  

### Atividades durante a Planning:
1. **Revisão da sprint futura**
   - Apresentar e discutir todas as tarefas propostas.  
   - Validar critérios de aceite e entendimento coletivo.  

 2. **Pontuação das tarefas** 
   - Utilizar técnica de estimativa (ex.: Planning Poker).  
   - Calcular capacidade do time para a sprint.  

3. **Definição dos responsáveis**  
   - Atribuir tarefas de acordo com a especialidade e capacidade de cada desenvolvedor.  
   - Evitar sobrecarga individual, respeitando a capacidade planejada.  

4. **Criação do documento de acompanhamento**  
   - Registrar as **principais entregas da sprint**.  
   - Documento servirá de guia nas **Dailys**, ajudando o time a manter o foco.  
   - Feito diretamente no Jira/Confluence para facilitar visibilidade.  

5. **Retirada de demanda da Sprint**
	- Caso o capacite exceda, sempre respeitando as prioridades.
	- Caso a demanda não cumpra o mínimo de detalhamento.
	- Itens bloqueados sem expectativa para o desbloqueio.

---

## 5. Daily

A **Daily** é um dos ritos mais importantes do nosso dia a dia.  
Seu objetivo é garantir **alinhamento diário** da equipe, identificar bloqueios rapidamente e manter o **foco nas entregas da sprint**.  

### Orientações gerais:
- A daily **deve acontecer todos os dias úteis**, mesmo que falte algum membro do time.  
- O **PO** deve usar o documento criado na *Planning* como guia para manter a equipe focada e como material de reporte para a gestão.  
- A reunião deve ser curta (até **15 minutos**) e objetiva.  

### Estrutura da Daily:
Cada colaborador deve responder a três perguntas:
1. **O que fiz ontem?**  
2. **O que farei hoje?**  
3. **Tenho algum bloqueio ou dificuldade?**  

### Papel do PO na Daily:
- 📌 Monitorar se as entregas estão de acordo com o foco da sprint.  
- 📌 Identificar possíveis desvios ou riscos.  
- 📌 Apoiar na remoção de impedimentos junto ao time e à gestão.  

### Boas práticas:
- 🎯 Manter o foco no andamento da sprint (evitar discussões longas → abrir reuniões paralelas quando necessário). 
- 🧭 Utilizar o documento de *Planning* como **referência visual** para progresso e metas. 
- ⏱️ Respeitar o limite de tempo para não comprometer a rotina da equipe. 

---

## 6. Review

A **Sprint Review** ocorre ao final da sprint e tem como objetivo **apresentar as entregas** realizadas ao PO, stakeholders e demais interessados.  

É o momento de validar se o que foi produzido atende às expectativas e gera valor de negócio.  

### Objetivos:

- ✅ Demonstrar as funcionalidades concluídas.  
- ✅ Validar se os objetivos da sprint foram atingidos.  
- ✅ Ajustar prioridades do backlog, se necessário.  

### Estrutura da Review:

1. **Abertura**: PO relembra os objetivos da sprint.  
2. **Demonstração**: time apresenta o que foi entregue.
3. **Encerramento**: alinhamento sobre próximos passos no backlog.  
    

---

## 7. Retrospective

A **Sprint Retrospective** é o momento em que o time analisa o processo da sprint finalizada, identificando pontos fortes, oportunidades de melhoria e ações para a próxima sprint.  

### Objetivos:

- ✅ Identificar o que funcionou bem.  
- ✅ Levantar dificuldades e problemas ocorridos.  
- ✅ Definir ações de melhoria contínua.  
- ✅ Reforçar a colaboração e transparência do time.  

### Estrutura da Retrospective:

1. **Abertura**: PO contextualiza o objetivo da reunião.  
2. **Discussão em grupo** (ex.: quadro “Start / Stop / Continue”):  
	1. O que devemos **continuar** fazendo?  
	2. O que devemos **parar** de fazer?  
	3. O que devemos **começar** a fazer?  

3. **Definição de ações**: priorizar 1 a 3 ações concretas de melhoria.  
4. **Encerramento**: registrar decisões e responsáveis pelas ações.  

- - -
 
## 7. Fluxo de card

	Na atuação o desenvolvedor irá puxar o card para o para desenvolvimento, ira criar a branch no projeto que irá atuar. Após conclusão detalhar com links, print