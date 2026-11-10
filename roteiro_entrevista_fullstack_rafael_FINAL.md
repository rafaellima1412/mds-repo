# Roteiro de Entrevista — Especialista em Desenvolvimento Full Stack 
### Versão FINAL — Rafael Barbosa de Lima

---

## Como usar

Esta é a versão consolidada, construída a partir da simulação completa que fizemos. Cada pergunta tem uma **resposta-modelo pronta**, já ajustada com seus exemplos reais. Não decore palavra por palavra — internalize a estrutura e os fatos, e fale com suas próprias palavras no dia.

⚠️ **Três pontos estratégicos para toda a entrevista:**

1. **Inglês:** seu nível é Intermediário, a vaga pede Avançado. Não force um discurso decorado — treine falado (voz alta, gravando) nos próximos dias, principalmente vocabulário técnico. Se travar numa pergunta complexa, é normal pedir para reformular.
2. **Kubernetes e EAP com brokers (Kafka/RabbitMQ):** gaps reais, mas você tem respostas honestas e maduras preparadas — não finja domínio que pode ser testado tecnicamente.
3. **Coerência:** você errou a ordem cronológica das empresas duas vezes durante o treino. Revise a ordem certa antes de dormir na véspera: **ArchOffice → Locus → ITRIAD → FITEC → Riolivre (atual).**

---

## Bloco 1 — Abertura

**1. "Fale um pouco sobre você e sua trajetória profissional."**

> **Resposta-modelo:** "Minha trajetória sempre teve foco em backend e arquitetura. Comecei como júnior na ArchOffice, depois fui pra Locus, e em seguida atuei como pleno na ITRIAD por dois anos e meio. Na FITEC, comecei como pleno e, quando o arquiteto responsável saiu do projeto, assumi a vaga de arquitetura em AWS — o que me permitiu aprender e aprofundar bastante nessa área. E hoje, no Riolivre, atuo como freelancer ponta a ponta — da arquitetura ao provisionamento da infraestrutura."

**2. "Por que você se interessou por esta vaga e por essa empresa?"**

> **Resposta-modelo:** "Eu já venho trabalhando com backend, arquitetura e infraestrutura em AWS, e essa vaga me chamou atenção porque une o que eu já faço bem — Python, arquitetura de sistemas, Clean Architecture — com áreas que quero aprofundar, como Kubernetes e arquiteturas orientadas a eventos em maior escala. Recentemente, na FITEC, assumi a responsabilidade de arquitetura AWS de forma mais direta, e isso despertou meu interesse em continuar crescendo nessa direção."

**3. "A vaga pede Javascript, React, Angular e/ou Python. Fale sobre sua experiência de front-end."**

> **Resposta-modelo:** "Meu foco sempre foi backend e arquitetura, mas já trabalhei com React em produção duas vezes: na Locus, num projeto com Python no backend e React no frontend, e na ITRIAD, num sistema de qualidade em Go com Gin no backend e React no frontend, que notificava defeitos detectados em peças. Então tenho contato prático real com React, mesmo não sendo minha frente principal — onde entrego mais valor é em arquitetura e backend."

---

## Bloco 2 — Experiências (situação → ação → resultado)

**4. "Me conte sobre seu projeto atual/mais recente, o Riolivre."**

> **Resposta-modelo:** "O Riolivre é uma plataforma que funciona como um 'Uber dos rios' — conecta usuários a rotas fluviais entre cidades como Manaus e Amapá, calculando a melhor rota por preço ou trajeto, considerando disponibilidade de portos. Meu papel foi arquitetar e sustentar o backend sozinho, como freelancer. O sistema usa algoritmos de grafo com DFS para calcular as melhores rotas entre portos. A modelagem do banco de dados foi crucial pra sustentar isso com performance. Estruturei a aplicação em Golang com Design Hexagonal, separando bem as regras de negócio da infraestrutura, e cuidei também do provisionamento AWS e da containerização com Docker."

**5. "Fale sobre a FITEC."**

> **Resposta-modelo:** "Entrei na FITEC como backend num projeto B2B, trabalhando com FastAPI e PostgreSQL, seguindo Clean Architecture — separando bem domínio, aplicação e infraestrutura. Comecei fazendo pair programming com devs júnior e apoiando decisões do time. Quando o arquiteto responsável saiu, assumi a vaga e passei a cuidar da infraestrutura de ponta a ponta: AWS com ECS e Lambda, VPC para rede, automação de deploy via CDK. Também implementei notificações via e-mail usando SNS e SQS, com filas mortas (DLQ) configuradas pra tratar falhas sem perder mensagens, e criei webhooks na aws. Depois, migrei pra outro projeto interno, o funil de vendas, atuando em arquitetura e code review."

**6. "Por que Clean Architecture e Arquitetura Hexagonal nesses projetos?"**

> **Resposta-modelo:** "Na FITEC, usei Clean Architecture porque ela separa a regra de negócio do framework e do banco — o domínio não depende de FastAPI nem de PostgreSQL,mongodb e s3 diretamente, só de interfaces. Isso trouxe testabilidade real, dá pra testar a lógica de negócio com mocks. No Riolivre, usei Arquitetura Hexagonal em Go porque o módulo de cálculo de rotas com grafos e DFS precisava ficar desacoplado de como os dados chegavam. Com portas e adapters, defini contratos claros, o que facilita testar isoladamente e trocar implementações sem tocar na lógica central."

**7. "Fale sobre sua experiência na ITRIAD."**

> **Resposta-modelo:** "Na ITRIAD atuei em múltiplos projetos ao longo de dois anos e meio. Um deles foi um sistema em Go com Gin e React, voltado pra qualidade em chão de fábrica, que enviava notificações quando detectava defeitos em peças. Outro foi em C# com SQL Server, também de qualidade, rodando em PCs e tablets, com um módulo de visão computacional usando YOLO pra reconhecimento de ferramentas — meu papel ali foi o backend de uma versão mobile, integrando com o resultado do modelo. Também tive contato com NestJS/TypeScript em outras frentes do time."
>
> *Se perguntarem detalhe técnico de como a notificação de defeito funcionava por trás:* "Era notificação de sistema"

**8. "Você já trabalhou com micros serviços e arquitetura orientada a eventos (EAP)? Me dê um exemplo."** (celery - biblioteca de código aberto escrita em Python usada para gerenciar **filas de tarefas assíncronas** e processamento em segundo plano)

> **Resposta-modelo:** "Sim. Na FITEC trabalhei com processamento assíncrono usando AWS SNS e SQS. Os eventos eram publicados no SNS e processados de forma assíncrona através de filas SQS, com DLQ para tratamento de falhas sem perder mensagens.
> 
> Além disso, implementei uma estrutura usando Celery integrado ao SQS como broker. Configurei workers para consumir as mensagens da fila, controle de concorrência e prefetch 
> (para evitar que o worker pegasse várias mensagens de uma vez). Também utilizei Celery Beat para agendar tarefas periódicas.
> 
> Então, além do conceito de publish/subscribe e filas assíncronas, tive contato prático com a implementação do processamento em background, roteamento de tarefas para filas específicas e execução de jobs agendados. Não tenho experiência profunda com Kafka ou RabbitMQ, mas já trabalhei diretamente com os principais conceitos de arquiteturas orientadas a eventos."

**9. "Fale sobre uso de gRPC ou APIs de comunicação além de REST."**

> **Resposta-modelo:** "Na ArchOffice, trabalhei num miniprojeto onde tínhamos dois backends em linguagens diferentes — um em Python e outro em Javascript — que precisavam se comunicar entre si. Usamos gRPC pra essa comunicação, justamente porque funciona bem nesse cenário poliglota, com contratos bem definidos via protobuf. Também usei a biblioteca `requests` no lado Python pra outras integrações via REST."
> (O **Java** usa a palavra `null` para representar a ausência de um valor ou objeto. Já o **Python** não usa a palavra `null`) ai no protobuf a gente pôs NONE se viesse NULL.

---

## Bloco 3 — Técnico: Infraestrutura, DevOps e Cloud

**10. "Descreva um pipeline de CI/CD que você construiu ou manteve, em cloud e on-premises."**

> **Resposta-modelo:** "Um exemplo real foi um pipeline que criei usando AWS CodeBuild com um buildspec.yml. O processo tinha três fases: no pre_build, autenticava no ECR; no build, fazia o build da imagem Docker e taggeava com o nome do ambiente; no post_build, dava push da imagem pro ECR e gerava o imagedefinitions.json, que o ECS usa pra saber qual imagem puxar no deploy. Isso automatizava o ciclo do commit até a imagem rodando em produção. Minha experiência de CI/CD é toda em cloud, especificamente AWS — não tenho vivência prática com pipelines on-premises, mas os conceitos centrais são os mesmos e tenho facilidade pra adaptar."
>
> ⚠️ Nunca mencione ou mostre o ID da conta AWS nem detalhes literais do YAML — descreva o processo, não leia o código.

**11. "Qual sua experiência com Kubernetes?"**

> **Resposta-modelo:** "Tenho contato com Kubernetes através do EKS na AWS — entendo os conceitos centrais, como pods, deployments e services. Não é minha área de maior profundidade ainda, minha experiência de orquestração foi mais forte com ECS e Lambda. É uma lacuna que já identifiquei e estou estudando ativamente, porque sei que é um requisito importante pra essa vaga."
>
> 💡 Antes da entrevista, tenha na cabeça 2-3 conceitos concretos: diferença entre Deployment e StatefulSet, Service ClusterIP vs. LoadBalancer, o que é um Ingress.

**12. "Como você lida com concorrência em Go?"** *(pergunta possível, mas fora do escopo da vaga)*

> A vaga não pede Go — essa pergunta só pode surgir se o entrevistador notar Go no seu CV/histórico. Não precisa se preparar tão a fundo quanto Kubernetes ou EAP; uma resposta simples com um exemplo de goroutines/channels de algum projeto real já é suficiente se vier à tona.

**13. "Fale sobre otimização de banco de dados — um exemplo real."**

> **Resposta-modelo:** "Um exemplo real foi no Riolivre, num banco PostgreSQL, onde usei o EXPLAIN pra identificar queries lentas — a partir daí, decidia entre criar um índice, reescrever a query, ou em casos mais extremos, particionar a tabela. Outro problema que resolvi foi identificar sessões fantasmas ocupando o pool de conexões do banco, o que estava limitando a capacidade de novas conexões. A partir disso, também fiz ajustes reais na capacidade do pool, calibrando com base nos recursos de infraestrutura disponíveis — disco e processador."

**14. "Quais serviços AWS você já usou?"**

> **Resposta-modelo (cite por categoria conforme a pergunta específica, não precisa recitar tudo de uma vez):**
> - Rede: VPC, subnets públicas e privadas, tabelas de rota, balanceadores de carga (ALB/NLB), Target Group, Transit Gateway
> - Computação: EC2, ECS, Lambda
> - Banco de dados: RDS, DocumentDB
> - Armazenamento: S3
> - Integração/mensageria: API Gateway, SNS(notificação), SQS(queue), SES(email)
> - Autenticação: Cognito
> - IaC: CloudFormation, CDK
> - Segurança: IAM
> - Observabilidade: CloudWatch
>
> 📝 **Atualize seu CV com essa lista** — hoje ele registra só ECS, Lambda, VPC, CDK, bem menos do que você realmente domina.

**15. "Você já trabalhou com workers ou processamento distribuído?"**

> **Resposta-modelo:** "Sim. Trabalhei com Celery utilizando AWS SQS como broker. Estruturei workers para consumir tarefas de uma fila específica e configurei parâmetros de concorrência e prefetch para controlar o consumo das mensagens.
> 
> Também utilizei Celery Beat para agendamento de tarefas periódicas, permitindo que determinadas rotinas fossem executadas automaticamente em horários definidos, sem depender de uma requisição HTTP.
> 
> Um exemplo era uma rotina de verificação do status de ordens de serviço, que era executada automaticamente e processava os dados em background."

---

## Bloco 4 — Negócio, metodologia e soft skills

**16. "Como funciona seu dia a dia com metodologias ágeis?"**

> **Resposta-modelo:** "Já trabalhei com times em diferentes formatos ágeis — Scrum, com sprints e cerimônias de review e retro, e também Kanban, mais focado em fluxo contínuo. Em alguns projetos as entregas eram diárias, em outros semanais. Também tive contato com XP, principalmente em práticas como pair programming, que usei bastante na FITEC."

**17. "Me dê um exemplo de quando você traduziu uma regra de negócio complexa para uma solução técnica."**

> **Resposta-modelo:** "Um exemplo bom foi na FITEC: uma integração direta com um ERP, do jeito planejado originalmente, levaria meses pra ficar pronta. Analisei a situação e identifiquei uma alternativa mais simples: em vez de fazer uma integração profunda, criei um endpoint no nosso backend pro ERP fazer um POST diretamente, invertendo a responsabilidade. Isso reduziu drasticamente o tempo de implementação e resolveu a necessidade real do negócio."

**18. "Conte sobre uma decisão técnica que você tomou sozinho, com pouca direção."**

> **Resposta-modelo:** "Na FITEC, os papéis de acesso do sistema só foram definidos bem depois do desenvolvimento já estar em andamento. Precisei encontrar uma forma de adicionar controle de acesso por tela sem reestruturar o sistema inteiro. Decidi sozinho usar anotações (decorators) em Python pra embutir informações de permissão de forma criptografada dentro do JWT — cada endpoint verificava via decorator se o usuário tinha a claim necessária pra acessar aquela tela, sem precisar de retrabalho grande na estrutura já existente."

**19. "Como você avalia se vale a pena implementar uma nova solução técnica?"**

> **Resposta-modelo:** "Costumo avaliar boas práticas primeiro — ver se existem bibliotecas ou padrões consolidados e ativamente mantidos, em vez de reinventar. Também discuto com outros devs do time, pra pegar diferentes perspectivas. Mas o ponto mais importante pra mim é medir o impacto real: no caso do ERP que mencionei antes, a decisão de criar um endpoint simples em vez de uma integração profunda economizou meses de trabalho sem perder o resultado que o negócio precisava. Meu critério final é: essa solução resolve o problema com o menor custo/risco possível, mantendo qualidade?"

---

## Bloco 5 — Inglês

**20. "Can you walk me through a project you worked on recently?"**

> ⚠️ Não treinamos essa parte no chat (faz mais sentido praticar falado, em voz alta, do que escrito). Prepare 2-3 respostas simples sobre Riolivre e FITEC, com vocabulário técnico básico (deploy, pipeline, architecture, scalability, database). Grave sua voz respondendo e reescute antes da entrevista.

---

## Bloco 6 — Encerramento

**21. "Onde você se vê daqui a alguns anos?"**

> **Resposta-modelo:** "Me vejo continuando a aprofundar o caminho que já venho trilhando — arquitetura de sistemas e infraestrutura. Já tive experiência real assumindo responsabilidade ponta a ponta, da arquitetura ao deploy, tanto na FITEC quanto no Riolivre, e quero continuar crescendo nessa direção, seja como arquiteto de soluções ou numa frente mais forte de DevOps, aprofundando principalmente em Kubernetes e arquiteturas orientadas a eventos em maior escala."

**22. "Pretensão salarial e disponibilidade?"**

> **Resposta-modelo:** "Minha pretensão gira em torno de R$ 9.000, considerando minha última remuneração de R$ 7.650 e a experiência que venho agregando em arquitetura e infraestrutura nos projetos mais recentes. Tenho flexibilidade pra conversar sobre isso dependendo do pacote completo da vaga. E minha disponibilidade é imediata."

**23. "Você tem perguntas para nós?"**

> **Perguntas sugeridas:**
> - "Como o time costuma atuar nos projetos — vocês trabalham em squads fixos ou rotativos entre projetos diferentes?"
> - "Essa vaga é pra dar continuidade a um projeto já em andamento, ou pra começar algo novo?"

---

## Fechamento sugerido

> "O que me atrai nessa vaga é continuar aprofundando o que já venho construindo: uma base sólida em backend com Python e Go, disciplina de Clean Architecture e Arquitetura Hexagonal, e a responsabilidade de decidir e sustentar decisões técnicas de ponta a ponta — da arquitetura ao deploy em produção. Vejo essa vaga como uma oportunidade de expandir isso para Kubernetes e arquiteturas orientadas a eventos em maior escala, além de consolidar meu inglês técnico no dia a dia."

---

## Checklist final (revisar na véspera)

1. **Ordem cronológica:** ArchOffice → Locus → ITRIAD → FITEC → Riolivre (atual). Você errou isso duas vezes no treino — revise escrito antes de dormir.
2. Go/Gin é a frente central na ITRIAD; C#/YOLO e NestJS são projetos paralelos na mesma empresa.
3. Front-end: dois casos reais (Locus e ITRIAD) — resposta já é sólida, sem precisar inflar.
4. gRPC: só o exemplo da ArchOffice (dois backends poliglotas, Python + Javascript).
5. EAP: SNS + SQS + DLQ na FITEC é exemplo real e forte — não é mais gap grande. Kafka/RabbitMQ seguem como próximo passo honesto.
6. Kubernetes: gap real — resposta madura já pronta (EKS + estudando ativamente).
7. Inglês: nível intermediário — praticar em voz alta antes da entrevista, não decorar texto complexo.
8. Nunca mencionar ID de conta AWS ou outros dados sensíveis de projetos ao descrever pipelines/infra.
9. Notificação automática de defeitos (ITRIAD, Go/React): não lembra o detalhe técnico — não inventar se perguntado.
10. Não usar números não documentados (% de melhoria, volume de requisições) a menos que tenha isso de cabeça.
11. **Atualizar o CV** com: lista completa de serviços AWS, SNS/SQS/DLQ, segundo caso de React (ITRIAD), Arquitetura Hexagonal.
