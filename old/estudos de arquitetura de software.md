
Arquitetura de sistemas
1.     Capacidade de Navegar na Ambiguidade: Você consegue pegar um problema vago e transformá-lo em requisitos técnicos concretos? 
2.     Visão Holística: Você entende como o banco de dados afeta a API, que afeta o Load Balancer, que afeta o cliente mobile?
3.     Comunicação: Você consegue explicar por que escolheu o banco de dados Cassandra em vez do MySQL? Você consegue justificar suas escolhas sem ficar na defensiva?

Fundamentos: Vamos dissecar o que significa Latência, Throughput e Disponibilidade. 
●       Os Blocos de Construção: Analisaremos cada componente de um sistema moderno: Load Balancers, Caches, Filas de Mensagem, Bancos de Dados e Proxies.
Os Padrões: Como unir esses blocos? Falaremos de Sharding, Replicação, Consistência Eventual e Microsserviços.


Escala Vertical vs. Escala Horizontal
vertical
No servidor, é a mesma coisa. Você está rodando numa instância t2.micro da AWS (1 CPU, 1GB RAM). O site fica lento. Você desliga a máquina e a liga novamente como uma m5.24xlarge (96 CPUs, 384GB RAM).

O Muro da Escala Vertical (Por que ela falha): A Escala Vertical tem um teto de vidro muito rígido e perigoso. ●    Limitação de Hardware: Não existe processador com velocidade infinita. Existe um limite físico de quanta RAM você consegue colocar em uma placa-mãe. Chega um ponto em que, mesmo que você tenha dinheiro infinito, não existe computador potente o suficiente no mercado para processar todo o tráfego do Facebook. ●    Lei dos Rendimentos Decrescentes: Uma máquina com o dobro de poder custa mais que o dobro do preço. Uma "supermáquina" é exponencialmente mais cara que várias máquinas comuns (commodity hardware). ●    Ponto Único de Falha (SPOF - Single Point of Failure): Se você coloca todos os seus ovos nessa única cesta dourada e a placa-mãe queima, seu negócio inteiro desaparece até você conseguir restaurar um backup em outra máquina. Em sistemas críticos, isso é inaceitável.

horizontal
Vantagens da Escala Horizontal: Crescimento Infinito (Teórico): Se o Google precisar indexar mais 1 bilhão de páginas, eles apenas compram mais racks de servidores. Não há limite físico.  Custo-Eficiência: Usar 100 computadores médios é muito mais barato do que tentar construir um supercomputador mainframe.  Resiliência: Se um servidor pegar fogo, os outros 99 continuam trabalhando. O usuário nem percebe.  O Preço da Escala Horizontal (A Complexidade): Aqui está a pegadinha. Quando você divide seu sistema em várias máquinas, você introduz problemas que não existiam antes: 
●    Onde está o dado? Se o usuário A salvou a foto no Servidor 1, e depois tenta ver a foto caindo no Servidor 2, como o Servidor 2 sabe onde a foto está? 
●    Latência de Rede: Servidores precisam conversar entre si via cabos de rede (RPC, HTTP). Isso é milhares de vezes mais lento que ler da memória RAM local. 
●    Consistência: Garantir que todos os 1.000 servidores concordem sobre qual é o saldo da sua conta bancária é um dos problemas mais difíceis da ciência da computação (veremos isso no Teorema CAP).

Resumo para Entrevistas: "Sempre comece com Escala Vertical para MVPs e startups, pela simplicidade. Mas projete o sistema sabendo que, inevitavelmente, você terá que migrar para Escala Horizontal quando atingir o 'Product-Market Fit' e a carga aumentar."

Latência vs. Throughput
No software: É quantas requisições por segundo (RPS) ou transações por segundo (TPS) seu servidor aguenta.

O Trade-off (A Troca) 
Geralmente, sistemas otimizados para Throughput sacrificam a Latência, e vice-versa. 
●    Exemplo de Throughput: Processamento em Lote (Batch). O banco espera juntar 1.000 transações e salva todas de uma vez no disco. É muito eficiente para o servidor (alto throughput), mas o primeiro usuário teve que esperar as outras 999 chegarem para ter sua transação salva (alta latência). 
●    Exemplo de Latência: Jogos online ou Trading de Alta Frequência. Cada milissegundo conta. Processamos cada pacote de dados imediatamente. O servidor gasta muito mais CPU fazendo trocas de contexto (baixo throughput total), mas a resposta é instantânea. Pergunta de 

Entrevista: "Seu sistema está lento?" Sua Resposta: "Lento como? Estamos com latência alta (demora para responder um usuário) ou throughput baixo (o servidor engasga com muitos usuários)? As soluções são opostas."

SLA, SLO e SLI (A Sopa de Letrinhas) ●    SLI (Indicator): O que estamos medindo? (Ex: A taxa de erro das requisições HTTP). ●    SLO (Objective): Qual é a nossa meta interna? (Ex: Queremos que 99.5% das requisições sejam sucesso). ●    SLA (Agreement): O contrato com o cliente. (Ex: Se o sistema cair mais que 99%, devolvemos o dinheiro do cliente). ○      Dica de Ouro: O SLA deve ser sempre menor que o SLO. Se sua meta interna é 99.9%, prometa ao cliente apenas 99%. Isso te dá uma margem de segurança.

O Teorema CAP: A Lei Física dos Sistemas Distribuídos Este é o conceito teórico mais importante deste livro. Proposto por Eric Brewer, o Teorema CAP afirma que em um sistema de dados distribuído (vários computadores guardando dados), você só pode garantir duas destas três propriedades simultaneamente:
consistency, availability, partiotion tolerance.

Bem-vindo ao problema do Estado. Servidores web são Stateless (sem estado); se um morrer, o próximo assume. Bancos de dados são Stateful (com estado); se ele morrer, os dados somem (ou ficam inacessíveis). Neste capítulo, vamos aprender a cirurgia mais delicada da arquitetura: como escalar a persistência de dados. 1. O Grande Cisma: SQL (Relacional) vs. NoSQL (Não-Relacional) A primeira decisão arquitetural é: como vamos estruturar os dados? Durante 40 anos, a resposta foi única: SQL. Hoje, a escolha é baseada em trade-offs.

Bem-vindo ao problema do Estado. Servidores web são Stateless (sem estado); se um morrer, o próximo assume. Bancos de dados são Stateful (com estado); se ele morrer, os dados somem (ou ficam inacessíveis). Neste capítulo, vamos aprender a cirurgia mais delicada da arquitetura: como escalar a persistência de dados. 1. O Grande Cisma: SQL (Relacional) vs. NoSQL (Não-Relacional) A primeira decisão arquitetural é: como vamos estruturar os dados? Durante 40 anos, a resposta foi única: SQL. Hoje, a escolha é baseada em trade-offs.
Atomicity, consistency, isolation e durability.
BA - Basically Available: O sistema funciona a maior parte do tempo.  S - Soft state: O estado pode mudar com o tempo, mesmo sem input novo (devido à replicação).  E - Eventual consistency: O dado vai ficar consistente... uma hora.

escalando a leitura: Replication (master-slave)
master: lider aceita comando escrita
slave: seguidores apenas leitura

fluxo: se vai salvar master se vai ler slave
vantagem: 
perfomace:1 master 20 slave a leitura e 20x maior.
retundancia: se master cair outro slave e promovido a master.
perigo: lag na replicação.

Escalando a Escrita: Sharding (Particionamento)
Chegou a hora da "Opção Nuclear": Sharding (Particionamento Horizontal).Dividir e fragmentar a arquitetura principal do banco de dados em várias máquinas.
custos: 
perde join tem que fazer no codigo em memoria.
transação complexa....
reshardoing: mover dados de um banco em produção sem tiralo do ar. 

usar framework de decisão:
dados são estruturados?
sim -->SQL
não-->Nosql
precisamos de ACID?
sim -->SQL
não-->Nosql
qual volume de dados?
até 5terabytes-->SQL
Petabytes -->Nosql
Qual a carga de teitura vs de escrita?
muita leitura-->SQL+Caching+read replicas
muita escrita--->Nosql e otimizado para engolir escrita.

passos:
1- podemos começar com monolito
2- escalou read replicas(master-slave)
3-precisa flexibilidade ou escala massiva escrita no sql
4 volume insustentável --> sharding mas perde capapcidade de fazer join facil e transação complexas.

caching

redis--->Quando usar: 99% dos casos modernos. Se você precisa de rankings (Leaderboards), filas, contadores ou Pub/Sub, Redis é a escolha padrão.

mensageria
message broker
modelo produtor-consumidor
produtor-->fila(queue/topic)-->consumidor

não basta essa fila no meio o problema e organnizar ela.
como organizamos?
diparar e esquecer
publish/ subscribe
dead letter queue

monolito e microserviço

monolito
deploy e simples
desenvolvimento e simples
perfomace
consistencia ACID

desvantangens:
acoplamento com tempo vira espaquete.
deploy lento 100 devs ci/cd fica enorme..
escala e tudo ou nada...
tecnologia unica.


microserviço

a lei de conway - "As organizações que projetam sistemas são restritas a produzir designs que são cópias das estruturas de comunicação dessas organizações."

Quando você tem 500 engenheiros, eles não conseguem trabalhar no mesmo código. Dividi-los em times pequenos com serviços próprios resolve o problema organizacional.

vantagem:
deploy independente
escala precisa
liberdade tecnológica
resiliencia (bulkheading)

desvantagenm:
complexidade distribuida
latencia
monolito distrbuido


reposta-entrevista: "Eu começaria com um monoilito modular. UM codigo bem organizado, separado em modulos intyernos, mas rodando num deploy unico. isso da velocidade de desenvolvimento no inicio. conforme o time crescer e indentificamos gargalos especificos de escala ou dominios que precisam de deploy independente, eu extrairia esses pedaços para microserviços usando o padrão stranger fig. começar com microserviço do zero adiciona complexidade desnecessaria antes de termos problemas reias para resolver."

serveless (fass) o novo garoto do bloco.
aws lambda

escala vertical - aumentar potencia da maquina
simples, 
consistência dos dados memoria compartilhada. 
velocidade comunicação- conversa de processos dentro da mesma cpu e mais rápido que por cabo de rede.

escala horizontal  - adicionar mais maquinas simples.
crescimento infinito - sem limite físico.
custo eficiência - usar pcs médios do que um super pc.
resiliência - se um pegar fogo outros continuam.

"pode iniciar com escala vertical mas quanto for maior a carga e nescessario ir para horizontal".

Latência e Throughput
Analogia:  rodovia.
latência--> tempo de viajem.
throughput-->capacidade da estrada.

No software quantas requisições por minuto (RPS) ou transações por minuto (TPS).

Exemplos:
throughput: batch esperar fazer 1000 transações no banco.
latência: Servidor d ejogo que usa muito mais CPU.

entrevista: "Sistema estar lento"?
reposta: "estar lento como ?" 
latencia alta? (demora a responder usuário)
ou throughput baixo? (o servidor engasga com muitos usuários)

SLA, SLI e SLO.

SLI: indicador ex.: Taxa de erros das requisições http.
SLO: nosso objetivo ex.: 99.5% sejam sucesso.
SLA: Contrato ex.: prometemos 99% de sucesso. 

CAP

consistência 
disponibilidade
tolerancia 




“Como funciona o processo de desenvolvimento? Vocês trabalham com code review, testes automatizados e CI/CD?”



