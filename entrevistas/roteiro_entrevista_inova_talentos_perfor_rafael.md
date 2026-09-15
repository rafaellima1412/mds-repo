# Roteiro de Entrevista — Vaga Inova Talentos / PERFOR
### Rafael Barbosa de Lima

---

## Leitura honesta da vaga antes de tudo

Essa vaga tem dois requisitos **obrigatórios** onde você tem gap real:
- **Java** — você estudou com Spring Boot, mas nunca usou profissionalmente.
- **Kafka + Kafka Connect** — você nunca usou; sua experiência de mensageria é toda com SQS/Celery.

Isso muda a estratégia: em vez de tentar disfarçar, a abordagem que mais funciona nesse tipo de programa (Inova Talentos costuma ter caráter de capacitação/pesquisa, não só contratação pura) é ser **direto sobre onde você está**, mostrar **velocidade de aprendizado comprovada** (você trocou de stack várias vezes na carreira: Python → Go → C# → Python de novo, sempre entregando) e apoiar-se forte nos pontos onde você é realmente bom: **AWS, PostgreSQL, arquitetura, CI/CD**.

⚠️ Se o processo tiver uma etapa técnica prática em Java/Kafka, vale considerar estudar o mínimo essencial antes (Spring Boot básico + conceitos de Kafka) — ver seção final "Estudo mínimo recomendado".

---

## Bloco 1 — Abertura

**1. "Fale sobre você e sua trajetória profissional."**

> **Resposta-modelo:** "Minha trajetória sempre teve foco em backend e arquitetura, passando por diferentes stacks — Python, Go, C# — sempre aplicando os mesmos princípios de Clean Architecture e boas práticas, independente da linguagem. Construí minha base em Go na ITRIAD, evoluí pra arquitetura completa em Python/AWS na FITEC, e mais recentemente atuei como freelancer ponta a ponta num projeto de logística fluvial, arquitetando o backend em Go com Design Hexagonal e cuidando de toda a infraestrutura AWS."

**2. "Por que se interessou por essa vaga e pelo programa Inova Talentos?"**

> **Resposta-modelo:** "Me interessei porque é uma oportunidade de unir o que já domino — arquitetura, AWS, backend — com tecnologias que quero aprofundar, como Java e Kafka. O caráter de pesquisa e inovação do programa também me atrai, porque minha trajetória sempre teve um lado de resolver problema com solução simples e bem fundamentada, não só aplicar tecnologia por aplicar."

**3. "Disponibilidade para viagens — isso é possível pra você?"**

> ⚠️ Isso é um requisito explícito da vaga. Responda com sinceridade — se você tem disponibilidade real, confirme com segurança; se tiver alguma restrição, é melhor alinhar isso agora do que depois de contratado.

---

## Bloco 2 — Java (o ponto mais delicado — trate com honestidade estratégica)

**4. "Fale sobre sua experiência com Java."**

> **Resposta-modelo:** "Não tenho experiência profissional com Java em produção, mas estudei a linguagem com Spring Boot, incluindo injeção de dependência, controllers REST e a estrutura de camadas que o framework propõe. Minha experiência principal é em Go e Python, aplicando Clean Architecture e SOLID no dia a dia — e boa parte desses conceitos são os mesmos que o ecossistema Java formaliza: inversão de controle, separação de responsabilidades, testes com mocks. Já troquei de stack várias vezes na carreira e sempre me adaptei rápido porque meu foco é mais em arquitetura do que em sintaxe específica de uma linguagem."

**5. "Você se sente confortável assumindo um projeto majoritariamente em Java?"**

> **Resposta-modelo:** "Sim, com a ressalva honesta de que levaria algumas semanas pra ganhar fluência total na sintaxe e no ecossistema específico (Maven/Gradle, convenções do Spring). Mas os conceitos de arquitetura, que são a parte mais difícil de aprender, eu já domino — então minha curva de aprendizado seria mais rápida que a de alguém começando do zero em desenvolvimento de software."

---

## Bloco 3 — React

**6. "Fale sobre sua experiência com React."**

> **Resposta-modelo:** "Tenho contato prático real com React em dois projetos: na Locus, um projeto comercial com Python no backend e React no frontend, e na ITRIAD, um sistema de qualidade industrial em Go no backend e React no frontend, que notificava defeitos detectados em peças. Meu foco sempre foi mais backend, mas tenho vivência real de como o frontend consome as APIs que eu desenho, o que ajuda a pensar em contratos de API mais úteis pra quem consome do lado do cliente."

---

## Bloco 4 — AWS (seu ponto mais forte — aproveite bem essa pergunta)

**7. "Quais serviços AWS você já usou?"**

> **Resposta-modelo (cite por categoria conforme a pergunta específica):**
> - Rede: VPC, subnets, tabelas de rota, balanceadores de carga, Target Group, Transit Gateway
> - Computação: EC2, ECS, Lambda
> - Banco de dados: RDS, DocumentDB
> - Armazenamento: S3
> - Integração/mensageria: API Gateway, SNS, SQS, SES
> - Autenticação: Cognito
> - IaC: CloudFormation, CDK
> - Segurança: IAM
> - Observabilidade: CloudWatch

**8. "Descreva um pipeline de CI/CD que você já construiu."**

> **Resposta-modelo:** "Um exemplo real foi um pipeline com AWS CodeBuild, usando um `buildspec.yml` com três fases: pre_build (autenticação no ECR), build (build da imagem Docker), e post_build (push da imagem e geração do `imagedefinitions.json`). Além disso, um `appspec.yml` integrava com o CodeDeploy pra automatizar o deploy no ECS a partir dessa imagem. Isso automatizava o ciclo do commit até a aplicação rodando em produção."

---

## Bloco 5 — Kafka + Kafka Connect (segundo ponto delicado)

**9. "Você já trabalhou com Kafka ou Kafka Connect?"**

> **Resposta-modelo:** "Não tenho experiência prática com Kafka especificamente. Minha experiência de mensageria foi com SQS da AWS — publicando eventos que disparavam processamento assíncrono, com filas mortas (DLQ) pra tratar falha sem perder mensagem — e com Celery, usando SQS como broker, para tarefas assíncronas e agendadas, incluindo controle de concorrência de worker e confirmação tardia de processamento pra garantir que falhas não perdessem tarefas. Entendo os conceitos centrais que também se aplicam a Kafka — publish/subscribe, consumidores, tratamento de falha — mas reconheço que a ferramenta específica e o Kafka Connect são uma lacuna que preciso desenvolver."

**10. "O que você sabe sobre a diferença entre Kafka e uma fila tradicional como SQS?"**

> **Estudo rápido antes da entrevista (conceito, não prática):** Kafka é voltado a **streaming de eventos** em alto volume, com **retenção de mensagens** por um período configurável (a mensagem não desaparece ao ser lida, como numa fila tradicional) e **múltiplos consumidores independentes** podem ler o mesmo tópico, cada um no seu próprio ritmo. SQS é mais simples: uma mensagem é consumida e geralmente removida da fila após processamento (ponto a ponto). Kafka Connect é um framework pra conectar Kafka a sistemas externos (bancos, outras filas) sem escrever código de integração do zero — usando "connectors" prontos.
>
> **Resposta-modelo:** "Pelo que estudei, a diferença central é que o SQS, que já usei, é uma fila ponto a ponto — a mensagem some depois de processada. Kafka é voltado a streaming de eventos, com retenção configurável, permitindo que múltiplos consumidores leiam o mesmo tópico de forma independente. O Kafka Connect facilita integrar o Kafka com sistemas externos sem escrever toda a lógica de integração manualmente, usando conectores prontos."

---

## Bloco 6 — Diferenciais (seus pontos fortes reais)

**11. "Fale sobre sua experiência com PostgreSQL."**

> **Resposta-modelo:** "Tenho experiência sólida com PostgreSQL — modelagem, otimização de queries usando EXPLAIN, e resolução de problemas reais como sessões fantasmas ocupando o pool de conexões, que ajustei recalibrando a capacidade do pool com base em recursos reais de infraestrutura, como disco e processador."

**12. "E com Redis?"**

> **Resposta-modelo:** "Estudei Redis, mas ainda não tive oportunidade de aplicar em produção. Entendo os conceitos de cache — TTL, estratégias de invalidação — e tenho facilidade pra aplicar isso na prática assim que surgir a oportunidade."

**13. "Como está seu inglês?"**

> **Resposta-modelo:** "Meu inglês está no nível intermediário — consigo ler documentação técnica e me comunicar em conversas mais simples, e venho praticando ativamente pra evoluir, principalmente o lado falado."
>
> 💡 Essa é uma das poucas vagas até agora onde seu nível real de inglês **bate exatamente** com o que é pedido — não é gap aqui, pode responder com mais segurança que nas vagas anteriores.

---

## Bloco 7 — Sobre o que você vai fazer no dia a dia

**14. "Como você lida com correção de bugs em código que não foi você quem escreveu?"**

> **Resposta-modelo:** "Começo entendendo o contexto antes de mexer — reproduzo o bug, leio os testes existentes (se houver) pra entender o comportamento esperado, e só depois parto pra correção. Prefiro corrigir a causa raiz a aplicar um paliativo, mesmo que isso peça mais investigação inicial. Um exemplo real disso foi na FITEC, onde investiguei sessões fantasmas ocupando o pool de conexões do banco — não era um bug óbvio, precisei investigar métricas e comportamento real antes de encontrar a causa."

**15. "Você tem perguntas para nós?"**

> **Sugestões:**
> - "Como funciona o processo de capacitação do programa Inova Talentos — há treinamento formal em Java/Kafka pra quem entra com mais experiência em outras stacks?"
> - "Como é a rotina de viagens da vaga — frequência e destinos costumam variar muito?"
> - "O time atua mais em manutenção de sistemas existentes ou desenvolvimento de funcionalidades novas?"

---

## Estudo mínimo recomendado antes da entrevista (se houver etapa técnica)

Dado que você já estudou Spring Boot, um reforço de 2-3 dias focado nisso já ajuda bastante:

- **Java:** revisar sintaxe básica, Streams API, e um CRUD simples com Spring Boot (Controller → Service → Repository).
- **Kafka:** assistir 1-2 vídeos introdutórios sobre producer/consumer/tópicos/partições, e o que faz o Kafka Connect — não precisa implementar nada, só entender o vocabulário pra não travar numa pergunta conceitual.

---

## Checklist final

1. Seja direto sobre os gaps de Java e Kafka — não tente disfarçar, mostre capacidade de aprendizado rápido com exemplos reais da sua trajetória (trocou de stack várias vezes).
2. AWS, PostgreSQL e CI/CD são seus trunfos nessa entrevista — capriche nessas respostas.
3. Inglês, pela primeira vez, não é um gap — responda com confiança.
4. Confirme a disponibilidade de viagens com sinceridade antes de qualquer outra coisa.
5. Não invente experiência com Kafka nem Java em produção — se pegarem você numa pergunta de profundidade que não sabe, é melhor dizer "não tenho certeza, mas [raciocínio conceitual]" do que travar tentando inventar.
