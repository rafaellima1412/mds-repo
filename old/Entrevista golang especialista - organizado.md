# Estudo Go — Payment Resilience

![[camadas_resiliencia_payment.png]]

Material de estudo baseado no projeto `payment-resilience` e nos conceitos que fomos construindo durante a conversa.

                         jobs (unbuffered)
   pagamentos ──producer──►  chan gateway.Pagamento                ┐
     (15 itens)                                                                                     │
                                                      ├──► worker 0 ─┐
                                                      ├──► worker 1 ─┤
                                                      ├──► worker 2 ─┤
                                                      └──► worker 3 ─┘
                                                              │
                                                              ▼
                                              CB.Executar( retry.Do( gateway.Cobrar ) )
                                                              │
                                                              ▼
                                        results (buffered, cap = len(pagamentos))
                                                              │
                                                              ▼
                                             main.go: for r := range results
                                                     (imprime + conta aprovados/falhados)
                                                     

CLOSED ──[3 falhas seguidas]──► OPEN
OPEN   ──[passou 1500ms + nova chamada]──► HALF-OPEN
HALF-OPEN ──[2 sucessos seguidos]──► CLOSED
HALF-OPEN ──[1 falha]──► OPEN   (direto, não espera acumular 3 de novo)

goroutine main()
  │
  │  chama pool.Processar(ctx, pagamentos)
  ▼
┌───────────────────────────────────────────────────────────────┐
│ Processar()  cria estas goroutines e retorna results na hora:  │
│                                                                 │
│  go func() { produtora }()          ──► 1 goroutine            │
│  go p.worker(0, ...)                ──► 1 goroutine            │
│  go p.worker(1, ...)                ──► 1 goroutine            │
│  go p.worker(2, ...)                ──► 1 goroutine            │
│  go p.worker(3, ...)                ──► 1 goroutine            │
│  go func() { wg.Wait(); close() }() ──► 1 goroutine            │
└───────────────────────────────────────────────────────────────┘
  │
  ▼
main() continua rodando e faz:
  for r := range results { ... }   ◄── bloqueia até ter dado ou canal fechar


┌─────────────────────────── linha do tempo ───────────────────────────┐
│                                                                        │
│  [main]  ──chama Processar()──► [retorna results, main segue]        │
│    │                                                                   │
│    ├──► [produtora]   escreve em jobs, um a um, até acabar a slice    │
│    │                    depois: close(jobs)                            │
│    │                                                                   │
│    ├──► [worker 0] ──┐                                                │
│    ├──► [worker 1] ──┤  todos lêem de `jobs`, processam               │
│    ├──► [worker 2] ──┤  (CB+retry+timeout), escrevem em `results`      │
│    ├──► [worker 3] ──┘  saem do loop quando jobs fecha                │
│    │                                                                   │
│    └──► [fechadora]  wg.Wait() bloqueada até os 4 workers            │
│                        chamarem wg.Done() (via defer)                  │
│                        aí faz close(results)                          │
│                                                                         │
│  [main] estava em `for r := range results`, recebendo cada resultado  │
│  conforme os workers publicam, e sai do loop quando results fecha     │
└─────────────────────────────────────────────────────────────────────┘
┌─────────────────────────────────────────────────────────────────────┐
│ CAMADA 1 — context do batch (ctx, WithTimeout 20s)                   │
│ Protege contra: processo rodando pra sempre                          │
│ Efeito: se estourar, cancela TUDO em cascata (produtora+workers)     │
│                                                                        │
│  ┌───────────────────────────────────────────────────────────────┐  │
│  │ CAMADA 2 — produtora escuta ctx.Done() ao escrever em `jobs`   │  │
│  │ Protege contra: goroutine leak                                  │  │
│  │ (sem isso: trava pra sempre tentando escrever em jobs           │  │
│  │  se o ctx cancelar no meio do envio)                            │  │
│  │                                                                   │  │
│  │  ┌─────────────────────────────────────────────────────────┐  │  │
│  │  │ CAMADA 3 — WaitGroup + goroutine fechadora                │  │  │
│  │  │ Protege contra: deadlock no `for range results` do main   │  │  │
│  │  │ (sem isso: main ficaria esperando pra sempre um            │  │  │
│  │  │  channel `results` que nunca fecha)                        │  │  │
│  │  │                                                              │  │  │
│  │  │  ┌───────────────────────────────────────────────────┐  │  │  │
│  │  │  │ CAMADA 4 — results bufferizado (cap=len(pagamentos))│  │  │  │
│  │  │  │ Protege contra: workers bloqueados esperando         │  │  │  │
│  │  │  │ alguém consumir (desacopla produção/consumo)         │  │  │  │
│  │  │  │                                                        │  │  │  │
│  │  │  │  ┌─────────────────────────────────────────────┐  │  │  │  │
│  │  │  │  │ CAMADA 5 — context por operação                │  │  │  │  │
│  │  │  │  │ (WithTimeout derivado do ctx do batch)          │  │  │  │  │
│  │  │  │  │ Protege contra: 1 pagamento lento travar        │  │  │  │  │
│  │  │  │  │ o worker pra sempre                              │  │  │  │  │
│  │  │  │  │                                                    │  │  │  │  │
│  │  │  │  │  ┌───────────────────────────────────────────┐│  │  │  │  │
│  │  │  │  │  │ CAMADA 6 — circuit breaker                   ││  │  │  │  │
│  │  │  │  │  │ Protege contra: martelar gateway fora do ar  ││  │  │  │  │
│  │  │  │  │  │ (falha rápido em vez de gastar tempo/recurso)││  │  │  │  │
│  │  │  │  │  │                                                ││  │  │  │  │
│  │  │  │  │  │  ┌─────────────────────────────────────┐  ││  │  │  │  │
│  │  │  │  │  │  │ CAMADA 7 — retry + backoff + jitter    │  ││  │  │  │  │
│  │  │  │  │  │  │ Protege contra: falha transiente        │  ││  │  │  │  │
│  │  │  │  │  │  │ isolada virar erro definitivo            │  ││  │  │  │  │
│  │  │  │  │  │  │ (nunca re-tenta erro de negócio!)         │  ││  │  │  │  │
│  │  │  │  │  │  │                                             │  ││  │  │  │  │
│  │  │  │  │  │  │   ┌───────────────────────────────┐    │  ││  │  │  │  │
│  │  │  │  │  │  │   │  gateway.Cobrar(ctx, pag)       │    │  ││  │  │  │  │
│  │  │  │  │  │  │   │  (a chamada real / instável)    │    │  ││  │  │  │  │
│  │  │  │  │  │  │   └───────────────────────────────┘    │  ││  │  │  │  │
│  │  │  │  │  │  └─────────────────────────────────────┘  ││  │  │  │  │
│  │  │  │  │  └───────────────────────────────────────────┘│  │  │  │  │
│  │  │  │  └─────────────────────────────────────────────┘  │  │  │  │
│  │  │  └───────────────────────────────────────────────────┘  │  │  │
│  │  └─────────────────────────────────────────────────────────┘  │  │
│  └───────────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────┐
│ A) CONTROLE DE VIDA DAS GOROUTINES            │
│    (garante que tudo nasce e morre certo)     │
│    → ctx do batch, ctx.Done() na produtora,   │
│      WaitGroup + fechadora, buffer do results │
├─────────────────────────────────────────────┤
│ B) CONTROLE DE TEMPO                          │
│    (garante que nada trava pra sempre)        │
│    → context.WithTimeout por operação          │
├─────────────────────────────────────────────┤
│ C) RESILIÊNCIA CONTRA O GATEWAY                │
│    (garante que falha externa não vira caos)   │
│    → circuit breaker (por fora)                │
│    → retry + backoff (por dentro)              │
└─────────────────────────────────────────────┘

## Ordem de estudo

```text
1. Retry
2. Backoff
3. Jitter
4. Circuit Breaker
5. Worker Pool
6. Channels / select
7. Goroutine leaks
8. Mutex / Race Condition
9. Producer / Consumer
10. Filas
11. Sistemas distribuídos
12. Kafka / RabbitMQ
```

---

# 1. Retry ✅

Retry permite tentar novamente uma operação que falhou quando existe possibilidade de recuperação.

Exemplos de falhas que podem ser transitórias:

- timeout
- HTTP 502
- HTTP 503
- falha temporária de conexão

Erros de negócio normalmente **não devem sofrer retry**.

```text
Pagamento
   ↓
Gateway
   ↓
503
   ↓
Retry
   ↓
nova tentativa
```

### Retry + Context

O `Retry` decide se vale tentar novamente.

O `context` decide se a operação ainda pode continuar.

> Retry decide se vale tentar novamente; o context decide se ainda podemos continuar.

O retry deve verificar o contexto antes de uma nova tentativa e também durante o backoff.

---

# 2. Backoff ✅

Backoff adiciona uma espera entre as tentativas.

Com exponential backoff, a espera pode crescer assim:

```text
100ms
200ms
400ms
800ms
1600ms
```

A ideia é evitar novas tentativas imediatas contra um serviço que acabou de falhar.

### Backoff e deadline

O backoff não pode crescer indefinidamente.

Ele precisa respeitar:

- deadline do `context`
- limite máximo de espera
- quantidade máxima de retries

Exemplo:

```text
Context: 3s

Tentativa 1
   ↓
backoff
   ↓
Tentativa 2
   ↓
backoff
   ↓
context expirou
   ↓
não tenta novamente
```

> Se o próprio backoff ultrapassar o tempo restante da operação, não faz sentido continuar esperando para tentar novamente.

---

# 3. Jitter ✅

Jitter adiciona uma pequena variação aleatória ao backoff.

Sem jitter, várias instâncias podem falhar e fazer retry exatamente ao mesmo tempo:

```text
API 1 → 1s → retry
API 2 → 1s → retry
API 3 → 1s → retry
API 4 → 1s → retry
```

Com jitter:

```text
API 1 → 1.1s → retry
API 2 → 1.7s → retry
API 3 → 0.9s → retry
API 4 → 1.4s → retry
```

O objetivo é espalhar os retries e reduzir o efeito de sincronização entre instâncias.

> Jitter reduz a chance de várias instâncias voltarem ao mesmo tempo para um serviço que já está apresentando problemas.

---

# 4. Circuit Breaker ✅

O Circuit Breaker protege o sistema quando um downstream está apresentando muitas falhas.

Enquanto o Retry pergunta:

> Essa operação merece outra tentativa?

O Circuit Breaker pergunta:

> Devemos continuar chamando esse downstream?

## Estados

```text
CLOSED
   ↓ muitas falhas
OPEN
   ↓ cooldown
HALF-OPEN
   ↓ sucesso suficiente
CLOSED
```

### CLOSED

Funcionamento normal. As chamadas são permitidas.

### OPEN

As chamadas para o downstream são bloqueadas.

```text
Pagamento
   ↓
Circuit Breaker OPEN
   ↓
❌ não chama Gateway
```

Nesse estado, não faz sentido continuar executando retries contra um serviço que o Circuit Breaker já identificou como problemático.

### HALF-OPEN

Depois do `Cooldown`, o Circuit Breaker permite uma chamada de teste para verificar se o serviço se recuperou.

Se funcionar:

```text
HALF-OPEN → CLOSED
```

Se falhar:

```text
HALF-OPEN → OPEN
```

## Máquina de estados do projeto

O projeto define:

```go
type Estado int

const (
    Closed Estado = iota
    Open
    HalfOpen
)
```

E possui uma função central para efetivar a mudança:

```go
func (cb *CircuitBreaker) transicionar(novo Estado)
```

Ela registra o estado anterior, atualiza o novo estado e, quando configurado, dispara:

```go
OnMudancaEstado(anterior, novo)
```

Esse callback pode ser usado para logs, métricas e monitoramento.

### Configurações importantes

```go
LimiteFalhas
```

Número de falhas consecutivas que fazem:

```text
CLOSED → OPEN
```

```go
Cooldown
```

Tempo que o circuito fica `OPEN` antes de permitir o teste:

```text
OPEN → HALF-OPEN
```

```go
SucessosParaFechar
```

Número de sucessos consecutivos em `HALF-OPEN` necessários para:

```text
HALF-OPEN → CLOSED
```

### Ponto importante

A mudança de estado pertence ao próprio Circuit Breaker.

O Retry **não** é responsável por fazer:

```text
OPEN → HALF-OPEN
```

nem por fazer:

```text
HALF-OPEN → CLOSED
```

O Circuit Breaker usa seu timer/cooldown e o resultado das chamadas para controlar essas transições.

---

# 5. Retry + Circuit Breaker ✅

Os dois mecanismos trabalham juntos, mas têm responsabilidades diferentes.

```text
Pagamento
   ↓
Circuit Breaker
   ↓
Retry
   ↓
Gateway
```

O Retry atua na operação individual.

O Circuit Breaker atua sobre a saúde do downstream.

Quando o circuito está `OPEN`, a requisição pode ser rejeitada imediatamente sem entrar no fluxo de retry.

---

# 6. Worker Pool ✅

Eu poderia criar uma goroutine para cada pagamento, mas isso não significa que essa seja uma boa estratégia em escala.

O Worker Pool limita a quantidade de operações concorrentes.

Exemplo:

```text
10.000 pagamentos
       ↓
     jobs
       ↓
 ┌─────┬─────┬─────┐
 │ W1  │ W2  │ W3  │ ...
 └─────┴─────┴─────┘
```

Com 10 workers, por exemplo, o sistema mantém uma quantidade controlada de processamento concorrente.

Isso ajuda a evitar sobrecarga de:

- CPU
- memória
- conexões
- banco de dados
- serviços externos

> O principal benefício do Worker Pool é controlar concorrência. Ele não existe principalmente para evitar mutex.

---

# 7. Channels / select ✅

Em Go, channels são seguros para uso concorrente.

Eles permitem comunicação e sincronização entre goroutines.

```text
CHANNEL
   ↓
Comunicação / sincronização
   ↓
Goroutines
```

Várias goroutines podem enviar valores para o mesmo channel sem precisar de mutex para proteger o próprio channel.

## Unbuffered

```go
jobs := make(chan Pagamento)
```

O envio pode bloquear até existir um receiver disponível.

```text
Producer
   │
   │ jobs <- pagamento
   ▼
[ BLOQUEADO ]
   │
   │ espera receiver
   ▼
Worker recebe
   │
   ▼
envio continua
```

## Buffered

```go
jobs := make(chan Pagamento, 100)
```

O channel consegue armazenar até 100 itens antes de bloquear o produtor.

```text
Producer
   │
   ▼
┌─────────────────┐
│ jobs buffer     │
└────────┬────────┘
         ▼
       Workers
```

Quando o buffer estiver cheio e os workers estiverem ocupados, o produtor volta a bloquear.

Isso introduz o conceito de **backpressure**.

## select

Permite aguardar múltiplas situações.

Exemplo:

```go
select {
case jobs <- pagamento:
    // enviou
case <-ctx.Done():
    // cancelado
}
```

Assim, o produtor não fica preso indefinidamente se o contexto for cancelado.

---

# 8. Fechamento de Channels ✅

Quando não haverá mais valores, o produtor/orquestrador normalmente fecha o channel:

```go
close(jobs)
```

Workers podem consumir assim:

```go
for job := range jobs {
    process(job)
}
```

O worker continua processando os valores que já estavam no channel. Quando o channel estiver **fechado e vazio**, o `range` termina.

> Quem envia normalmente é quem fecha o channel.

O worker não sabe se outro produtor ainda poderá enviar alguma coisa.

### Duas formas de terminar um worker

```text
Worker
   │
   ├── jobs fechado ──→ return
   │
   └── ctx cancelado ─→ return
```

> `close()` não destrói o channel nem encerra goroutines automaticamente.

Enviar para um channel fechado causa `panic`.

---

# 9. sync.WaitGroup ✅

O `WaitGroup` serve para a goroutine principal esperar outras goroutines terminarem.

Exemplo:

```go
var wg sync.WaitGroup

wg.Add(3)

// worker 1
wg.Done()

// worker 2
wg.Done()

// worker 3
wg.Done()

wg.Wait()
```

Os métodos principais são:

```go
wg.Add(n) // registra quantas goroutines devem terminar
wg.Done() // sinaliza que uma terminou
wg.Wait() // espera todas terminarem
```

Visualmente:

```text
main
 │
 ├── Worker 1 ──┐
 ├── Worker 2 ──┤
 ├── Worker 3 ──┤
 │              │
 │           wg.Wait()
 │              │
 └──────────────┘
       terminou
```

### WaitGroup não evita goroutine leak

Se uma goroutine ficar bloqueada para sempre e nunca executar `Done()`, o `wg.Wait()` também pode ficar bloqueado para sempre.

Portanto:

```text
Context
  ↓
controla cancelamento

WaitGroup
  ↓
espera término
```

Os dois podem trabalhar juntos.

---

# 10. Goroutine Leaks ✅

Uma goroutine leak acontece quando uma goroutine fica bloqueada e não consegue terminar quando deveria.

Exemplo problemático:

```go
func worker(jobs <-chan Pagamento) {
    for {
        job := <-jobs
        process(job)
    }
}
```

Se ninguém enviar mais nada e o channel nunca for fechado, essa goroutine pode ficar bloqueada indefinidamente.

Uma alternativa:

```go
for job := range jobs {
    process(job)
}
```

Ou usar `context`:

```go
select {
case job, ok := <-jobs:
    if !ok {
        return
    }
    process(job)

case <-ctx.Done():
    return
}
```

### Observação importante sobre Context

`context.Cancel()` **não mata uma goroutine**.

O cancelamento é cooperativo.

A goroutine precisa observar:

```go
<-ctx.Done()
```

ou:

```go
ctx.Err()
```

Se uma operação bloqueante ignorar o contexto, ela pode continuar presa mesmo após o cancelamento.

---

# 11. Context + Timeout ✅

O `context.Context` controla o ciclo de vida das operações.

Podemos ter uma hierarquia:

```text
batch
  ↓
pool
  ↓
worker
  ↓
retry
  ↓
gateway
```

Exemplo:

```text
Batch: 10s

Pagamento 1: 2s
Pagamento 2: 3s
Pagamento 3: 8s
Pagamento 4: 15s  ← não pode ficar eternamente esperando
```

Também podemos derivar contextos filhos:

```text
ctxBatch
   │
   ├── ctxPagamento1
   ├── ctxPagamento2
   ├── ctxPagamento3
   └── ctxPagamento4
```

Um filho pode ser cancelado individualmente.

Se o pai for cancelado, o cancelamento é propagado para os filhos.

### Timeout por operação

Podemos ter, por exemplo:

```text
Batch timeout: 30s
Pagamento timeout: 3s
```

O timeout do pagamento limita aquela operação sem necessariamente cancelar todos os outros pagamentos.

Uma operação de rede também deve respeitar o contexto; em HTTP, por exemplo, a requisição pode ser criada com `http.NewRequestWithContext`.

---

# 12. Idempotência ✅ — conceito introduzido

Falamos sobre o problema de retry em pagamentos.

Cenário:

```text
Nossa aplicação
      ↓
   Gateway
      ↓
pagamento processado ✅
      ↓
resposta perdida / timeout ❌
```

A aplicação pode não saber se o pagamento realmente foi processado.

Se fizer retry sem proteção:

```text
retry
  ↓
Gateway
  ↓
novo processamento
```

podemos gerar uma cobrança duplicada.

## Idempotency Key

Uma chave identifica a operação que não deve produzir efeitos duplicados quando repetida.

```text
Idempotency-Key: abc-123
```

Exemplo:

```text
Tentativa 1 → abc-123 → processa
Tentativa 2 → abc-123 → não cria outro pagamento
Tentativa 3 → abc-123 → não cria outro pagamento
```

### Payment ID x Idempotency Key

`Payment ID`:

> identifica a entidade/pagamento.

`Idempotency Key`:

> identifica a operação que não deve produzir efeitos duplicados quando repetida.

### Situação no projeto

O projeto possui o conceito/hook de `IdempotencyKey`, mas não possui uma camada completa de persistência de idempotência.

Como o projeto não possui banco, não existe uma camada durável para algo como:

```text
idempotency_key → resultado/status
```

Uma evolução possível seria:

```text
IdempotencyStore
```

Primeiro em memória e, depois, uma implementação persistente com PostgreSQL ou Redis.

---

# 13. Resiliência — visão geral

```text
                    GO
                     │
         ┌───────────┴───────────┐
         │                       │
   Concorrência          Sistemas distribuídos
         │                       │
   goroutines                 serviços
   channels                    APIs
   worker pool                 filas
   mutex                       retries
   WaitGroup                   timeout
   select                      circuit breaker
         │                       │
         └───────────┬───────────┘
                     │
                RESILIÊNCIA
```

No fluxo que estudamos:

```text
Retry
  ↓
Backoff
  ↓
Jitter
  ↓
Timeout / Context
  ↓
Circuit Breaker
  ↓
Worker Pool / Bulkhead
  ↓
Idempotência
```

---

# 14. Próximo assunto ⏭️

## Mutex / Race Condition

Ainda precisamos aprofundar:

- `sync.Mutex`
- `sync.RWMutex`
- race condition
- `go test -race`
- quando usar mutex em vez de channel
- estado compartilhado entre goroutines

Depois seguimos para:

```text
Mutex / Race Condition
        ↓
Producer / Consumer
        ↓
Filas
        ↓
Sistemas distribuídos
        ↓
Kafka / RabbitMQ
```

---

# 15. Próximas fases

## APIs

- `net/http`
- REST
- middleware
- JSON
- status codes
- timeouts
- graceful shutdown---> delsligar suavemente.
- gRPC---> No **gRPC**, o framework cria uma "ponte" (chamada de _Stub_). Para o seu código, parece que a função está rodando na sua própria máquina, mas por baixo dos panos o gRPC faz a mágica de enviar o dado pela rede usando **Protobuf** e **HTTP/2**-O gRPC junta os três conceitos que conversamos (os **Interceptors** para middlewares de rede, o **Streaming** para tráfego contínuo e o **Protobuf** para compactar os dados) em um ecossistema focado em altíssima performance para comunicação entre microsserviços.
- protobuf--->Ao contrário do JSON (que é baseado em texto humano), o Protobuf é **binário**. Veja o comparativo de como o mesmo dado trafega
- streaming--->**Server-side Streaming**O cliente envia **uma** requisição e o servidor responde com **várias** mensagens em fluxo.Feed de redes sociais ou download de um arquivo.**Client-side Streaming**O cliente envia **várias** mensagens em fluxo e o servidor responde com **uma** única mensagem no final.Upload de fotos/vídeos ou envio de logs em lote.**Bi-directional Streaming**Tanto cliente quanto servidor enviam **várias** mensagens simultaneamente (fluxo livre).Chat em tempo real ou jogos multiplayer.
- interceptors -->- **Middleware HTTP:** Geralmente manipula `http.ResponseWriter` e `*http.Request`. Ele decide se chama o próximo handler invocando `next.ServeHTTP(w, r)`.

- **Interceptor gRPC:** Manipula contextos (`context.Context`), estruturas de dados tipadas (`interface{}`) e metadados específicos do gRPC, invocando o parâmetro `handler`.

## Banco

- PostgreSQL
- `database/sql`
- connection pool
- transactions
- isolation
- indexes
- N+1
- prepared statements
- context + SQL

## Sistemas distribuídos

- microservices
- service discovery
- timeouts
- retry
- idempotency
- eventual consistency
- distributed transactions
- outbox pattern
- saga
- DLQ
- backpressure
- observability

## Mensageria

- Producer
- Consumer
- Partition
- Offset
- Consumer Group
- At-least-once
- At-most-once
- Exactly-once
- DLQ
- Retry queue
- Ordering

## Docker + Kubernetes

- Pod
- Deployment
- Service
- ConfigMap
- Secret
- Liveness
- Readiness
- Horizontal Pod Autoscaler
- Graceful shutdown

## AWS + CI/CD

```text
Go
 ↓
Docker
 ↓
ECR
 ↓
ECS/EKS
 ↓
ALB
 ↓
CloudWatch
 ↓
CI/CD
```

## Testes em Go

- `testing`
- table-driven tests
- test doubles
- mock
- integration test
- `httptest`
- benchmark
- race detector

## System Design

```text
                 ┌──────────────┐
                 │ API Gateway  │
                 └──────┬───────┘
                        ↓
                 ┌───────────────┐
                 │ Payment API   │
                 └───────┬───────┘
                         ↓
                  ┌─────────────┐
                  │    Queue    │
                  └──────┬──────┘
                         ↓
                  ┌─────────────┐
                  │ Go Workers  │
                  └──────┬──────┘
                         ↓
                ┌──────────────────┐
                │ Circuit Breaker  │
                └────────┬─────────┘
                         ↓
                     PSP Gateway
```

Problemas que esse desenho permite estudar:

- PSP fora do ar
- respostas lentas
- resposta perdida após processamento
- worker morrendo
- mensagem processada duas vezes
- idempotência
- retry
- abertura do circuit breaker
- avalanche de retries
- observabilidade
- escalabilidade
- deploy sem derrubar pagamentos

Race Condition
    ↓
várias goroutines
    ↓
mesmo estado compartilhado
    ↓
acesso simultâneo
    ↓
resultado inconsistente


Mutex
    ↓
protege o estado compartilhado
    ↓
uma goroutine por vez

CHANNEL
   ↓
"Quem tem esse dado?"
"Como envio esse dado?"

MUTEX
   ↓
"Quem pode mexer nesse dado agora?"