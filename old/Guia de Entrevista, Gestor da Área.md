
## 1. Apresentação para o gestor

"Minha trajetória é em desenvolvimento web, com foco forte em backend e arquitetura. Comecei como júnior e fui assumindo responsabilidades maiores, passando por Python, Go, C# e TypeScript, sempre mantendo os mesmos princípios de arquitetura e qualidade de código.

Na Itriad, trabalhei no Leakage, um sistema em produção feito em TypeScript, com Express no backend e React no front, aplicando Clean Architecture. Atuei no backend, principalmente no módulo de notificações e no controle de acesso. Também tive uma experiência mais breve com Angular, em um projeto com backend em C#.

Na FITEC, assumi parte da responsabilidade de arquitetura e AWS quando o arquiteto do projeto saiu. E como freelancer, conduzi um projeto de logística de ponta a ponta, da arquitetura à infraestrutura.

Hoje busco um lugar onde eu possa contribuir tecnicamente, participar das decisões de arquitetura e ajudar o time a evoluir."

## 2. Mapa: requisito da vaga x evidência

|                                    |                                                                           |
| ---------------------------------- | ------------------------------------------------------------------------- |
| Requisito                          | Evidência                                                                 |
| Node.js + TypeScript               | Leakage (produção), projeto Eldorado                                      |
| Angular                            | Itriad (experiência breve, com backend C#), projeto Eldorado (Angular 13) |
| Docker, Git                        | Leakage (Docker, Gitflow)                                                 |
| SQL e NoSQL                        | Leakage (MySQL, ORM + query nativa), API com MongoDB                      |
| Clean Architecture                 | Leakage                                                                   |
| Ágil, CI                           | Leakage (Gitflow + pipeline no GitHub)                                    |
| TDD                                | Ponto de atenção (ver item 6)                                             |
| Escalabilidade (diferencial)       | FITEC / AWS                                                               |
| Requisitos e cliente (diferencial) | Freelance de logística                                                    |
| Code review (diferencial)          | Guia de RH, item 7                                                        |
| Python, React (diferencial)        | Experiência anterior, Leakage                                             |

## 3. História principal: Leakage (Itriad)

**Contexto:** sistema para catalogar peças com vazamento e notificar os setores internos e os fornecedores responsáveis.

**Seu papel:** backend, com foco em notificações e controle de acesso.

**O que vale explicar:**

- **Controle de acesso em dois níveis:** quais partes do sistema o usuário acessa (funcionalidades) e quais dados ele pode ver (escopo). Um fornecedor só via as peças do seu cliente.
- **Notificações em cima das permissões:** cada usuário recebia apenas o que era do seu escopo. E-mail como canal externo.
- **Leitura compartilhada por setor:** quando alguém do setor lia a notificação, ela ficava como lida para todo o setor. Evita retrabalho e duas pessoas tratando o mesmo vazamento. É uma regra de negócio; mostre que entende o motivo.
- **ORM e queries nativas:** ORM no dia a dia; query nativa onde precisava de mais controle ou performance (consultas complexas, relatórios).
- **Resultado:** sistema em produção, com Docker, Gitflow e pipeline no GitHub.

Revise cada ponto com o que você realmente lembra. Se o gestor perguntar "como você implementou a leitura por setor?", tenha a resposta da implementação real. Confirme se o banco era MySQL; se não tiver certeza, diga "banco relacional, MySQL se não me engano".

## 4. Clean Architecture

**Explicação curta:** "A ideia é proteger a regra de negócio. O centro tem as entidades e os casos de uso, que não conhecem Express, ORM nem banco. Em volta ficam os adaptadores, como controllers e implementações de repositório, e na borda os frameworks. A dependência sempre aponta para dentro: o caso de uso depende de uma interface de repositório, e a implementação com o ORM é injetada."

**Benefícios:** regra de negócio testável isoladamente, troca de banco ou framework com menos impacto, código organizado por responsabilidade.

**Trade-off (mostra maturidade):** "Gera mais arquivos e abstrações. Para um CRUD simples pode ser exagero; vale quando a regra de negócio é relevante."

Prepare como era a estrutura de pastas no Leakage, para descrever com segurança.

## 5. Angular

**Resposta-base:** "Na Itriad tive uma experiência mais breve com Angular, em um projeto com backend em C#, e também tenho um projeto próprio em Angular 13 com TypeScript, componentes, módulos e rotas. Meu frontend mais extenso foi com React, mas os fundamentos são os mesmos, e o modelo do Angular, com injeção de dependência e estrutura opinativa, conversa bem com a forma como organizo o backend."

Tenha um exemplo concreto do que fez no Angular na Itriad, mesmo simples (uma tela, um formulário, uma integração com a API).

**Revisar hoje:**

- services com HttpClient
- Observables (RxJS)
- formulários reativos
- guards de rota
- interceptors (para enviar o token JWT)

## 6. Testes e TDD (ponto de atenção)

TDD é requisito da vaga e testes não apareceram no Leakage. Não afirme TDD se não praticava.

**Conceitos para dominar:**

- **TDD:** escrever o teste antes (falha), implementar o mínimo para passar, refatorar.
- **Pirâmide:** muitos unitários, menos de integração, poucos e2e.
- **Ligação com Clean Architecture:** casos de uso isolados são fáceis de testar com o repositório mockado.
- **Ferramenta:** Jest é o padrão em Node/TS.

**Resposta-base:** "[]. Entendo o TDD e o valor dele, principalmente em regra de negócio, e a Clean Architecture facilita muito isso, porque o caso de uso pode ser testado sem banco."

## 7. Docker e CI

- **Docker:** mesma imagem em todos os ambientes; docker-compose para subir API e banco localmente; multi-stage build para imagem menor.
- **CI:** "O fluxo era Gitflow, com pipeline no GitHub (GitHub Actions) rodando build e deploy a cada merge." Se não tiver certeza de que era Actions, diga apenas "pipeline no GitHub".
- **Gitflow:** feature, develop, release, master/main, hotfix. É a estratégia de branches, não o pipeline.

## 8. Escalabilidade e disponibilidade

Use a experiência real de AWS na FITEC como base da resposta.

**Conceitos de apoio:** API sem estado (stateless) para escalar horizontalmente, cache, índices no banco, processamento assíncrono com filas e observabilidade (logs, métricas, alertas).

**Gancho com o Leakage:** "envio de e-mail é um bom candidato a processamento assíncrono, para não travar a requisição." Só diga que fizeram assim se foi o caso; senão, use como "o que eu faria".

## 9. Documentação

Evidência real: documentação de API com OpenAPI/Swagger (projeto Node + MongoDB). Complementar com README de setup, diagramas de arquitetura e registro de decisões (ADR).

## 10. Projeto Eldorado: "o que eu faria diferente hoje"

Se o gestor abrir o GitHub ou perguntar sobre projetos pessoais:

- segredo do JWT em variável de ambiente, nunca fixo no código;
- aplicar os validadores nas rotas;
- separar casos de uso dos controllers (Clean Architecture);
- testes, Docker e pipeline;
- tratamento de erro centralizado;
- completar a integração do front Angular com a API.

- Remover o `.env` do repositório público hoje

## 11. Perguntas para fazer ao gestor

1. Qual é o projeto e em que fase ele está?
2. Como é o time hoje e onde essa posição se encaixa?
3. Como funciona o fluxo: sprints, code review, pipeline, testes?
4. Qual o maior desafio técnico atual?
5. O que seria um bom resultado nos primeiros 3 meses?

