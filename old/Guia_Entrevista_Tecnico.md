# GUIA DE ENTREVISTA — TÉCNICO

## 1. Perfil e posicionamento
- Backend com foco em Python/FastAPI, APIs REST, integrações, automações e PostgreSQL.
- Experiência prática com Clean Architecture, Use Cases, Repositories, DI, testes, Docker e AWS.
- Atuação como referência técnica: revisão de PRs, mentoria, decomposição de tarefas, estimativas, riscos, incidentes e decisões arquiteturais.
- Liderança técnica sem depender de autoridade formal.

## 2. Python / FastAPI
### Pontos para revisar
- FastAPI, Pydantic v2, SQLAlchemy, Alembic.
- Dependency Injection.
- Routers chamam Use Cases.
- Use Cases concentram fluxo/regras de negócio.
- Repositories concentram persistência.
- DTOs separam entrada, domínio e saída.
- pytest + TestClient.
- Mocks/Fakes para testar Use Cases sem banco real.

### Resposta-chave
“Eu procuro manter a regra de negócio independente de infraestrutura. O Use Case coordena o fluxo, o Repository abstrai persistência e as dependências são injetadas. Isso facilita testes e manutenção.”

## 3. Clean Architecture / SOLID
### SRP
Uma responsabilidade bem definida.

### Dependency Inversion
O Use Case depende de uma abstração, não de PostgreSQL diretamente.

Exemplo:
CreateUser → UserRepository (interface) → PostgresUserRepository

### Por que isso importa?
- Testabilidade
- Baixo acoplamento
- Manutenção
- Troca de infraestrutura
- Clareza de responsabilidades

## 4. APIs REST
Revisar:
- HTTP methods
- Status codes
- autenticação/autorização
- validação
- paginação
- tratamento de erros
- versionamento quando necessário
- idempotência em operações apropriadas

## 5. Webhooks
### Conceito
Um sistema notifica outro quando um evento acontece.

### Experiência Microsoft Graph (Conecta serviços: como Outlook)
- Subscription para eventos.
- Endpoint `/webhook/events`.
- Validação por `validationToken`.
- Processamento do payload.
- Tratamento de eventos `deleted`.
- Renovação periódica porque subscriptions expiram.

### Boa resposta
“Webhook é um mecanismo orientado a eventos. Eu recebo a notificação, valido a requisição, processo ou publico para processamento assíncrono e respondo rapidamente. Como subscriptions podem expirar, também é necessário cuidar da renovação.”

## 6. Segurança / OWASP
### Authentication x Authorization
- Authentication: quem é você?
- Authorization: o que você pode acessar/fazer?

### Broken Access Control
Nunca confiar apenas em token válido. Verificar se aquele usuário pode acessar aquele recurso.

### XSS
Não é resolvido simplesmente com regex.
Defesas:
- validação
- sanitização quando HTML é permitido
- escaping/encoding de saída=- **O dado no banco/API:** `<script>` **O dado na saída (o que o React envia para o HTML do navegador):** `&lt;script&gt;`
- framework escaping = **HTML Escaping** + **SQL Escaping**
- CSP ==> Content-Security-Policy: default-src 'self'; script-src 'self' https://meudominio.com


### SQL Injection
- ORM ajuda.
- Queries parametrizadas também são seguras.
- Evitar SQL construído com f-string/input direto.  sim ---># O ":" ou "%s" (depende do banco) indica que ali vai um parâmetro seguro
  não --> query = f"SELECT * FROM users WHERE username = '{username}'" # Se o username for: admin' OR '1'='1 # A query vira: SELECT * FROM users WHERE username = 'admin' OR '1'='1' (Acessa tudo!)

### CSRF
O problema é um navegador autenticado executar ação indevida.
Defesas:
- CSRF token
- SameSite cookies
- configuração adequada de cookies
- expiração reduz janela de exposição, mas não substitui proteção CSRF.

### LGPD
Revisar:
- minimização de dados
- finalidade
- acesso restrito
- proteção de dados pessoais
- retenção
- auditoria quando aplicável.

## 7. CI/CD
### Fluxo prático
1. Branch de desenvolvimento.
2. Desenvolvedor cria branch (`feature`, `fix`, `bug`).
3. Implementa.
4. Testes + lint/pre-commit.
5. Pull Request.
6. Code Review.
7. QA.
8. Pipeline.
9. Deploy.
10. Monitoramento.
11. Rollback se necessário.

### AWS
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

### Importante
Pre-commit é uma validação local. CI deve repetir as validações no servidor.

## 8. Deploy
### Rolling
V1 V1 V1
→ V2 V1 V1
→ V2 V2 V1
→ V2 V2 V2

### Canary
V1 atende quase todos.
V2 começa com pequena porcentagem:
5% → 20% → 50% → 100%

Se houver erro, interrompe/rollback.

### Rollback
Voltar para a última versão estável quando o novo deploy apresenta impacto relevante.

## 9. Incidente em produção
Resposta ideal:
“Primeiro procuro reduzir o impacto para o usuário. Verifico métricas, logs e alarmes e correlaciono com o último deploy. Se houver forte indício de regressão e uma versão estável disponível, faço rollback. Depois da estabilização, investigo causa raiz e implemento a correção definitiva.”


## 14. Filas / eventos
Quando webhook ou integração pode sofrer indisponibilidade:
Webhook → valida → publica na fila → responde 2xx → worker processa.

Benefícios:
- desacoplamento
- processamento assíncrono
- retry
- resiliência
- DLQ para falhas persistentes

## 15. Idempotência
Mesma mensagem pode chegar mais de uma vez.
Usar identificador único do evento e uma garantia de processamento único, normalmente apoiada por constraint/controle transacional.

Não aprofundar em Outbox a menos que o entrevistador peça.

## 16. SRE / Observabilidade
Revisar:
- logs
- métricas
- traces
- alertas
- health checks
- disponibilidade
- latência
- taxa de erro
- capacidade

Ideia central:
“Não basta o sistema funcionar; precisamos conseguir perceber quando ele está degradando.”

## 17. Git
Conhecer:
- feature branches=Boas Práticas para Nomear
- PR
- code review
- GitFlow =é um **modelo de gestão de ramos (branching model)** para o Git
- trunk-based development =Se precisares de criar uma branch para uma tarefa (`feature/*`), ela deve durar no máximo **um ou dois dias**
- merge/rebase = usado no trunk-based development o dev deve ficar atualizando constantemente sua brach.
- tags/releases = tags de versão.

Não confundir estratégia de branches com CI/CD.

## 18. DDD
Revisar apenas o essencial:
- domínio
- entidades
- value objects
- agregados
- serviços de domínio
- bounded contexts

Não tentar decorar termos sem entender o problema que resolvem.


## 20. Regra para responder
Estrutura:
Contexto → decisão → motivo → resultado/aprendizado.

Evitar respostas excessivamente teóricas. Priorizar exemplos reais.

