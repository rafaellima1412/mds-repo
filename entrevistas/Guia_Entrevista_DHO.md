# Guia de Entrevista: DHO / RH

## 1. Apresentação profissional

**Estrutura:**

- Quem sou profissionalmente
- Experiência em backend
- Atuação como referência técnica
- Gosto por resolver problemas
- Mentoria e colaboração
- Busca por ambientes onde possa contribuir tecnicamente e ajudar o time a evoluir

**Resposta-base:** "Minha trajetória sempre foi muito ligada a desenvolvimento backend, arquitetura e, ao longo do tempo, liderança técnica.

Comecei minha carreira como desenvolvedor **Júnior na ArchOffice**, onde tive meus primeiros contatos mais profundos com desenvolvimento de software, principalmente trabalhando com **SQL e gRPC**. Foi uma fase importante para construir minha base de desenvolvimento e entender melhor comunicação entre serviços e persistência de dados.

Depois fui para a **Lotus**, ainda como Júnior, atuando principalmente com **Python**. Foi onde comecei a ganhar mais experiência no desenvolvimento backend e a trabalhar de forma mais estruturada com a linguagem.

Na sequência, entrei na **iTriad**, inicialmente como **Backend Pleno**, trabalhando principalmente com **Python**. Depois passei também a atuar como **Full Stack Pleno**, ampliando minha experiência para outras partes da aplicação.

No meu último projeto dentro da iTriad, tive uma evolução importante: passei a atuar em uma posição mais próxima de **Senior**, sendo referência técnica para um time formado por **dois desenvolvedores Plenos e dois Júniores**. Além de desenvolver, comecei a atuar mais fortemente em decisões técnicas, revisão de código, orientação do time e resolução de problemas.

Depois, na **FITEC**, tive uma experiência ainda mais forte em arquitetura. O arquiteto responsável pelo projeto saiu e eu assumi parte dessa responsabilidade, principalmente relacionada à **arquitetura da solução e infraestrutura AWS**.

Nesse período, minha atuação não ficou somente no desenvolvimento. Eu fazia **code reviews**, participava dos **refinamentos técnicos dos cards no Jira**, ajudava a avaliar a complexidade das tarefas, riscos e impactos das decisões de desenvolvimento, além de identificar e tratar impedimentos que poderiam afetar o andamento do projeto. Isso me deu uma visão muito mais ampla do ciclo de desenvolvimento e também da responsabilidade técnica sobre o produto.

Mais recentemente, também atuei como **freelancer em um projeto de logística**, trabalhando de ponta a ponta: desde definição de arquitetura e desenvolvimento backend até infraestrutura e deploy. Nesse projeto tive contato novamente com **Go**, além de tecnologias relacionadas a infraestrutura e cloud.

Ao longo da minha trajetória, passei por diferentes linguagens e stacks, principalmente **Python e Go**, além de experiências com tecnologias como **SQL, gRPC e AWS**. Mas o que permaneceu constante foi a preocupação com arquitetura, boas práticas, qualidade de código, observabilidade, testes e construção de soluções que sejam sustentáveis no longo prazo.

Hoje me vejo muito nesse ponto de interseção entre **desenvolvimento backend, arquitetura e liderança técnica**. Gosto de continuar colocando a mão no código, mas também gosto de participar das decisões, revisar soluções, identificar riscos, ajudar o time a evoluir e transformar problemas de negócio em soluções técnicas.

É justamente essa combinação que vejo como bastante alinhada com a oportunidade."

**Versão curta (cerca de 1 minuto):**

"Minha trajetória sempre foi muito ligada a desenvolvimento backend e arquitetura. Comecei minha carreira como júnior, fui evoluindo por diferentes empresas e tecnologias e, ao longo desse caminho, fui assumindo responsabilidades cada vez maiores.
Passei por experiências com Python, Go e C#, mas sempre procurei manter os mesmos princípios de boas práticas, arquitetura e qualidade de código, independente da tecnologia.
Na FITEC, tive uma oportunidade importante quando o arquiteto responsável pelo projeto saiu e eu assumi parte dessa responsabilidade, principalmente relacionada à arquitetura e AWS. Foi um momento em que cresci bastante tecnicamente.
Mais recentemente, também atuei como freelancer em um projeto de logística, trabalhando de ponta a ponta, desde arquitetura e desenvolvimento até infraestrutura.
Hoje, além da parte técnica, gosto muito de atuar como referência para o time, ajudando nas decisões, revisões, resolução de problemas e desenvolvimento de outros profissionais. É justamente essa combinação entre backend, arquitetura e liderança técnica que me aproxima bastante dessa oportunidade."

---

## 2. Mensagem central da sua candidatura

"Sou um profissional técnico que também se preocupa com o desenvolvimento das pessoas e com a qualidade da entrega. Gosto de entender o problema, organizar o caminho, ajudar o time a superar obstáculos e criar autonomia, mantendo responsabilidade sobre resultado."

---

## 3. "Por que você quer essa vaga?"

**Foco:**

- Aderência entre experiência e desafios
- Oportunidade de atuar tecnicamente
- Arquitetura/backend
- Mentoria
- Impacto no time
- Evolução profissional

⚠️ Evitar responder apenas "porque preciso de uma oportunidade".

---

## 4. "Por que deveríamos escolher você?"

**Resposta-base:** "Acredito que posso contribuir porque sou muito dedicado ao trabalho, mas principalmente porque procuro equilibrar cobrança e suporte ao time. Uma liderança técnica precisa cobrar resultados e manter responsabilidade sobre entregas, mas também entender quando uma pessoa está enfrentando dificuldade e ajudá-la a superar. Cobrança excessiva pode gerar resistência e prejudicar desempenho. Quando o time entende os objetivos, participa das decisões e percebe que a liderança está ali para ajudar a resolver problemas, a colaboração aumenta e a entrega tende a ser melhor."

**Frase forte:** "Meu objetivo não é criar dependência no líder, mas ajudar o time a ganhar cada vez mais autonomia."

---

## 5. Como você resolve conflito técnico?

**História real:**
"Eu tive uma situação na FiTEC em que houve um conflito entre Frontend e Backend sobre a responsabilidade por uma determinada regra de negócio.

O contexto era um sistema de gestão de usuários que funcionava praticamente como uma caixa-preta. Ele não disponibilizava uma API para que os outros sistemas simplesmente consultassem as permissões. Dentro desse sistema eram configurados os usuários, os sistemas que eles podiam acessar e também quais módulos e ações cada usuário poderia executar.

Como resultado, quando o usuário era autenticado, esse sistema devolvia essas informações dentro de um token criptografado. O Frontend precisava receber esse token, utilizá-lo no fluxo da aplicação e encaminhá-lo para o Backend. O Backend, por sua vez, também fazia suas próprias validações antes de permitir determinadas operações.

O conflito surgiu porque o time de Frontend não queria assumir o processamento dessa regra de negócio, entendendo que aquela responsabilidade deveria estar no Backend.

Em vez de simplesmente decidir com base em uma preferência técnica, procurei entender primeiro por que a arquitetura tinha chegado àquele ponto. Como o time era relativamente novo naquele módulo, buscamos também o arquiteto que conhecia o histórico da solução.

Ao analisar o contexto, ficou claro que não era simplesmente uma questão de escolher entre Frontend ou Backend. Existia uma limitação arquitetural importante: o sistema responsável pela gestão de usuários não possuía uma API que permitisse ao Backend assumir sozinho todo aquele fluxo. O Frontend já fazia parte desse processo porque precisava receber e transportar o token gerado por esse sistema.

A partir disso, conseguimos separar melhor as responsabilidades. O Frontend precisava lidar com aquela parte do fluxo por uma limitação e uma decisão arquitetural existente, enquanto o Backend continuava responsável pelas validações necessárias para garantir que a operação fosse autorizada.

O mais importante para mim nessa situação foi não transformar a discussão em uma disputa de opiniões entre Frontend e Backend. Primeiro procurei entender a arquitetura existente, as limitações do sistema, o motivo daquela decisão e os impactos de mudar o fluxo.

Minha forma de resolver conflitos técnicos é justamente essa: **tirar a discussão do campo da opinião e levar para contexto, responsabilidades, limitações arquiteturais, riscos e impacto da decisão.**"

**Mensagem principal:** "Eu procuro tirar a discussão do campo da opinião e entender contexto, origem da decisão, impacto e responsabilidade de cada camada."

---

## 6. Como orienta um desenvolvedor menos experiente (júnior)?

**Resposta-base:** "Eu começo entendendo a maturidade e o nível de autonomia do desenvolvedor. Com alguém mais júnior, normalmente acompanho mais de perto, faço perguntas, explico os motivos das decisões e, quando necessário, faço pair programming. Conforme percebo que a pessoa ganhou autonomia, reduzo a intervenção e deixo que ela conduza mais o trabalho, continuando disponível para apoiar quando necessário."

**Frase forte:** "Meu objetivo não é criar dependência, mas ajudar a pessoa a desenvolver autonomia."

⚠️ Cuidado: evitar dizer "fico longe", substituir por "reduzo a intervenção conforme a autonomia aumenta".

---

## 7. Como faz code review?

"Eu não vejo code review apenas como procurar erros. Primeiro verifico se a solução atende ao requisito e está alinhada à arquitetura. Depois avalio qualidade, legibilidade, boas práticas da linguagem, separação de responsabilidades, tratamento de erros, segurança, testes e impactos futuros. Procuro fazer comentários construtivos, explicando o motivo, porque review também é uma forma de compartilhar conhecimento."

---

## 8. Como decide entre duas soluções?

"Comparo complexidade, manutenção, segurança, performance, custo operacional, impacto no prazo, aderência à arquitetura e risco. Não necessariamente escolho a solução mais sofisticada, mas a que resolve o problema com complexidade adequada ao contexto."

---

## 9. Como lida com débito técnico?

Primeiro procuro tornar o débito técnico **visível**.

Avalio o impacto dele em:

- risco de incidente;
- velocidade de desenvolvimento;
- manutenção;
- segurança;
- capacidade de evolução.

Se o débito aumenta significativamente o risco ou começa a bloquear evolução, priorizo sua correção.

Se não for possível resolver imediatamente, registro, acompanho e procuro encaixá-lo no planejamento.

**Frase-chave:**

> “Débito técnico não é simplesmente código feio. É uma decisão ou condição que gera custo futuro. O importante é tornar esse custo visível e administrar o risco.”

---

## 10. Como estima uma tarefa?

"Primeiro procuro entender o requisito e as prioridades com Produto. Depois quebro em tarefas menores, identifico dependências, integrações e riscos. A estimativa considera a capacidade real do time e o período da sprint. Se houver muita incerteza, deixo isso explícito. A estimativa é do time, não apenas minha."

---

## 11. Como identifica riscos?

"Durante o refinamento, procuro identificar dependências, integrações externas, alterações de banco, infraestrutura, segurança e pontos ainda pouco claros. Avalio o impacto e, quando o risco pode comprometer prazo, qualidade ou estabilidade, antecipo e comunico ao time e ao Produto."

---

## 12. Como lida com incidente?

"Primeiro procuro reduzir o impacto. Depois verifico métricas, logs e alarmes e correlaciono com alterações recentes. Se houver uma regressão clara e uma versão estável disponível, faço rollback. Com o sistema estabilizado, investigamos causa raiz e aplicamos a correção definitiva."

---

## 13. Como trabalha com Produto/Negócio?

"Procuro trabalhar próximo do PO para entender o problema de negócio, não apenas o que precisa ser desenvolvido. Ajudo na priorização, identificação de riscos e dependências e decomposição em cards/tarefas. Quando existe uma limitação técnica, procuro não simplesmente dizer não: apresento alternativas e deixo claros os trade-offs."

**Frase forte:** "Meu papel técnico não é simplesmente dizer não; é apresentar alternativas e deixar claros os trade-offs."

---

## 14. Como lida com pressão?

"Primeiro procuro entender o que realmente é prioridade e o que pode esperar. Quando existe muita coisa acontecendo ao mesmo tempo, tento organizar as demandas pelo impacto, urgência e dependências. Também acho importante comunicar quando uma pressão de prazo pode comprometer qualidade ou gerar risco. Prefiro deixar claro o trade-off e buscar uma solução junto ao time do que simplesmente transferir a pressão para os desenvolvedores."

**Pontos de apoio, caso precise detalhar:**

- Separar urgência de importância
- Negociar prazo/escopo quando necessário
- Manter qualidade mínima necessária

---

## 15. Como recebe feedback?

"Procuro entender o feedback antes de reagir. Mesmo quando não concordo inicialmente, tento identificar o que posso aprender e, se necessário, pergunto por exemplos concretos. Feedback para mim é ferramenta de evolução."

---

## 16. Como dá feedback?

**Estrutura:**

- Privado
- Específico
- Baseado em comportamento/fato
- Impacto
- Orientação para melhoria
- Acompanhamento

⚠️ Evitar julgamento pessoal.

---

## 17. Como reage quando discordam de você / contestam sua decisão?

"Eu procuro entender o contexto antes de defender minha opinião. Para mim, o contexto e as necessidades do projeto são mais importantes do que estar certo. Se a outra pessoa apresentar um argumento melhor, não tenho problema em mudar de posição. E se eu continuar acreditando na minha proposta, explico os motivos e os impactos para chegarmos à melhor decisão para o projeto."

**Mensagem-chave:** o importante é a melhor decisão para o projeto, não ganhar uma discussão.

---

## 18. Como lida com pessoa resistente?

"Primeiro procuro entender o motivo da resistência. Nem sempre resistência significa falta de colaboração; às vezes a pessoa tem uma preocupação que ainda não foi considerada. Converso, tento entender o ponto de vista e explico o objetivo da decisão. Se houver um argumento melhor, não tenho problema em mudar de posição. Se a decisão continuar sendo a melhor para o projeto, procuro alinhar isso com a pessoa e acompanhar a execução."

⚠️ Escalar somente quando necessário.

---

## 19. Como organiza múltiplas demandas?

"Principalmente por prioridade e impacto. Procuro entender o que é mais importante para o negócio, quais são as dependências e quais demandas podem bloquear outras. Também considero a capacidade do time para não colocar tudo como prioridade ao mesmo tempo. Quando existe conflito entre prioridades, procuro alinhar com Produto e deixar claro o impacto de cada escolha."

⚠️ Não tentar fazer tudo ao mesmo tempo.

---

## 20. Mudança de prioridade

"Procuro entender o motivo da mudança e o impacto sobre o que já está em andamento. Reavalio prioridades com Produto, verifico dependências e capacidade e deixo claro o que será adiado ou alterado. Assim a mudança fica explícita em vez de simplesmente adicionar trabalho ao time."

---

## 21. Conte um erro.

> Foi deixar uma responsabilidade muito diluída dentro do time, sem deixar claramente definido quem seria responsável por algumas partes importantes da entrega. Percebemos já próximo da entrega que algumas configurações relacionadas a permissões e autorização ainda não estavam prontas. Isso era especialmente crítico porque o sistema precisava garantir que um vendedor não pudesse visualizar informações que pertenciam a outro vendedor, como preços e dados da operação. Como o prazo estava muito próximo, precisei assumir diretamente a situação e trabalhar em cima da solução em um tempo muito curto para garantir que a entrega não comprometesse a segurança do sistema. O principal aprendizado para mim foi que não basta dividir uma responsabilidade entre várias pessoas; é importante existir uma pessoa claramente responsável por cada entrega crítica e acompanhar esses pontos durante o desenvolvimento, principalmente quando envolvem segurança, autorização ou dependências que podem bloquear a entrega.
> 
> Depois disso, passei a dar mais atenção à definição clara de responsáveis, acompanhamento dos pontos críticos e validação antecipada das configurações de autorização, evitando deixar esse tipo de preocupação para o final.

---

## 22. O que espera de uma liderança?

"Eu valorizo uma liderança que deixe as expectativas e objetivos claros, que mantenha uma boa comunicação com o time e dê feedbacks sobre nosso trabalho. Também considero importante ter clareza sobre possibilidades de evolução e progressão de carreira. Gosto de saber o que é esperado de mim, quais são os objetivos e o que preciso desenvolver para crescer profissionalmente."

---

## 23. Onde quer estar profissionalmente?

"Quero continuar evoluindo tecnicamente e chegar a uma posição de maior responsabilidade, provavelmente seguindo uma trajetória mais próxima de arquitetura de software ou de liderança técnica. Também tenho interesse em aspectos de agilidade e gestão de fluxo, mas não quero me afastar completamente do técnico. Gosto de estar próximo do desenvolvimento e das decisões de arquitetura."

---

## 24. Por que saiu da empresa anterior / está buscando nova oportunidade?

"O projeto passou por uma mudança de gestão e, com a nova gestão, houve uma reorganização da equipe. O novo responsável trouxe profissionais com quem já trabalhava anteriormente, e isso acabou gerando uma mudança na composição do time. Foi uma situação mais relacionada à reorganização do projeto do que ao meu desempenho. A partir disso, comecei a buscar uma nova oportunidade para continuar minha evolução profissional."

---

## 25. Tem disponibilidade para viagens?

"Sim, tenho disponibilidade para viagens quando houver necessidade da posição."

---

## 26. Como funciona sua disponibilidade para trabalho híbrido?

"Tenho disponibilidade para o modelo híbrido. Minha mudança para Curitiba já está planejada para este mês e, estando na cidade, a localização onde vou ficar fica aproximadamente 30 minutos do local de trabalho."

---

## 27. Como responder depois de dois anos fora

Não pedir desculpas.

Não dizer:

> "Estou enferrujado."

Preferir:

> Fiquei aproximadamente dois anos fora do mercado formal, mas continuei estudando e desenvolvendo projetos. Nesse período aprofundei bastante conhecimentos em Go, arquitetura, AWS, CI/CD, testes e sistemas distribuídos. Estou justamente buscando voltar ao mercado formal levando essa bagagem técnica para uma posição em que eu possa contribuir.

---

## 28. O que fazer quando não lembrar

Não inventar.

Frases úteis:

> "Não lembro exatamente qual implementação usamos naquela época, mas lembro do conceito."

> "Não tive experiência prática profunda nessa ferramenta, mas conheço o conceito e consigo explicar como eu abordaria."

> "Essa parte específica eu precisaria revisar, mas o princípio que eu seguiria seria..."

> "Não quero afirmar algo que não tenho certeza."

Isso passa mais segurança do que tentar adivinhar.

---

## 29. Liderança: conceito central do seu discurso

- Cobrar sem pressionar excessivamente
- Apoiar sem fazer pelo outro
- Criar confiança
- Reduzir resistência
- Desenvolver autonomia
- Manter responsabilidade por resultado

**Resumo:** "Liderança técnica, para mim, é equilibrar exigência e suporte. Eu cobro resultado, mas também procuro entender o que está impedindo a pessoa ou o time de entregar e ajudar a remover esse obstáculo."

---

## 30. Histórias reais para usar

| História                                                    | Tema                                                                        |
| ----------------------------------------------------------- | --------------------------------------------------------------------------- |
| Conflito Front x Back                                       | conflito, arquitetura, escuta, influência                                   |
| Dev júnior / problema em alteração (ex. aggregate do Mongo) | feedback, processo, testes, aprendizado, não focar em culpa                |
| Mentoria                                                    | acompanhamento conforme maturidade, pair programming, autonomia             |
| Code review                                                 | qualidade, arquitetura, segurança, testes, compartilhamento de conhecimento |
| Incidente                                                   | calma, mitigação, observabilidade, rollback, causa raiz                     |

---

## 31. Regra de ouro para respostas comportamentais

**Usar:** Contexto → ação → resultado → aprendizado.

Não decorar texto palavra por palavra. Decorar a história e os pontos principais.

---

## 32. Postura

- Não tentar parecer perfeito.
- Não falar mal de pessoas/empresas.
- Não culpar júnior.
- Não transformar liderança em autoridade.
- Não responder com excesso de jargão.
- Dar exemplos reais.
- Mostrar aprendizado.
- Admitir quando não sabe algo.
- Demonstrar segurança sem arrogância.
