# Guia de Entrevista Técnica: Conceitos Universais

Conceitos independentes de linguagem. Os detalhes e exemplos de código de cada stack estão nos guias de Python, Go e TypeScript.

> Fase técnica, foco em respostas práticas, objetivas e baseadas em experiência.
>
> Regra geral: **Contexto → decisão → motivo → resultado/aprendizado.**
>
> Evite respostas excessivamente teóricas. Priorize exemplos reais.

---

## Parte 1. Posicionamento e regras de resposta

### Perfil e posicionamento

- Backend com foco em APIs REST, integrações, automações e PostgreSQL.

- Experiência prática com Clean Architecture, Use Cases, Repositories, DI, testes, Docker e AWS.
- Atuação como referência técnica: revisão de PRs, mentoria, decomposição de tarefas, estimativas, riscos, incidentes e decisões arquiteturais.
- Liderança técnica sem depender de autoridade formal.

### REGRA DE OURO DA ENTREVISTA

Não tente parecer um livro.

Você já trabalha com os conceitos. A resposta mais forte é mostrar **como você pensa e toma decisões**.

Quando possível:

```text
PROBLEMA
   ↓
CONTEXTO
   ↓
DECISÃO
   ↓
POR QUE?
   ↓
RESULTADO
   ↓
APRENDIZADO
```

A entrevista técnica não precisa provar que você sabe todos os termos.

Ela precisa mostrar que você consegue **resolver problemas de engenharia com critério**.

### Regra de ouro (versão com experiência real)

Responder:

```text
Experiência real
      ↓
Conceito
      ↓
Decisão
      ↓
Motivo
      ↓
Resultado
```

Evitar respostas excessivamente teóricas.

Priorizar exemplos reais.

### RESPOSTA CORINGA

Quando a pergunta for aberta e você precisar organizar o raciocínio:

> “Eu começaria entendendo o requisito e as restrições do problema. Depois avaliaria as alternativas considerando complexidade, manutenção, segurança, performance, custo e risco. A partir disso escolheria a solução mais simples que atende ao contexto, implementaria com testes e observabilidade e acompanharia o comportamento em produção.”

### O que fazer quando não lembrar

Não inventar.

Frases úteis:

> "Não lembro exatamente qual implementação usamos naquela época, mas lembro do conceito."

> "Não tive experiência prática profunda nessa ferramenta, mas conheço o conceito e consigo explicar como eu abordaria."

> "Essa parte específica eu precisaria revisar, mas o princípio que eu seguiria seria..."

> "Não quero afirmar algo que não tenho certeza."

Isso passa mais segurança do que tentar adivinhar.

### Frase final para lembrar

> "Eu procuro escolher a solução adequada ao contexto, com complexidade proporcional ao problema. Não necessariamente a solução mais sofisticada."

Essa ideia resume bem a postura esperada de alguém com experiência em arquitetura.

---

## Parte 2. Perguntas técnicas prováveis

### Como você estrutura uma API?

Eu procuro separar as responsabilidades. Os **routers** ficam responsáveis pela camada HTTP, validação inicial e entrada da requisição. Eles chamam os **Use Cases**, que concentram o fluxo e as regras de negócio. Os **Repositories** abstraem a persistência.

As dependências são fornecidas por Dependency Injection. (Stack Python: ver Guia Python.)

A ideia é evitar que o router conheça detalhes de banco ou infraestrutura.

**Frase-chave:**

> “Eu procuro manter a regra de negócio independente de infraestrutura. O Use Case coordena o fluxo, o Repository abstrai persistência e as dependências são injetadas. Isso facilita testes e manutenção.”

---

### Por que Use Case e Repository separados?

Porque são responsabilidades diferentes.

O **Use Case** representa uma ação do sistema e concentra o fluxo/regra de negócio. O **Repository** cuida de como os dados são persistidos ou recuperados.

Por exemplo:

```text
CreateUser
    ↓
UserRepository (interface)
    ↓
PostgresUserRepository
```

Assim, o Use Case não depende diretamente do PostgreSQL.

Isso melhora:

- testabilidade;
- baixo acoplamento;
- manutenção;
- possibilidade de trocar infraestrutura;
- clareza de responsabilidades.

**Frase-chave:**

> “O Use Case não precisa saber se os dados estão no PostgreSQL, em outro banco ou em um mock. Ele depende de uma abstração.”

---

### Como testa regra de negócio?

Eu prefiro testar a regra de negócio isoladamente, sem depender do banco ou de serviços externos.

Para isso, o Use Case recebe suas dependências por injeção e, nos testes, posso utilizar **mocks ou fakes** dos repositories e serviços externos.

Exemplo:

```text
Teste
  ↓
Use Case
  ↓
Fake/MOCK Repository
```

Assim consigo verificar cenários de sucesso, validações e falhas sem precisar subir toda a infraestrutura.

---

### Como faria integração com serviço externo?

Primeiro eu isolaria a integração em um componente próprio, evitando espalhar chamadas HTTP pela aplicação.

O fluxo poderia ser:

```text
Use Case
   ↓
Interface do serviço
   ↓
Implementação HTTP
   ↓
Serviço externo
```

Eu trataria:

- timeout;
- autenticação;
- validação da resposta;
- erros HTTP;
- retry quando fizer sentido;
- logs;
- idempotência quando a operação permitir; Outbox_Pattern salvar o usuário e o evento na mesma transação:
- processamento assíncrono quando a integração não precisar ser síncrona.

Se a indisponibilidade do serviço externo não puder bloquear a requisição, colocaria a operação em uma fila.

---

### Como trata falha de integração?

Primeiro diferencio falhas transitórias de falhas permanentes.

Para falhas transitórias, posso utilizar:

- timeout; 
- retry com backoff; após 1 segundo, depois após 2 segundos
- limite de tentativas;
- circuit breaker;
- fila para processamento assíncrono.

Para mensagens que continuam falhando, utilizaria uma **DLQ (Dead Letter Queue)** para evitar processamento infinito.

Um desenho possível:

```text
API / Webhook
      ↓
    valida
      ↓
    Queue
      ↓
    Worker
      ↓
Serviço externo
   ↙       ↘
sucesso    falha
             ↓
           retry
             ↓
            DLQ
```

A decisão depende da criticidade e do comportamento esperado da integração.

---

### Como faria um webhook resiliente?

Eu evitaria fazer todo o processamento dentro da requisição HTTP.

O fluxo seria:

```text
Webhook
   ↓
Validar requisição
   ↓
Publicar evento na fila
   ↓
Responder 2xx rapidamente
   ↓
Worker processa
   ↓
Integração / regra de negócio
```

Isso reduz o tempo de resposta e desacopla o recebimento do processamento.

Também cuidaria de:

- autenticação/assinatura;
- validação do payload;
- IDEMPOTÊNCIA

Se perguntarem sobre webhooks ou filas:

Uma mesma mensagem pode chegar mais de uma vez. Por isso, o processamento precisa ser idempotente quando o domínio permitir.

Uma abordagem é utilizar um identificador único do evento e registrar o processamento de forma que uma segunda tentativa não produza o mesmo efeito novamente.

```text
Evento 123
   ↓
Processa
   ↓
Marca evento 123 como processado

Evento 123 novamente
   ↓
Detecta que já foi processado
   ↓
Não duplica operação
```

---

- retry;
- DLQ;
- logs e métricas.

**Frase-chave:**

> “Eu recebo a notificação, valido a requisição, publico para processamento assíncrono e respondo rapidamente. O worker fica responsável pelo processamento e pelos retries.”

---

### Como garante autorização?

Primeiro separo **authentication** de **authorization**.

- Authentication: **quem é você?**
- Authorization: **o que você pode acessar ou fazer?**

Um token válido não significa automaticamente que o usuário pode acessar qualquer recurso.

Por exemplo, depois de validar o token, eu verificaria se aquele usuário possui permissão para acessar determinado recurso.

Também considero:

- roles/permissões;
- escopo da operação;
- ownership do recurso;
- princípio do menor privilégio.

**Ponto importante:**

> “Não confio apenas no fato de o token ser válido. Também preciso verificar se aquele usuário pode executar aquela ação naquele recurso.”

---

### Como evita SQL Injection?

Evito construir SQL concatenando diretamente dados fornecidos pelo usuário.

Por exemplo, não montaria a query concatenando o valor recebido do usuário dentro do texto SQL (exemplo em código no Guia Python).

Prefiro ORM ou queries parametrizadas (prepared statements).

```text
Entrada do usuário
      ↓
Parâmetro
      ↓
Query parametrizada
      ↓
Banco
```

O ORM ajuda, mas o ponto fundamental é **não misturar input do usuário diretamente com SQL executável**.

---

### Como funciona seu CI/CD?

Um fluxo que utilizo pode ser:

```text
Branch
  ↓
Implementação
  ↓
Testes / lint / pre-commit
  ↓
Pull Request
  ↓
Code Review
  ↓
QA
  ↓
Pipeline
  ↓
Build
  ↓
Docker Image
  ↓
ECR
  ↓
Deploy
  ↓
Monitoramento
```

Tenho experiência com AWS utilizando **CodePipeline, CodeBuild, ECR, ECS, Lambda, API Gateway, RDS, S3 e CloudFormation**, entre outros serviços.

O pre-commit ajuda na validação local, mas as validações importantes também precisam ser executadas no CI.

**Frase-chave:**

> “A ideia é reduzir intervenção manual e tornar a entrega repetível, testável e rastreável.”

---

### Como faria rollback?

Primeiro preciso ter uma versão estável conhecida.

Se o novo deploy apresentar uma regressão relevante, posso interromper a progressão e voltar para a versão anterior.

```text
V1, estável
 ↓
Deploy V2
 ↓
Problema
 ↓
Rollback
 ↓
V1
```

O rollback deve ser rápido e previsível.

Também procuro correlacionar o problema com o deploy, métricas e logs antes de tomar a decisão, sem deixar o usuário sofrendo enquanto investigamos eternamente.

---

### Rolling x Canary?

#### Rolling

A nova versão substitui gradualmente as instâncias antigas.

```text
V1 V1 V1
 ↓
V2 V1 V1
 ↓
V2 V2 V1
 ↓
V2 V2 V2
```

A mudança acontece progressivamente.

#### Canary

A nova versão recebe inicialmente uma pequena porcentagem do tráfego.

```text
V1 → 95%
V2 → 5%

depois:

V1 → 80%
V2 → 20%

depois:

V1 → 50%
V2 → 50%

final:

V2 → 100%
```

Se as métricas indicarem problema, interrompo a progressão e faço rollback.

**Diferença simples:**

> Rolling troca as instâncias gradualmente. Canary controla a exposição da nova versão a uma parcela do tráfego.

---

### Como investiga incidente em produção?

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

---

### Como escolhe entre duas arquiteturas?

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

Exemplo: nem todo problema precisa de microsserviços. Às vezes um monólito bem estruturado é a decisão mais correta.

---

### Como trabalha débito técnico?

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

### Como estima uma tarefa?

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

**Frase-chave:**

> “A estimativa é do time, não apenas minha.”

Se houver muita incerteza, eu sinalizo isso em vez de inventar uma precisão que não existe.

---

### Como orienta outro desenvolvedor?

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

> **Nota sobre prepared statements:** Outra vantagem dos _Prepared Statements_ é que, se utilizados corretamente, ajudam a evitar ataques de Injeção de SQL. **Note que para isso** é preciso que os parâmetros da consulta sejam atribuídos através dos métodos `setInt()`, `setString()`, etc. presentes na interface `PreparedStatement` e não por concatenação de strings.

### MAPA RÁPIDO PARA REVISÃO

| Pergunta | Palavra-chave |
|---|---|
| Estrutura de API | Router/Controller → Use Case → Repository |
| Use Case x Repository | Separação de responsabilidades |
| Testes | mocks/fakes |
| Serviço externo | Timeout + retry + isolamento |
| Falha de integração | Retry + circuit breaker + fila |
| Webhook | Validar → Queue → 2xx → Worker |
| Autorização | Permissão além do token |
| SQL Injection | ORM / queries parametrizadas |
| CI/CD | Test → Build → ECR → Deploy → Monitor |
| Rollback | Voltar para versão estável |
| Rolling | Substituição gradual |
| Canary | Pequena porcentagem de tráfego |
| Incidente | Mitigar → observar → rollback → RCA |
| Arquitetura | Contexto + risco + complexidade |
| Débito técnico | Tornar custo/risco visível |
| Estimativa | Quebrar + dependências + riscos |
| Mentoria | Aumentar autonomia |

---

## Parte 3. Arquitetura

### Clean Architecture

##### Explicação curta

> A ideia é proteger a regra de negócio. O centro tem as entidades e os casos de uso, que não conhecem framework web, ORM nem banco. Em volta ficam os adaptadores, como controllers e implementações de repositório. A dependência sempre aponta para dentro.

##### Exemplo mental

```text
Controller
    ↓
Use Case
    ↓
Repository Interface
    ↓
Repository Implementation
    ↓
ORM / Banco
```

O Use Case não precisa saber qual ORM ou banco está por trás (exemplos de ORMs por linguagem nos guias específicos).

##### Ligação com testes

No teste:

```text
CriarUsuario
     ↓
Mock UserRepository
```

Assim é possível testar a regra de negócio sem banco, ORM ou HTTP.

##### Frase para entrevista

> A Clean Architecture facilita os testes porque consigo testar o caso de uso isoladamente, substituindo infraestrutura como banco e ORM por mocks.

##### Trade-off

> Gera mais arquivos e abstrações. Para um CRUD simples pode ser exagero; vale quando a regra de negócio é relevante.

### SOLID
#### SRP
Uma responsabilidade bem definida.

#### Dependency Inversion
O Use Case depende de uma abstração, não de PostgreSQL diretamente.

Exemplo:
CreateUser → UserRepository (interface) → PostgresUserRepository

#### Por que isso importa?
- Testabilidade
- Baixo acoplamento
- Manutenção
- Troca de infraestrutura
- Clareza de responsabilidades

### DDD

Revisar apenas o essencial:

- domínio;
- entidades;
- value objects;
- agregados;
- serviços de domínio;
- bounded contexts.

Não decorar termos por decorar.

A pergunta importante é:

> “Qual problema esse conceito resolve?”

---

## Parte 4. Testes

### Pirâmide de testes

A ideia utilizada nos meus projetos:

```text
             E2E
          /       \
       Integração
      /             \
    Unitários, muitos
```

- Muitos testes unitários
- Menos testes de integração
- Poucos testes E2E

Definições rápidas:

- **Testes unitários**: Validam o menor pedaço de código isolado, como uma função ou método específico.
- **Testes de integração**: Verificam se diferentes módulos ou sistemas conversam bem entre si.
- **Testes ponta a ponta (E2E)**: Simulam a ação real de um usuário navegando por toda a aplicação

### TDD: ciclo

```text
RED
 ↓
Escrever teste que falha
 ↓
GREEN
 ↓
Implementar o mínimo
 ↓
REFACTOR
 ↓
Melhorar o código
```

### Ligação com Clean Architecture

Casos de uso isolados são mais fáceis de testar:

```text
Use Case
   ↓
Interface Repository
   ↓
Mock
```

O teste não precisa subir banco nem depender do ORM.

---

## Parte 5. APIs, banco de dados e webhooks

### APIs REST: o que revisar
Revisar:
- HTTP methods
- Status codes
- autenticação/autorização
- validação
- paginação
- tratamento de erros
- versionamento quando necessário
- idempotência em operações apropriadas

### Status codes

- `200` OK
- `201` Created
- `204` No Content
- `400` Bad Request
- `401` Unauthorized
- `403` Forbidden
- `404` Not Found
- `409` Conflict
- `422` Unprocessable Entity
- `500` Internal Server Error

### Webhooks
#### Conceito
Um sistema notifica outro quando um evento acontece.

#### Experiência Microsoft Graph (Conecta serviços: como Outlook)
- Subscription para eventos.
- Endpoint `/webhook/events`.
- Validação por `validationToken`.
- Processamento do payload.
- Tratamento de eventos `deleted`.
- Renovação periódica porque subscriptions expiram.

#### Boa resposta
“Webhook é um mecanismo orientado a eventos. Eu recebo a notificação, valido a requisição, processo ou publico para processamento assíncrono e respondo rapidamente. Como subscriptions podem expirar, também é necessário cuidar da renovação.”

### Banco de dados

#### Pontos para revisar

- Índices
- Transações
- Constraints
- Foreign Keys
- N+1
- Paginação
- SQL Injection
- Connection Pool
- Queries complexas
- Performance

##### ORM x SQL nativo

> Uso ORM para operações comuns e produtividade. Quando tenho uma consulta muito específica, complexa ou que exige maior controle de performance, posso utilizar SQL nativo.

---

## Parte 6. Resiliência, filas e idempotência

Implementação completa em código (retry, backoff, jitter, circuit breaker, worker pool) no Guia Go.

### Resiliência

Conceitos para lembrar:

```text
Timeout
   ↓
Retry
   ↓
Backoff
   ↓
Jitter
   ↓
Circuit Breaker
   ↓
Idempotência
```

##### Retry

Nem todo erro merece retry.

Erro permanente:

```text
400 / validação
→ não repetir
```

Erro transitório:

```text
timeout / 503
→ pode repetir
```

##### Idempotência

A mesma mensagem pode chegar mais de uma vez.

> Uso um identificador único do evento e uma garantia de processamento único, normalmente apoiada por constraint ou controle transacional.

Não aprofundar em Outbox a menos que o entrevistador peça.

### Filas e eventos

Quando uma integração externa pode sofrer indisponibilidade:

```text
Webhook
   ↓
Validação
   ↓
Fila
   ↓
2xx
   ↓
Worker
   ↓
Processamento
```

Benefícios:

- Desacoplamento
- Processamento assíncrono
- Retry
- Resiliência
- DLQ

##### DLQ

Mensagens que continuam falhando depois das tentativas podem ser direcionadas para uma Dead Letter Queue para investigação.

---

## Parte 7. Docker, Git, CI/CD, AWS e escalabilidade

### Docker

##### Resposta curta

> Uso Docker para padronizar o ambiente e garantir que aplicação e dependências tenham comportamento consistente entre desenvolvimento, CI e produção.

##### Conceitos

- Dockerfile
- Image
- Container
- Volume
- Network
- Docker Compose
- Multi-stage build

- **Docker no dia a dia:** mesma imagem em todos os ambientes; docker-compose para subir API e banco localmente; multi-stage build para imagem menor.

### Git

Conhecer:
- feature branches=Boas Práticas para Nomear
- PR
- code review
- GitFlow =é um **modelo de gestão de ramos (branching model)** para o Git
- trunk-based development =Se precisares de criar uma branch para uma tarefa (`feature/*`), ela deve durar no máximo **um ou dois dias**
- merge/rebase = usado no trunk-based development o dev deve ficar atualizando constantemente sua brach.
- tags/releases = tags de versão.

Não confundir estratégia de branches com CI/CD.

#### GitFlow

```text
feature
   ↓
develop
   ↓
release
   ↓
main/master
```

Hotfix sai para correções urgentes.

**Importante:** GitFlow é estratégia de branches. Não é CI/CD.

#### Code Review

> Para mim, code review não é apenas procurar erro de sintaxe. É uma oportunidade de verificar regra de negócio, arquitetura, segurança, testes, legibilidade e manutenção futura.

### CI/CD e deploy

O fluxo completo, rollback e rolling x canary estão na Parte 2.

**Importante:**
Pre-commit é uma validação local. CI deve repetir as validações no servidor.

### AWS

Usar principalmente a experiência da FITEC. Não transformar a resposta em lista de serviços. Explicar o problema que cada serviço resolveu.

**Serviços com experiência:**
Experiência com:
- CodePipeline
- CodeBuild
- ECR
- ECS
- S3
- CloudFront
- API Gateway
- Lambda
- RDS PostgreSQL
- CloudFormation
- SSM
- Secrets Manager
- CloudWatch

**Outros serviços que posso mencionar como conceito:** SQS, SNS, Parameter Store.

### Escalabilidade e disponibilidade

Conceitos:

- API stateless
- Escalabilidade horizontal
- Cache
- Índices
- Filas
- Processamento assíncrono
- Observabilidade

##### Exemplo

Envio de e-mail pode ser assíncrono:

```text
API
 ↓
Queue
 ↓
Worker
 ↓
E-mail
```

Assim uma indisponibilidade do serviço externo não precisa bloquear a requisição principal.

---

## Parte 8. Observabilidade e SRE

### SRE

Se perguntarem **“O que você entende por SRE?”**:

SRE é uma abordagem de engenharia para tratar confiabilidade como parte do desenvolvimento e da operação do software.

Eu penso principalmente em:

- disponibilidade;
- latência;
- taxa de erro;
- capacidade;
- monitoramento;
- logs;
- métricas;
- traces;
- alertas;
- health checks;
- resposta a incidentes.

**Frase-chave:**

> “Não basta o sistema funcionar; precisamos conseguir perceber quando ele está degradando.”

O roteiro de investigação de incidente está na Parte 2.

---

## Parte 9. Segurança

### OWASP

Se perguntarem **“Quais conceitos de OWASP você conhece?”**:

Eu não tento decorar a lista inteira. Penso principalmente nos riscos que podem aparecer no sistema que estou desenvolvendo.

Para uma API backend, eu presto atenção principalmente em:

- controle de acesso;
    
- autenticação;
    
- SQL Injection;
    
- XSS;
    
- validação de entrada;
    
- proteção de dados sensíveis;
    
- configuração segura;
    
- dependências vulneráveis;
    
- logs e monitoramento;
    
- segurança nas integrações.
    

---

#### SQL Injection

O problema acontece quando dados fornecidos pelo usuário acabam sendo interpretados como parte do SQL.

Eu evito construir SQL concatenando entrada do usuário.

Não montaria a query concatenando o valor recebido do usuário dentro do texto SQL (exemplo em código no Guia Python).

Prefiro utilizar ORM ou queries parametrizadas.

```text
Entrada do usuário
      ↓
Parâmetro
      ↓
Query parametrizada
      ↓
Banco
```

A ideia principal é separar **dados** de **comandos SQL**.

**Frase-chave:**

> “Eu não confio que a entrada do usuário seja segura. Ela precisa ser tratada como dado e nunca ser misturada diretamente com SQL executável.”

---

#### XSS: Cross-Site Scripting

XSS acontece quando uma aplicação permite que conteúdo fornecido pelo usuário seja interpretado como código no navegador.

Um exemplo seria uma aplicação receber um comentário contendo código JavaScript e depois renderizar esse conteúdo sem o tratamento adequado.

Existem alguns tipos importantes:

- Stored XSS;
    
- Reflected XSS;
    
- DOM-based XSS.
    

De forma simples:

```text
Entrada do usuário
      ↓
Aplicação não trata corretamente
      ↓
Conteúdo chega ao navegador
      ↓
JavaScript é interpretado
```

Para evitar isso, considero principalmente:

- output encoding;
    
- sanitização quando realmente necessária;
    
- validação de entrada;
    
- evitar inserir conteúdo não confiável diretamente em HTML ou JavaScript;
    
- utilização adequada dos mecanismos de segurança do framework.
    

**Frase-chave:**

> “O problema do XSS é deixar uma entrada controlada pelo usuário ser interpretada pelo navegador como código.”

---

#### Broken Access Control

Uma aplicação pode autenticar corretamente o usuário e ainda assim permitir que ele acesse algo que não deveria.

Por isso eu separo:

```text
Authentication
      ↓
Quem é você?

Authorization
      ↓
O que você pode fazer?
```

Depois de validar a identidade, ainda preciso verificar se aquele usuário pode executar aquela ação naquele recurso.

Considero:

- roles;
    
- permissões;
    
- scopes;
    
- ownership;
    
- princípio do menor privilégio.
    

Por exemplo, não basta o usuário estar autenticado. Se ele tentar acessar o pedido de outro cliente, a aplicação precisa verificar se aquele recurso realmente pertence ao contexto dele.

**Frase-chave:**

> “Estar autenticado não significa ter acesso a tudo.”

---

#### Authentication x Authorization

Eu gosto de separar os dois conceitos.

**Authentication:**

> “Quem é você?”

Pode envolver:

- usuário e senha;
    
- OAuth 2.0; >> entrar com google 
    
- OpenID Connect;
    
- JWT;
    
- identidade fornecida por um Identity Provider.
    

**Authorization:**

> “O que você pode fazer?”

Pode envolver:

- roles;
    
- permissions;
    
- scopes;
    
- ownership;
    
- políticas de acesso.
    

Um token válido não significa automaticamente que o usuário pode executar qualquer operação.

---

#### CSRF ( Ele impede o envio do cookie em requisições cross-site (de outros sites)

CSRF acontece quando um usuário autenticado é induzido a realizar uma operação que não pretendia realizar.

É especialmente importante em aplicações que utilizam autenticação baseada em cookies.

Uma proteção comum é utilizar mecanismos como:

- CSRF token;
    
- SameSite cookies;
    
- validação de origem quando apropriado.
    

Também preciso entender como o mecanismo de autenticação utilizado pela aplicação influencia esse risco.

**Frase-chave:**

> “CSRF explora o fato de o navegador enviar automaticamente determinadas credenciais em uma requisição.”

---

#### SSRF

SSRF acontece quando a aplicação permite que um usuário controle, direta ou indiretamente, uma requisição feita pelo servidor.

Um exemplo seria uma API que recebe uma URL fornecida pelo usuário e simplesmente faz uma requisição para ela.

```text
Usuário
   ↓
API
   ↓
URL fornecida pelo usuário
   ↓
Servidor faz requisição
```

O risco é o servidor conseguir acessar recursos internos que não deveriam estar disponíveis ao usuário.

Para reduzir o risco, considero:

- validação da URL;
    
- allowlist quando possível;
    
- bloqueio de destinos internos;
    
- cuidado com redirects;
    
- restrição de acesso de rede;
    
- menor privilégio.
    

---

#### Cryptographic Failures

Aqui penso principalmente em proteção de dados sensíveis.

Não devo:

- armazenar senha em texto puro;
    
- colocar secrets no código;
    
- expor dados sensíveis em logs;
    
- utilizar criptografia inadequada;
    
- transmitir dados sensíveis sem proteção.
    

Senhas devem utilizar algoritmos apropriados de hashing para senha, e secrets devem ficar em mecanismos próprios de gerenciamento de credenciais.

Na AWS, por exemplo, posso utilizar serviços como Secrets Manager ou Parameter Store dependendo do cenário.

---

#### Security Misconfiguration

Mesmo que o código esteja correto, uma configuração insegura pode abrir uma vulnerabilidade.

Exemplos:

- credenciais padrão;
    
- endpoints administrativos expostos;
    
- permissões excessivas;
    
- debug habilitado em produção;
    
- serviços desnecessários expostos;
    
- configurações inseguras de CORS;
    
- containers ou infraestrutura com privilégios excessivos.
    

Eu procuro aplicar configurações diferentes para desenvolvimento e produção e manter o princípio do menor privilégio.

---

#### Vulnerable and Outdated Components

Também preciso considerar as dependências utilizadas pela aplicação.

Uma aplicação pode ter código próprio correto, mas utilizar uma biblioteca vulnerável.

Por isso considero:

- atualização das dependências;
    
- análise de vulnerabilidades;
    
- controle das versões;
    
- atualização de imagens Docker;
    
- revisão periódica das dependências.
    

No CI/CD, essas verificações podem fazer parte do pipeline.

---

#### Security Logging and Monitoring

Segurança também depende de conseguir perceber comportamentos suspeitos.

Eu considero:

- logs;
    
- métricas;
    
- alertas;
    
- auditoria;
    
- monitoramento;
    
- rastreabilidade.
    

Mas também preciso tomar cuidado para não colocar informações sensíveis nos logs.

Por exemplo:

```text
ERRADO:
senha=123456
token=eyJhbGci...

CERTO:
login_failed
user_id=123
reason=invalid_credentials
```

A ideia é conseguir investigar um incidente sem transformar o próprio log em uma fonte de vazamento.

---

#### Input Validation

Eu não considero a validação de entrada apenas uma preocupação do frontend.

O backend precisa validar aquilo que recebe.

Por exemplo:

```text
Request
   ↓
Validação
   ↓
Autorização
   ↓
Regra de negócio
   ↓
Persistência
```

Bibliotecas de validação ajudam na validação estrutural dos dados recebidos (exemplo Python: Pydantic, ver Guia Python).

Mas validação de formato não substitui autorização ou regra de negócio.

---

#### Secrets

Não colocaria:

```text
DATABASE_PASSWORD = "senha-super-secreta"
```

nem:

```text
.env
```

dentro do repositório.

Prefiro utilizar mecanismos apropriados para armazenamento de secrets e injetá-los na aplicação através da infraestrutura.

Também evito expor secrets em:

- logs;
    
- mensagens de erro;
    
- código;
    
- imagens Docker;
    
- repositórios Git.
    

---

#### Princípio do menor privilégio

Sempre que possível, cada usuário, serviço ou componente deve possuir apenas as permissões necessárias para executar sua função.

Por exemplo:

```text
API
 ↓
IAM Role específica
 ↓
Permissões necessárias
```

Evito dar acesso administrativo completo simplesmente porque é mais fácil.

**Frase-chave:**

> “Se um componente for comprometido, quero limitar o estrago que ele consegue causar.”

---

### Como eu penso em segurança

Eu não vejo segurança como uma etapa que acontece apenas no final do desenvolvimento.

Procuro pensar nela desde:

```text
Requisito
   ↓
Arquitetura
   ↓
Implementação
   ↓
Testes
   ↓
CI/CD
   ↓
Deploy
   ↓
Monitoramento
```

Por exemplo, ao criar uma API eu já penso em:

- quem pode acessar;
    
- quais dados podem ser expostos;
    
- como autenticar;
    
- como autorizar;
    
- como validar entrada;
    
- como proteger banco;
    
- como armazenar secrets;
    
- como registrar eventos;
    
- como detectar problemas.
    

**Frase-chave:**

> “Para mim, segurança não é colocar uma camada no final. É uma preocupação que acompanha o sistema desde o desenho até produção.”

---

### Resposta curta se perguntarem sobre OWASP

> “Eu conheço os principais riscos do OWASP e tento aplicar isso no dia a dia. Penso bastante em controle de acesso, autenticação, SQL Injection, XSS, validação de entrada, secrets, configuração segura e dependências vulneráveis. Não vejo OWASP como uma lista para decorar; uso como referência para pensar nos riscos da aplicação.”

---

### MAPA RÁPIDO: OWASP (ô-uasp)

| Conceito                  | Palavra-chave                     |
| ------------------------- | --------------------------------- |
| SQL Injection             | Query parametrizada / ORM         |
| XSS                       | Output encoding / sanitização     |
| Broken Access Control     | Permissão / ownership             |
| Authentication            | Identidade                        |
| Authorization             | Permissão                         |
| CSRF                      | Cookie / CSRF token               |
| SSRF                      | Requisição feita pelo servidor    |
| Cryptographic Failures    | Proteção de dados                 |
| Security Misconfiguration | Configuração segura               |
| Vulnerable Components     | Dependências                      |
| Security Logging          | Auditoria / monitoramento         |
| Input Validation          | Validar entrada                   |
| Secrets                   | Não colocar credenciais no código |
| Least Privilege           | Menor permissão necessária        |

### Notas complementares (XSS, SQL Injection, CSRF, LGPD)

#### XSS
Não é resolvido simplesmente com regex.
Defesas:
- validação
- sanitização quando HTML é permitido
- escaping/encoding de saída=- **O dado no banco/API:** `<script>` **O dado na saída (o que o React envia para o HTML do navegador):** `&lt;script&gt;`
- framework escaping = **HTML Escaping** + **SQL Escaping**
- CSP ==> Content-Security-Policy: default-src 'self'; script-src 'self' https://meudominio.com

#### CSRF
O problema é um navegador autenticado executar ação indevida.
Defesas:
- CSRF token
- SameSite cookies
- configuração adequada de cookies
- expiração reduz janela de exposição, mas não substitui proteção CSRF.

#### LGPD
Revisar:
- minimização de dados
- finalidade
- acesso restrito
- proteção de dados pessoais
- retenção
- auditoria quando aplicável.

Caso real de análise de autenticação e autorização: ver Guia Python.

---

## Parte 10. Inteligência Artificial

### INTELIGÊNCIA ARTIFICIAL: CONCEITOS PARA ENTREVISTA

#### Inteligência Artificial

IA é um campo amplo que busca criar sistemas capazes de executar tarefas que normalmente exigiriam algum tipo de inteligência humana, como reconhecer padrões, interpretar linguagem, tomar decisões ou gerar conteúdo.

Para desenvolvimento de sistemas, o mais importante é entender como integrar IA com aplicações reais.

---

#### Machine Learning

Machine Learning é uma área da IA em que os sistemas aprendem padrões a partir de dados.

Principais tipos:

- **Supervisionado**, aprende usando exemplos com respostas conhecidas.
    
- **Não supervisionado**, procura padrões ou agrupamentos nos próprios dados.
    
- **Reforço**, aprende por tentativa, erro e recompensas.
    

---

#### Deep Learning

Deep Learning é uma abordagem de Machine Learning baseada em redes neurais profundas.

A ideia é que a própria rede consiga aprender representações e padrões a partir dos dados, ajustando seus parâmetros durante o treinamento.

---

#### Rede Neural

Uma rede neural possui várias camadas de unidades matemáticas conectadas.

Durante o treinamento, os pesos dessas conexões são ajustados para que a rede aprenda padrões dos dados.

De forma simplificada:

```text
Entrada
   ↓
Camadas da rede
   ↓
Processamento
   ↓
Saída
```

---

#### LLM: Large Language Model

LLM é um modelo de linguagem treinado em grandes volumes de texto para aprender padrões da linguagem e gerar texto a partir do contexto recebido.

Uma forma simples de explicar:

> "É um modelo que aprendeu padrões da linguagem e consegue gerar uma continuação coerente a partir do contexto que recebe."

O modelo não funciona simplesmente como um banco de dados que procura uma resposta pronta.

---

#### Tokens

Tokens são unidades utilizadas pelo modelo para representar o texto.

Um token pode representar:

- uma palavra;
    
- parte de uma palavra;
    
- pontuação;
    
- outros fragmentos de texto.
    

Tokens são importantes porque afetam:

- limite de contexto;
    
- custo;
    
- quantidade de entrada;
    
- quantidade de saída;
    
- desempenho.
    

---

#### Treinamento x Inferência

##### Treinamento

É o processo em que o modelo aprende e ajusta seus parâmetros a partir dos dados.

##### Inferência

É quando utilizamos o modelo já treinado para produzir uma resposta a partir de uma entrada.

Forma simples:

> "Treinamento é quando o modelo aprende. Inferência é quando usamos o modelo."

---

#### Parâmetros

Parâmetros são valores internos ajustados durante o treinamento para que o modelo consiga representar padrões aprendidos.

Modelos modernos podem possuir milhões ou bilhões de parâmetros.

De forma simplificada:

```text
Dados
  ↓
Treinamento
  ↓
Ajuste dos parâmetros
  ↓
Modelo
  ↓
Inferência
```

---

#### Transformer

Transformers são uma arquitetura de redes neurais muito utilizada nos modelos modernos de linguagem.

Antes deles, arquiteturas como RNN processavam sequências de forma mais sequencial.

O Transformer introduziu o mecanismo de **attention**, permitindo que o modelo avalie a relação entre diferentes partes da sequência e determine quais informações são mais relevantes para aquele contexto.

---

#### Attention

Attention permite que o modelo dê diferentes pesos às partes da entrada de acordo com sua relevância para produzir a saída.

Uma explicação simples:

> "O modelo consegue olhar para diferentes partes do contexto e determinar quais são mais importantes para o que ele está tentando gerar."

---

### EMBEDDINGS E BANCO VETORIAL

#### Embedding

Embedding é uma representação numérica de uma informação.

Por exemplo:

```text
"Notebook Acer Aspire"
          ↓
       Embedding
          ↓
[0.12, -0.45, 0.87, ...]
```

Textos semanticamente parecidos tendem a possuir vetores próximos em um espaço vetorial.

---

#### Banco Vetorial

Um banco vetorial é especializado em armazenar embeddings e realizar buscas por similaridade de forma muito eficiente.

Importante:

> O banco vetorial normalmente não é responsável por transformar texto em vetor. Quem faz isso é o modelo de embedding.

Fluxo:

```text
Texto
  ↓
Modelo de Embedding
  ↓
Vetor
  ↓
Banco Vetorial
```

Na consulta:

```text
Pergunta
   ↓
Embedding
   ↓
Busca por similaridade
   ↓
Vetores/documentos mais próximos
```

---

### RAG

#### RAG: Retrieval-Augmented Generation

RAG combina busca de informação com geração de resposta.

Em vez de depender somente do conhecimento aprendido durante o treinamento do modelo, o sistema primeiro busca informações relevantes e coloca esse contexto junto da pergunta.

Fluxo:

```text
Pergunta do usuário
        ↓
    Embedding
        ↓
Busca no banco vetorial
        ↓
Documentos relevantes
        ↓
Contexto + pergunta
        ↓
       LLM
        ↓
     Resposta
```

O RAG ajuda a:

- utilizar informações externas;
    
- consultar documentos internos;
    
- trabalhar com informações atualizadas;
    
- reduzir o risco de alucinação;
    
- responder com base em uma fonte de contexto.
    

Importante:

> RAG não elimina completamente as alucinações. Ele reduz o risco ao fornecer contexto relevante para o modelo.

Uma boa resposta de entrevista:

> "No RAG eu busco a informação antes de gerar a resposta. Essa informação recuperada entra como contexto junto com a pergunta e o modelo consegue responder baseado não apenas no que aprendeu durante o treinamento, mas também nos dados que foram recuperados."

---

### AGENTES

#### Agente

Um agente utiliza um modelo para interpretar uma tarefa, tomar decisões e utilizar ferramentas para executar ações.

Um fluxo simplificado:

```text
Usuário
   ↓
Agente
   ↓
Decisão
   ↓
Tool
   ↓
Sistema externo
   ↓
Resultado
   ↓
Agente
   ↓
Resposta
```

Um agente pode, por exemplo:

```text
Consultar cliente
Consultar pedido
Consultar estoque
Criar chamado
Enviar e-mail
```

---

#### Tools

Tools são capacidades que o agente pode utilizar para executar ações ou consultar informações.

Exemplo:

```text
consultar_cliente()
consultar_pedido()
consultar_estoque()
criar_ticket()
```

A ideia importante é:

> "O modelo decide que precisa utilizar uma ferramenta, mas a execução real acontece através de uma função ou sistema controlado pela aplicação."

Isso é importante para segurança.

---

#### Skills

O conceito de Skill pode variar de acordo com a plataforma ou framework.

De forma geral, pode representar uma capacidade mais estruturada que o agente sabe executar.

Exemplo:

```text
Skill: atendimento ao cliente

1. identificar cliente
2. consultar pedido
3. verificar status
4. responder cliente
```

Não tratar "Skill" como um padrão universal. O significado depende da tecnologia utilizada.

---

### MEMÓRIA

#### Memória de Agentes

Um sistema de IA pode armazenar informações relevantes de uma interação para recuperá-las posteriormente.

Por exemplo:

```text
Conversa
   ↓
Informação importante
   ↓
Memória
   ↓
Banco
   ↓
Nova conversa
   ↓
Recuperação da memória
   ↓
Contexto do agente
```

Isso permite que o sistema mantenha informações entre diferentes interações.

A memória pode ser implementada utilizando diferentes estratégias e bancos, dependendo da necessidade.

---

### ORQUESTRAÇÃO E MULTIAGENTES

#### Orquestrador

Um orquestrador coordena diferentes agentes ou etapas de uma tarefa.

Por exemplo:

```text
                 ORQUESTRADOR
                /      |       \
               ↓       ↓        ↓
           Agente   Agente   Agente
           Vendas   Financeiro Suporte
               \       |       /
                \      |      /
                 Resultado
```

Uma tarefa complexa pode ser dividida em partes menores e distribuída entre agentes especializados.

---

#### Multiagentes

Arquitetura multiagente utiliza vários agentes especializados trabalhando em conjunto.

Exemplo:

```text
Agente Coordenador
       ↓
 ┌─────┼─────┐
 ↓     ↓     ↓
Venda Financeiro Suporte
```

Cada agente pode possuir ferramentas, responsabilidades e contexto diferentes.

---

#### Agente Autônomo

Um agente autônomo consegue executar uma tarefa com pouca intervenção humana.

Ele pode:

1. receber um objetivo;
    
2. analisar o problema;
    
3. decidir os próximos passos;
    
4. utilizar ferramentas;
    
5. analisar os resultados;
    
6. continuar até atingir o objetivo ou encontrar uma condição de parada.
    

---

### FINE-TUNING E RLHF

#### Fine-Tuning

Fine-tuning é utilizar um modelo previamente treinado e realizar um treinamento adicional para especializá-lo em determinado comportamento, tarefa ou domínio.

De forma simples:

```text
Modelo pré-treinado
        ↓
Treinamento adicional
        ↓
Modelo especializado
```

Não significa necessariamente criar um modelo do zero.

---

#### RLHF

RLHF significa:

**Reinforcement Learning from Human Feedback**

É uma técnica/processo em que feedback humano é utilizado para ajudar a alinhar o comportamento do modelo.

De forma simplificada:

```text
Modelo gera respostas
        ↓
Humanos avaliam
        ↓
Feedback
        ↓
Processo de treinamento/alinhamento
        ↓
Modelo
```

---

### GERAÇÃO DE TEXTO

#### Temperature

Temperature influencia a aleatoriedade da geração.

De forma simplificada:

- temperatura mais baixa → respostas mais determinísticas;
    
- temperatura mais alta → maior variedade na geração.
    

Não significa literalmente "seguir regras" ou "ser criativo", mas é uma boa forma intuitiva de entender o efeito.

---

#### Top-K

Top-K limita a geração aos K tokens mais prováveis.

Exemplo:

```text
Top-K = 5
```

O modelo considera os 5 candidatos mais prováveis naquele momento.

---

#### Top-P

Top-P utiliza uma probabilidade acumulada para determinar quais tokens podem participar da geração.

De forma simples:

> Top-K limita pela quantidade de candidatos. Top-P limita pela probabilidade acumulada.

---

### PROMPT ENGINEERING

#### Engenharia de Prompt

Prompt Engineering é a prática de estruturar instruções, contexto, exemplos e restrições para orientar o comportamento do modelo.

Não é apenas "escrever um prompt bonito".

Também envolve:

- contexto;
    
- instruções;
    
- formato esperado;
    
- restrições;
    
- exemplos;
    
- ferramentas;
    
- segurança;
    
- dados fornecidos ao modelo.
    

Hoje, uma aplicação de IA bem projetada depende muito mais da arquitetura ao redor do modelo do que de encontrar simplesmente um "prompt mágico".

---

### IA GENERATIVA

#### IA Descritiva x IA Generativa

##### IA Descritiva

Analisa dados existentes e descreve o que aconteceu.

Exemplo:

> "As vendas caíram 15% no último mês."

##### IA Generativa

Gera conteúdo novo a partir dos dados e instruções recebidas.

Exemplo:

> "Gere um relatório explicando possíveis causas da queda nas vendas."

---

### TREINAMENTO DE MODELOS

#### Criar um modelo x utilizar um modelo

Existem níveis diferentes de trabalho com IA.

##### Utilizar um modelo

```text
Aplicação
   ↓
API
   ↓
LLM
```

##### RAG

```text
Aplicação
   ↓
Busca
   ↓
Contexto
   ↓
LLM
```

##### Fine-tuning

```text
Modelo existente
       ↓
Treinamento adicional
       ↓
Modelo especializado
```

##### Treinar um modelo

É um processo muito mais complexo, envolvendo dados, arquitetura, treinamento, infraestrutura, parâmetros e grande capacidade computacional.

Para uma vaga de backend, normalmente é muito mais importante saber **integrar e operar modelos** do que saber treinar um LLM do zero.

---

### IA EM SISTEMAS REAIS

#### Como eu penso em IA dentro de uma aplicação

Eu não vejo IA apenas como colocar um ChatGPT dentro do sistema.

Eu penso na arquitetura:

```text
                  APLICAÇÃO
                      ↓
                    API
                      ↓
              ┌───────┴───────┐
              ↓               ↓
             RAG            AGENTE
              ↓               ↓
        Banco Vetorial      Tools
                              ↓
                     Sistemas externos
```

E junto disso preciso pensar em:

- autenticação;
    
- autorização;
    
- segurança;
    
- dados sensíveis;
    
- LGPD;
    
- observabilidade;
    
- custo;
    
- latência;
    
- limites;
    
- auditoria;
    
- controle das ferramentas;
    
- tratamento de erros.
    

---

### SEGURANÇA EM IA

#### Prompt Injection

Prompt Injection acontece quando uma entrada tenta manipular as instruções do modelo.

Por exemplo, um usuário pode tentar fazer o modelo ignorar as regras originais e executar uma ação que não deveria.

Por isso, não devemos confiar apenas no modelo para segurança.

As permissões precisam estar na aplicação.

---

#### Controle de acesso

Um agente não deve conseguir executar qualquer ferramenta apenas porque o modelo pediu.

Exemplo:

```text
LLM
 ↓
"quero excluir cliente"
 ↓
API
 ↓
Verifica autorização
 ↓
Executa ou bloqueia
```

A decisão de segurança deve estar no backend.

---

#### Dados sensíveis

Ao trabalhar com IA é importante avaliar:

- informações pessoais;
    
- credenciais;
    
- dados financeiros;
    
- informações internas;
    
- LGPD;
    
- armazenamento;
    
- logs;
    
- quem pode acessar o contexto.
    

---

### RESPOSTA PARA ENTREVISTA

Se perguntarem:

**"Qual sua experiência com Inteligência Artificial?"**

Uma resposta natural seria:

> "Tenho estudado bastante IA pelo lado de engenharia e aplicação em sistemas. Entendo embeddings e banco vetorial, RAG, agentes, tools, memória, orquestração e arquitetura multiagente. Também conheço os conceitos de treinamento, inferência, fine-tuning, RLHF, tokens, LLM, Transformer, temperature, Top-K e Top-P.
> 
> O meu foco é mais na integração do que em treinamento de modelo do zero. Por exemplo, num RAG eu consigo entender o fluxo de transformar a pergunta em embedding, buscar informações relevantes no banco vetorial e colocar esse contexto junto da pergunta antes de mandar para o modelo.
> 
> E quando penso em colocar IA numa aplicação real, não penso só na resposta do modelo. Penso também em API, autenticação, autorização, segurança, controle das tools, memória, observabilidade, custo e integração com os sistemas existentes."

---

### RESUMO PARA DECORAR

```text
IA
│
├── Machine Learning
│   ├── Supervisionado
│   ├── Não supervisionado
│   └── Reforço
│
├── Deep Learning
│   └── Redes Neurais
│
├── LLM
│   ├── Tokens
│   ├── Parâmetros
│   └── Transformer
│       └── Attention
│
├── Embeddings
│   └── Banco Vetorial
│
├── RAG
│   └── Busca + Contexto + LLM
│
├── Agentes
│   ├── Tools
│   ├── Skills
│   ├── Memória
│   └── Autonomia
│
├── Multiagentes
│   └── Orquestrador
│
├── Fine-Tuning
│
├── RLHF
│
├── Inferência
│
└── IA Generativa
```

#### REGRA PRINCIPAL PARA A ENTREVISTA

Não tente demonstrar que sabe tudo de IA.

Se perguntarem algo que você conhece:

> explique com um exemplo.

Se perguntarem algo que você conhece superficialmente:

> diga até onde você conhece.

E principalmente:

> "Minha experiência está mais na aplicação e integração de IA em sistemas do que na pesquisa ou treinamento de modelos."

Isso é totalmente coerente com uma posição de **referência técnica/backend**.

---

## Parte 11. Histórias técnicas e documentação

### Histórias reais para usar

- Conflito Front x Back
- Dev júnior / alteração problemática
- Mentoria
- Code review
- Incidente
- Arquitetura na FITEC
- Projeto de logística como freelancer

Não focar em culpa. Focar em decisão, comunicação e resultado.

### Documentação

Evidência real: documentação de API com OpenAPI/Swagger (projeto Node + MongoDB). Complementar com README de setup, diagramas de arquitetura e registro de decisões (ADR).

---

## Parte 12. Perguntas para fazer ao gestor

1. Qual é o projeto e em que fase ele está?

2. Como é o time hoje e onde essa posição se encaixa?

3. Qual é hoje o maior desafio técnico que vocês enfrentam?

4. Como funciona o processo de desenvolvimento? Vocês trabalham com code review, testes automatizados e CI/CD?

5. O que seria considerado um bom resultado para essa posição nos primeiros três meses?

6. Existe espaço para o desenvolvedor participar das decisões de arquitetura?

---

## Parte 13. Checklist antes da entrevista

### Banco

- [ ] SQL nativo
- [ ] Índices
- [ ] Transações
- [ ] N+1
- [ ] MongoDB

### Arquitetura

- [ ] Clean Architecture
- [ ] SOLID
- [ ] DDD básico
- [ ] Repository
- [ ] Use Case
- [ ] Dependency Injection

### Testes

- [ ] Pirâmide
- [ ] Unitários
- [ ] Integração
- [ ] E2E
- [ ] Mock
- [ ] TDD

### DevOps

- [ ] Docker
- [ ] Git
- [ ] GitFlow
- [ ] Code Review
- [ ] CI/CD
- [ ] AWS

### Resiliência

- [ ] Timeout
- [ ] Retry
- [ ] Backoff
- [ ] Jitter
- [ ] Circuit Breaker
- [ ] Idempotência
- [ ] DLQ

### Observabilidade

- [ ] Logs
- [ ] Métricas
- [ ] Traces
- [ ] Alertas
- [ ] Health Check

### Mapa de revisão

**Banco**  
→ ORM × SQL nativo, índices, transações, N+1, SQL Injection

**Clean Architecture**  
→ Entity → Use Case → Interface → Adapter

**Testes**  
→ unitário → integração → E2E → mocks → TDD

**Resiliência**  
→ timeout → retry → backoff → jitter → circuit breaker → idempotência

**Mensageria**  
→ producer → consumer → fila → retry → DLQ

**Docker / CI/CD**  
→ build → testes → imagem → deploy → rollback

**Observabilidade**  
→ logs → métricas → traces → health check → latência → erros

**Git**  
→ PR → code review → GitFlow → trunk-based
