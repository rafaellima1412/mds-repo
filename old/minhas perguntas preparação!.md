
Como trata falha de integração?
Primeiro diferencio falhas transitórias de falhas permanentes.

Como funciona seu CI/CD?
Tenho experiência com AWS utilizando **CodePipeline, CodeBuild, ECR, ECS, Lambda, API Gateway, RDS, S3 e CloudFormation**, entre outros serviços.
O pre-commit ajuda na validação local, mas as validações importantes também precisam ser executadas no CI.
“A ideia é reduzir intervenção manual e tornar a entrega repetível, testável e rastreável.”

Como faria rollback?
Primeiro preciso ter uma versão estável conhecida.
Se o novo deploy apresentar uma regressão relevante, posso interromper a progressão e voltar para a versão anterior.
O rollback deve ser rápido e previsível.
Também procuro correlacionar o problema com o deploy, métricas e logs antes de tomar a decisão, sem deixar o usuário sofrendo enquanto investigamos eternamente.

Como investiga incidente em produção?
Minha primeira preocupação é **reduzir o impacto para o usuário**.
Depois verifico:
1. métricas;
2. logs;
3. alertas;
4. health checks;
5. traces, quando disponíveis;
6. alterações recentes;
7. último deploy.
Se houver forte evidência de regressão e uma versão estável disponível, faço rollback.
Depois que o sistema estiver estabilizado, parto para a investigação da causa raiz e para a correção definitiva.
**Resposta forte:**

> “Primeiro estabilizo o sistema e reduzo o impacto. Depois investigo a causa raiz. Não quero que uma investigação tecnicamente interessante mantenha o usuário sofrendo.”

Como escolhe entre duas arquiteturas?
Não escolho automaticamente a arquitetura mais sofisticada.
Comparo:
- complexidade;
- manutenção;
- segurança;
- performance;
- custo operacional;
- prazo;
- risco;
- contexto de negócio;
- capacidade do time.
A solução precisa ser proporcional ao problema.
**Frase forte:**

> “Eu não escolho necessariamente a solução mais sofisticada, mas a solução adequada ao contexto e com complexidade proporcional ao problema.”

Exemplo: nem todo problema precisa de micros serviços. Às vezes um monólito bem estruturado é a decisão mais correta.

Como trabalha débito técnico?
Primeiro procuro tornar o débito técnico **visível**.
Avalio o impacto dele em:
- risco de incidente;
- velocidade de desenvolvimento;
- manutenção;
- segurança;
- capacidade de evolução.
Se o débito aumenta significativamente o risco ou começa a bloquear evolução, priorizo sua correção.
Se não for possível resolver imediatamente, registro, acompanho e procuro encaixá-lo no planejamento.
**Frase chave:**

> “Débito técnico não é simplesmente código feio. É uma decisão ou condição que gera custo futuro. O importante é tornar esse custo visível e administrar o risco.”
> 

Como estima uma tarefa?
Eu não gosto de estimar apenas olhando o título da tarefa.
Meu processo:
1. entender o requisito;
2. alinhar prioridade com Produto;
3. quebrar a tarefa em partes menores;
4. identificar dependências;
5. identificar riscos;
6. avaliar a capacidade real do time;
7. estimar conforme o método utilizado pela equipe;
8. deixar explícitas as incertezas.
**Frase chave:**

> “A estimativa é do time, não apenas minha.”

Se houver muita incerteza, eu sinalizo isso em vez de inventar uma precisão que não existe.


Como orienta outro desenvolvedor?
Eu começo entendendo o problema e o nível de contexto que a pessoa já possui.
Procuro não simplesmente entregar a solução pronta. Explico o **porquê** da decisão e mostro os impactos.
Também utilizo:
- code review;
- pair programming quando necessário;
- exemplos;
- decomposição de problemas;
- documentação;
- feedback.
No Code Review, avalio:
- requisito;
- arquitetura;
- legibilidade;
- boas práticas;
- separação de responsabilidades;
- tratamento de erros;
- segurança;
- testes;
- impactos futuros.
Evito transformar review em julgamento pessoal.
**Frase-chave:**

> “Meu objetivo ao orientar alguém é aumentar a autonomia da pessoa, não criar dependência de mim.”

GIT
Conhecer a diferença entre estratégia de branches e CI/CD.
Conceitos importantes:
- feature branches;
- Pull Request;
- Code Review;
- GitFlow;
- trunk-based development;
- merge;
- rebase;
- tags;
- releases.
**Importante:** Git/branching é uma parte do processo de desenvolvimento. CI/CD é o processo automatizado de validação e entrega.
# RESPOSTA CORINGA

Quando a pergunta for aberta e você precisar organizar o raciocínio:

> “Eu começaria entendendo o requisito e as restrições do problema. Depois avaliaria as alternativas considerando complexidade, manutenção, segurança, performance, custo e risco. A partir disso escolheria a solução mais simples que atende ao contexto, implementaria com testes e observabilidade e acompanharia o comportamento em produção.”

“Por que deveríamos escolher você?”
Resposta-base:
“Acredito que posso contribuir porque sou muito dedicado ao trabalho, mas principalmente porque procuro equilibrar cobrança e suporte ao time. Uma liderança técnica precisa cobrar resultados e manter responsabilidade sobre entregas, mas também entender quando uma pessoa está enfrentando dificuldade e ajudá-la a superar. Cobrança excessiva pode gerar resistência e prejudicar desempenho. Quando o time entende os objetivos, participa das decisões e percebe que a liderança está ali para ajudar a resolver problemas, a colaboração aumenta e a entrega tende a ser melhor.”

Frase forte:
“Meu objetivo não é criar dependência no líder, mas ajudar o time a ganhar cada vez mais autonomia.”

**Como reage quando uma decisão sua é contestada?**
“Eu procuro avaliar os argumentos, independentemente de quem apresenta a ideia. Um exemplo foi quando estávamos implementando uma paginação. Minha proposta inicial era fazer a implementação nativamente na linguagem, mas um desenvolvedor do time encontrou uma biblioteca que oferecia funcionalidades extras que atendiam melhor às nossas necessidades naquele momento.

Em vez de manter minha decisão inicial, avaliei a biblioteca, principalmente em relação à manutenção e ao risco de descontinuidade. Como a solução fazia sentido para o projeto, concordei em adotá-la.

Para mim, ser referência técnica não significa ter sempre a melhor ideia, mas ajudar o time a encontrar a melhor solução. O importante é o resultado do projeto, não quem teve a ideia.”

Como lida com pessoa resistente?
 **“Primeiro procuro entender o motivo da resistência. Nem sempre resistência significa falta de colaboração; às vezes a pessoa tem uma preocupação que ainda não foi considerada. Converso, tento entender o ponto de vista e explico o objetivo da decisão. Se houver um argumento melhor, não tenho problema em mudar de posição. Se a decisão continuar sendo a melhor para o projeto, procuro alinhar isso com a pessoa e acompanhar a execução.”**
 
- conversar individualmente
- entender causa da resistência (que pode estar ligada a uma falsa certeza ou compreensão errada de algo)
- esclarecer objetivo
- ouvir argumentos
- combinar expectativa
- acompanhar
- escalar somente quando necessário

Como organiza múltiplas demandas?
 **“Principalmente por prioridade e impacto. Procuro entender o que é mais importante para o negócio, quais são as dependências e quais demandas podem bloquear outras. Também considero a capacidade do time para não colocar tudo como prioridade ao mesmo tempo. Quando existe conflito entre prioridades, procuro alinhar com Produto e deixar claro o impacto de cada escolha.”**
- prioridade de negócio
- urgência

Mudança de prioridade?
“Procuro entender o motivo da mudança e o impacto sobre o que já está em andamento. Reavalio prioridades com Produto, verifico dependências e capacidade e deixo claro o que será adiado ou alterado. Assim a mudança fica explícita em vez de simplesmente adicionar trabalho ao time.”

O que espera de uma liderança?
> **“Eu valorizo uma liderança que deixe as expectativas e objetivos claros, que mantenha uma boa comunicação com o time e dê feedbacks sobre nosso trabalho. Também considero importante ter clareza sobre possibilidades de evolução e progressão de carreira. Gosto de saber o que é esperado de mim, quais são os objetivos e o que preciso desenvolver para crescer profissionalmente.”**

Onde quer estar profissionalmente?
> **“Quero continuar evoluindo tecnicamente e chegar a uma posição de maior responsabilidade, provavelmente seguindo uma trajetória mais próxima de arquitetura de software ou de liderança técnica. Também tenho interesse em aspectos de agilidade e gestão de fluxo, mas não quero me afastar completamente do técnico. Gosto de estar próximo do desenvolvimento e das decisões de arquitetura.”**
 
  Por que sairia/entrou na empresa?
> **“O projeto passou por uma mudança de gestão e, com a nova gestão, houve uma reorganização da equipe. O novo responsável trouxe profissionais com quem já trabalhava anteriormente, e isso acabou gerando uma mudança na composição do time. Foi uma situação mais relacionada à reorganização do projeto do que ao meu desempenho. A partir disso, comecei a buscar uma nova oportunidade para continuar minha evolução profissional.”**

Como lida com pressão?
"Primeiro procuro entender o que realmente é prioridade e o que pode esperar. Quando existe muita coisa acontecendo ao mesmo tempo, tento organizar as demandas pelo impacto, urgência e dependências. Também acho importante comunicar quando uma pressão de prazo pode comprometer qualidade ou gerar risco. Prefiro deixar claro o trade-off e buscar uma solução junto ao time do que simplesmente transferir a pressão para os desenvolvedores."
**Pontos de apoio, caso precise detalhar:**
- Separar urgência de importância
- Negociar prazo/escopo quando necessário
- Manter qualidade mínima necessária

