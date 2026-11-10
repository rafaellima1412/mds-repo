# Guia de Entrevista — Gestor da Área
## Vaga: Node.js / TypeScript

---

## 1. Apresentação para o gestor

> Minha trajetória é em desenvolvimento web, com foco forte em backend e arquitetura. Comecei como júnior e fui assumindo responsabilidades maiores, passando por Python, Go, C# e TypeScript, sempre mantendo os mesmos princípios de arquitetura e qualidade de código.
>
> Na Itriad, trabalhei no Leakage, um sistema em produção feito em TypeScript, com Express no backend e React no front, aplicando Clean Architecture. Atuei no backend, principalmente no módulo de notificações e no controle de acesso. Também tive uma experiência mais breve com Angular, em um projeto com backend em C#.
>
> Na FITEC, assumi parte da responsabilidade de arquitetura e AWS quando o arquiteto do projeto saiu. E como freelancer, conduzi um projeto de logística de ponta a ponta, da arquitetura à infraestrutura.
>
> Hoje busco um lugar onde eu possa contribuir tecnicamente, participar das decisões de arquitetura e ajudar o time a evoluir.

### Se perguntarem sobre os dois anos fora do mercado formal

> Fiquei aproximadamente dois anos fora do mercado formal, mas não fiquei parado tecnicamente. Continuei estudando e desenvolvendo projetos, principalmente em Go, arquitetura, AWS, CI/CD, testes e sistemas distribuídos. Então vejo esse momento mais como uma retomada do mercado formal do que como uma pausa na minha evolução técnica.

---

## 2. Mapa: requisito da vaga x evidência

| Requisito            | Evidência                                                                               |
| -------------------- | --------------------------------------------------------------------------------------- |
| Node.js + TypeScript | Leakage (produção), projeto Eldorado                                                    |
| Angular              | Itriad (experiência breve, com backend C#), projeto Eldorado (Angular 13)               |
| Docker, Git          | Leakage (Docker, Gitflow)                                                               |
| SQL e NoSQL          | Leakage (MySQL, typeORM + query nativa), API com MongoDB                                |
| Clean Architecture   | Leakage                                                                                 |
| Ágil, CI             | Leakage (Gitflow + pipeline no GitHub)                                                  |
| TDD                  | Experiência com pirâmide de testes; conhecer diferença entre TDD e estratégia de testes |
| Escalabilidade       | FITEC / AWS                                                                             |
| Requisitos e cliente | Freelance de logística                                                                  |
| Code review          | Experiência de liderança técnica                                                        |
| Python, React        | Experiência anterior, Leakage                                                           |

---

## 3. História principal: Leakage (Itriad)

### Contexto

Sistema para catalogar peças com vazamento e notificar os setores internos e os fornecedores responsáveis.

### Seu papel

Backend, com foco em notificações e controle de acesso.

### Pontos importantes

- **Controle de acesso em dois níveis:** quais partes do sistema o usuário acessa (funcionalidades) e quais dados ele pode ver (escopo). Um fornecedor só via as peças do seu cliente.
- **Notificações em cima das permissões:** cada usuário recebia apenas o que era do seu escopo.
- **Leitura compartilhada por setor:** quando alguém do setor lia a notificação, ela ficava como lida para todo o setor. Evita retrabalho e duas pessoas tratando o mesmo vazamento.
- **ORM + queries nativas:** ORM no dia a dia; query nativa onde precisava de mais controle ou performance.
- **Resultado:** sistema em produção, com Docker, Gitflow e pipeline no GitHub.

### Banco / ORM

A lembrança é de **TypeORM + query nativa**.

Para recordar:

#### TypeORM

```ts
@Entity()
class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;
}
```

Uso típico:

```ts
const users = await userRepository.find({
  where: { active: true }
});

const user = userRepository.create({
  name: "Rafael"
});

await userRepository.save(user);
```

**Palavras-chave:** `@Entity`, `@Column`, `@PrimaryGeneratedColumn`, `Repository`.

#### Sequelize

Outro ORM comum no ecossistema Node:

```ts
const users = await User.findAll({
  where: { active: true }
});

const user = await User.create({
  name: "Rafael"
});
```

**Palavras-chave:** `Model.findAll()`, `findOne()`, `create()`.

### Outros nomes para reconhecer em entrevista

- **TypeORM** — ORM tradicional, decorators, Entity e Repository.
- **Prisma** — schema + Prisma Client, muito forte em TypeScript.
- **Sequelize** — ORM tradicional do Node.js.
- **MikroORM** — ORM TypeScript.
- **Knex.js** — query builder, mais próximo do SQL.
- **Drizzle ORM** — solução moderna, TypeScript-first.

Não afirmar experiência prática com uma ferramenta que não lembra de ter usado.

---

# 4. Clean Architecture

### Explicação curta

> A ideia é proteger a regra de negócio. O centro tem as entidades e os casos de uso, que não conhecem Express, ORM nem banco. Em volta ficam os adaptadores, como controllers e implementações de repositório. A dependência sempre aponta para dentro.

### Exemplo mental

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

O Use Case não precisa saber se o banco usa:

- TypeORM
- Prisma
- Sequelize
- SQL puro

### Ligação com testes

No teste:

```text
CriarUsuario
     ↓
Mock UserRepository
```

Assim é possível testar a regra de negócio sem banco, ORM ou HTTP.

### Frase para entrevista

> A Clean Architecture facilita os testes porque consigo testar o caso de uso isoladamente, substituindo infraestrutura como banco e ORM por mocks.

### Trade-off

> Gera mais arquivos e abstrações. Para um CRUD simples pode ser exagero; vale quando a regra de negócio é relevante.

---

# 5. Angular

### Resposta-base

> Na Itriad tive uma experiência mais breve com Angular, em um projeto com backend em C#, e também tenho um projeto próprio em Angular 13 com TypeScript, componentes, módulos e rotas. Meu frontend mais extenso foi com React, mas os fundamentos são os mesmos, e o modelo do Angular, com injeção de dependência e estrutura opinativa, conversa bem com a forma como organizo o backend.

### Revisar

- Services com `HttpClient`
- Observables / RxJS
- Formulários reativos
- Guards de rota
- Interceptors
- JWT

---

# 6. Testes e TDD

## Pirâmide de testes

A ideia utilizada nos meus projetos:

```text
             E2E
          /       \
       Integração
      /             \
    Unitários — muitos
```

- Muitos testes unitários
- Menos testes de integração
- Poucos testes E2E

## Experiência no Leakage

> Nos meus projetos sempre procurei seguir a pirâmide de testes, priorizando testes unitários, depois testes de integração e uma quantidade menor de testes E2E. No Leakage, os testes unitários ajudavam a garantir que os objetos fossem criados respeitando as regras de negócio. Então, se durante o desenvolvimento alguém alterasse o comportamento esperado de uma classe ou de seus atributos, a suíte de testes conseguia identificar essa regressão.

## Se perguntarem: "Isso é TDD?"

> A pirâmide de testes é uma estratégia de distribuição dos testes. TDD é uma abordagem de desenvolvimento em que escrevemos o teste antes, fazemos ele falhar, implementamos o mínimo para passar e depois refatoramos. Eu conheço o processo e seus benefícios, mas não afirmaria que todo o desenvolvimento do Leakage foi feito formalmente seguindo TDD.

## TDD — ciclo

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

## Ligação com Clean Architecture

Casos de uso isolados são mais fáceis de testar:

```text
Use Case
   ↓
Interface Repository
   ↓
Mock
```

O teste não precisa subir banco nem depender do ORM.

### Jest

No ecossistema Node/TypeScript:

- Jest
- mocks
- spies
- assertions
- testes unitários
- integração

---

# 7. Node.js + TypeScript

## Conceitos para lembrar

- Event Loop
- Async/Await
- Promises
- Middleware
- REST
- HTTP status codes
- Tratamento de erros
- Validação
- Dependency Injection
- JWT
- Environment variables
- Logging
- Graceful shutdown

### Async/Await

```ts
const user = await userService.findById(id);
```

### Promise

```ts
const result = await Promise.all([
  serviceA.execute(),
  serviceB.execute()
]);
```

---

# 8. API / REST

Perguntas possíveis:

### Como estruturaria uma API?

> Eu separaria responsabilidades entre controller, caso de uso, domínio e infraestrutura. O controller trata HTTP, o caso de uso concentra a regra de negócio e os adapters fazem a comunicação com banco e serviços externos.

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

---

# 9. Banco de dados

## Relacional

Leakage:

- MySQL
- ORM
- queries nativas

API:

- MongoDB

## Pontos para revisar

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

### ORM x SQL nativo

> Uso ORM para operações comuns e produtividade. Quando tenho uma consulta muito específica, complexa ou que exige maior controle de performance, posso utilizar SQL nativo.

---

# 10. Docker

### Resposta curta

> Uso Docker para padronizar o ambiente e garantir que aplicação e dependências tenham comportamento consistente entre desenvolvimento, CI e produção.

### Conceitos

- Dockerfile
- Image
- Container
- Volume
- Network
- Docker Compose
- Multi-stage build

---

# 11. Git / GitFlow / Code Review

## GitFlow

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

## Code Review

> Para mim, code review não é apenas procurar erro de sintaxe. É uma oportunidade de verificar regra de negócio, arquitetura, segurança, testes, legibilidade e manutenção futura.

---

# 12. CI/CD

Fluxo possível:

```text
Git
 ↓
Pull Request
 ↓
Code Review
 ↓
Build
 ↓
Testes
 ↓
Docker Image
 ↓
Deploy
 ↓
Monitoramento
```

No Leakage havia Gitflow e pipeline no GitHub.

---

# 13. Escalabilidade e disponibilidade

Conceitos:

- API stateless
- Escalabilidade horizontal
- Cache
- Índices
- Filas
- Processamento assíncrono
- Observabilidade

### Exemplo

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

# 14. Resiliência

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

### Retry

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

### Idempotência

A mesma mensagem pode chegar mais de uma vez.

> Uso um identificador único do evento e uma garantia de processamento único, normalmente apoiada por constraint ou controle transacional.

---

# 15. Filas e eventos

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

### DLQ

Mensagens que continuam falhando depois das tentativas podem ser direcionadas para uma Dead Letter Queue para investigação.

---

# 16. Observabilidade

Revisar:

- Logs
- Métricas
- Traces
- Alertas
- Health checks
- Disponibilidade
- Latência
- Taxa de erro
- Capacidade

Frase boa:

> Não basta o sistema funcionar; precisamos conseguir perceber quando ele está degradando.

---

# 17. AWS

Usar principalmente a experiência da FITEC.

Conceitos que posso mencionar:

- API Gateway
- Lambda
- ECS
- ECR
- RDS
- S3
- CloudWatch
- SQS
- SNS
- Secrets Manager
- Parameter Store
- CodeBuild
- CodePipeline

Não transformar a resposta em lista de serviços. Explicar o problema que cada serviço resolveu.

---

# 18. DDD

Revisar apenas o essencial:

- Domínio
- Entidades
- Value Objects
- Agregados
- Serviços de domínio
- Bounded Contexts

Não decorar termos sem entender o problema que resolvem.

---

# 19. Perguntas técnicas prováveis

### Como estrutura uma API Node/TypeScript?

Controller → Use Case → Repository → Infrastructure.

### Por que separar Use Case e Repository?

Para separar regra de negócio de infraestrutura e facilitar testes e manutenção.

### Como testa regra de negócio?

Com testes unitários, isolando o caso de uso e substituindo dependências externas por mocks.

### Como faria integração com serviço externo?

Timeout, tratamento de erros, retry quando aplicável, backoff, circuit breaker e idempotência quando necessário.

### Como faria um webhook resiliente?

Receber → validar → persistir/publicar → responder rapidamente → processar de forma assíncrona.

### Como garante autorização?

Autenticação identifica o usuário. Autorização verifica se ele pode executar aquela operação e acessar aquele recurso.

### Como evita SQL Injection?

Queries parametrizadas / [^1]prepared statements e nunca concatenar entrada do usuário diretamente no SQL.

### Como funciona seu CI/CD?

Build → testes → criação da imagem → publicação → deploy → monitoramento.

### Como faria rollback?

Manter uma versão anterior conhecida e permitir voltar rapidamente para ela caso o deploy apresente problema.

### Como investiga um incidente?

Logs → métricas → traces → identificar impacto → mitigar → corrigir → analisar causa raiz → evitar recorrência.

---

# 20. Perguntas comportamentais

Usar:

```text
Contexto
   ↓
Problema
   ↓
Ação
   ↓
Resultado
   ↓
Aprendizado
```

### Histórias reais para usar

- Conflito Front x Back
- Dev júnior / alteração problemática
- Mentoria
- Code review
- Incidente
- Arquitetura na FITEC
- Projeto de logística como freelancer

Não focar em culpa. Focar em decisão, comunicação e resultado.

---

# 21. Perguntas para fazer ao gestor

**3.**“Como funciona o processo de desenvolvimento? Vocês trabalham com code review, testes automatizados e CI/CD?”

**4.**“O que seria considerado um bom resultado para essa posição nos primeiros três meses?”

### Projeto

> Qual é o projeto e em que fase ele está?

### Desafio

> Qual é hoje o maior desafio técnico que vocês enfrentam?

### Processo

> Como funciona o processo de desenvolvimento? Vocês trabalham com code review, tes[^2]tes automatizados e CI/CD?

### Primeiros meses

> O que seria considerado um bom resultado para essa posição nos primeiros três meses?

### Arquitetura

> Existe espaço para o desenvolvedor participar das decisões de arquitetura?

---

# 22. Como responder depois de dois anos fora

Não pedir desculpas.

Não dizer:

> "Estou enferrujado."

Preferir:

> Fiquei aproximadamente dois anos fora do mercado formal, mas continuei estudando e desenvolvendo projetos. Nesse período aprofundei bastante conhecimentos em Go, arquitetura, AWS, CI/CD, testes e sistemas distribuídos. Estou justamente buscando voltar ao mercado formal levando essa bagagem técnica para uma posição em que eu possa contribuir.

---

# 23. O que fazer quando não lembrar

Não inventar.

Frases úteis:

> "Não lembro exatamente qual implementação usamos naquela época, mas lembro do conceito."

> "Não tive experiência prática profunda nessa ferramenta, mas conheço o conceito e consigo explicar como eu abordaria."

> "Essa parte específica eu precisaria revisar, mas o princípio que eu seguiria seria..."

> "Não quero afirmar algo que não tenho certeza."

Isso passa mais segurança do que tentar adivinhar.

---

# 24. Regra de ouro

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

---

# 25. Checklist antes da entrevista

## TypeScript / Node

- [ ] Async/Await
- [ ] Promises
- [ ] Event Loop
- [ ] Express
- [ ] REST
- [ ] Middleware
- [ ] TypeScript

## Banco

- [ ] TypeORM
- [ ] Prisma
- [ ] Sequelize
- [ ] SQL nativo
- [ ] Índices
- [ ] Transações
- [ ] N+1
- [ ] MongoDB

## Arquitetura

- [ ] Clean Architecture
- [ ] SOLID
- [ ] DDD básico
- [ ] Repository
- [ ] Use Case
- [ ] Dependency Injection

## Testes

- [ ] Pirâmide
- [ ] Unitários
- [ ] Integração
- [ ] E2E
- [ ] Mock
- [ ] Jest
- [ ] TDD

## DevOps

- [ ] Docker
- [ ] Git
- [ ] GitFlow
- [ ] Code Review
- [ ] CI/CD
- [ ] AWS

## Resiliência

- [ ] Timeout
- [ ] Retry
- [ ] Backoff
- [ ] Jitter
- [ ] Circuit Breaker
- [ ] Idempotência
- [ ] DLQ

## Observabilidade

- [ ] Logs
- [ ] Métricas
- [ ] Traces
- [ ] Alertas
- [ ] Health Check

---

# 26. Frase final para lembrar

> "Eu procuro escolher a solução adequada ao contexto, com complexidade proporcional ao problema. Não necessariamente a solução mais sofisticada."

Essa ideia resume bem a postura esperada de alguém com experiência em arquitetura.



**Node/TypeScript**  
→ async/await, Promises, REST, middleware, erros

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

[^1]: Outra vantagem dos _Prepared Statements_ é que, se utilizados corretamente, ajudam a evitar ataques de Injeção de SQL. **Note que para isso** é preciso que os parâmetros da consulta sejam atribuídos através dos métodos `setInt()`, `setString()`, etc. presentes na interface `PreparedStatement` e não por concatenação de strings.

[^2]: - **Testes unitários**: Validam o menor pedaço de código isolado, como uma função ou método específico.
	- **Testes de integração**: Verificam se diferentes módulos ou sistemas conversam bem entre si.
	- **Testes ponta a ponta (E2E)**: Simulam a ação real de um usuário navegando por toda a aplicação
