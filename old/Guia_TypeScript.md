# Guia TypeScript / Node.js

Específico da stack Node/TypeScript e da vaga Node.js / TypeScript. Os conceitos gerais estão no Guia de Conceitos Universais.

---

## Apresentação para o gestor

> Minha trajetória é em desenvolvimento web, com foco forte em backend e arquitetura. Comecei como júnior e fui assumindo responsabilidades maiores, passando por Python, Go, C# e TypeScript, sempre mantendo os mesmos princípios de arquitetura e qualidade de código.
>
> Na Itriad, trabalhei no Leakage, um sistema em produção feito em TypeScript, com Express no backend e React no front, aplicando Clean Architecture. Atuei no backend, principalmente no módulo de notificações e no controle de acesso. Também tive uma experiência mais breve com Angular, em um projeto com backend em C#.
>
> Na FITEC, assumi parte da responsabilidade de arquitetura e AWS quando o arquiteto do projeto saiu. E como freelancer, conduzi um projeto de logística de ponta a ponta, da arquitetura à infraestrutura.
>
> Hoje busco um lugar onde eu possa contribuir tecnicamente, participar das decisões de arquitetura e ajudar o time a evoluir.

### Se perguntarem sobre os dois anos fora do mercado formal

> Fiquei aproximadamente dois anos fora do mercado formal, mas não fiquei parado tecnicamente. Continuei estudando e desenvolvendo projetos, principalmente em Go, arquitetura, AWS, CI/CD, testes e sistemas distribuídos. Então vejo esse momento mais como uma retomada do mercado formal do que como uma pausa na minha evolução técnica.

## Mapa: requisito da vaga x evidência

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

## História principal: Leakage (Itriad)

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

- **TypeORM**, ORM tradicional, decorators, Entity e Repository.
- **Prisma**, schema + Prisma Client, muito forte em TypeScript.
- **Sequelize**, ORM tradicional do Node.js.
- **MikroORM**, ORM TypeScript.
- **Knex.js**, query builder, mais próximo do SQL.
- **Drizzle ORM**, solução moderna, TypeScript-first.

Não afirmar experiência prática com uma ferramenta que não lembra de ter usado.

**Nota de preparação:** Revise cada ponto com o que você realmente lembra. Se o gestor perguntar "como você implementou a leitura por setor?", tenha a resposta da implementação real. Confirme se o banco era MySQL; se não tiver certeza, diga "banco relacional, MySQL se não me engano".

**Clean Architecture no Leakage:** Prepare como era a estrutura de pastas no Leakage, para descrever com segurança.

No contexto Node, o Use Case não precisa saber se o banco usa TypeORM, Prisma, Sequelize ou SQL puro.

---

## Angular

#### Resposta-base

> Na Itriad tive uma experiência mais breve com Angular, em um projeto com backend em C#, e também tenho um projeto próprio em Angular 13 com TypeScript, componentes, módulos e rotas. Meu frontend mais extenso foi com React, mas os fundamentos são os mesmos, e o modelo do Angular, com injeção de dependência e estrutura opinativa, conversa bem com a forma como organizo o backend.

#### Revisar

- Services com `HttpClient`
- Observables / RxJS
- Formulários reativos
- Guards de rota
- Interceptors
- JWT

Tenha um exemplo concreto do que fez no Angular na Itriad, mesmo simples (uma tela, um formulário, uma integração com a API).

---

## Testes no Leakage e TDD

### Experiência no Leakage

> Nos meus projetos sempre procurei seguir a pirâmide de testes, priorizando testes unitários, depois testes de integração e uma quantidade menor de testes E2E. No Leakage, os testes unitários ajudavam a garantir que os objetos fossem criados respeitando as regras de negócio. Então, se durante o desenvolvimento alguém alterasse o comportamento esperado de uma classe ou de seus atributos, a suíte de testes conseguia identificar essa regressão.

### Se perguntarem: "Isso é TDD?"

> A pirâmide de testes é uma estratégia de distribuição dos testes. TDD é uma abordagem de desenvolvimento em que escrevemos o teste antes, fazemos ele falhar, implementamos o mínimo para passar e depois refatoramos. Eu conheço o processo e seus benefícios, mas não afirmaria que todo o desenvolvimento do Leakage foi feito formalmente seguindo TDD.

Atenção: TDD é requisito da vaga e testes não apareceram no Leakage. Não afirme TDD se não praticava.

### Jest

No ecossistema Node/TypeScript:

- Jest
- mocks
- spies
- assertions
- testes unitários
- integração

---

## Node.js + TypeScript

### Conceitos para lembrar

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

#### Async/Await

```ts
const user = await userService.findById(id);
```

#### Promise

```ts
const result = await Promise.all([
  serviceA.execute(),
  serviceB.execute()
]);
```

### Como estruturaria uma API Node/TypeScript?

> Eu separaria responsabilidades entre controller, caso de uso, domínio e infraestrutura. O controller trata HTTP, o caso de uso concentra a regra de negócio e os adapters fazem a comunicação com banco e serviços externos.

Resumo: Controller → Use Case → Repository → Infrastructure.

---

## Banco, CI e escalabilidade no Leakage

Leakage:

- MySQL
- ORM
- queries nativas

API:

- MongoDB

No Leakage havia Gitflow e pipeline no GitHub.

- **CI:** "O fluxo era Gitflow, com pipeline no GitHub (GitHub Actions) rodando build e deploy a cada merge." Se não tiver certeza de que era Actions, diga apenas "pipeline no GitHub".

- **Gancho de escalabilidade:** "envio de e-mail é um bom candidato a processamento assíncrono, para não travar a requisição." Só diga que fizeram assim se foi o caso; senão, use como "o que eu faria".

---

## Projeto Eldorado: "o que eu faria diferente hoje"

Se o gestor abrir o GitHub ou perguntar sobre projetos pessoais:

- segredo do JWT em variável de ambiente, nunca fixo no código;
- aplicar os validadores nas rotas;
- separar casos de uso dos controllers (Clean Architecture);
- testes, Docker e pipeline;
- tratamento de erro centralizado;
- completar a integração do front Angular com a API.

- Remover o `.env` do repositório público hoje

---

## Checklist Node/TypeScript

- [ ] Async/Await
- [ ] Promises
- [ ] Event Loop
- [ ] Express
- [ ] REST
- [ ] Middleware
- [ ] TypeScript

- [ ] TypeORM
- [ ] Prisma
- [ ] Sequelize
- [ ] Jest


