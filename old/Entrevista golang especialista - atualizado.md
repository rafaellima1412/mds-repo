# Estudo Go — Payment Resilience

Material de estudo baseado no projeto `payment-resilience` e nos conceitos que fomos construindo durante a conversa.

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


# 14. Formas de evitar concorrência de dados

Existem três abordagens principais.

## 14.1 Apenas leitura — não escrever na variável

Várias goroutines podem ler uma variável compartilhada. O problema aparece quando existe escrita concorrente.

```go
package main

import (
	"fmt"
	"sync"
)

var config = map[string]string{
	"host": "localhost",
	"port": "5432",
}

func main() {
	var wg sync.WaitGroup

	wg.Add(2)

	go func() {
		defer wg.Done()
		fmt.Println("Goroutine 1:", config["host"])
	}()

	go func() {
		defer wg.Done()
		fmt.Println("Goroutine 2:", config["port"])
	}()

	wg.Wait()
}
```

Se houver escrita concorrente:

```go
config["port"] = "5432" // pode causar data race
```

A regra é:

```text
Somente leitura concorrente → geralmente seguro
Leitura + escrita concorrente → precisa de proteção
```

## 14.2 Não compartilhar a variável

Outra forma é evitar que várias goroutines acessem a mesma variável.

```go
package main

import (
	"fmt"
	"sync"
)

func worker(id int, wg *sync.WaitGroup) {
	defer wg.Done()

	resultado := 0

	resultado += id

	fmt.Println("Goroutine:", id, "Resultado:", resultado)
}

func main() {
	var wg sync.WaitGroup

	wg.Add(2)

	go worker(1, &wg)
	go worker(2, &wg)

	wg.Wait()
}
```

Cada goroutine possui seu próprio `resultado`.

```text
Sem variável compartilhada → sem race
```

## 14.3 Compartilhar usando exclusão mútua

Quando a variável precisa ser compartilhada, podemos permitir que apenas uma goroutine por vez acesse a região crítica.

```go
package main

import (
	"fmt"
	"sync"
)

var contador int
var mu sync.Mutex

func incrementar(wg *sync.WaitGroup) {
	defer wg.Done()

	mu.Lock()
	defer mu.Unlock()

	contador++
}

func main() {
	var wg sync.WaitGroup

	for i := 0; i < 100; i++ {
		wg.Add(1)
		go incrementar(&wg)
	}

	wg.Wait()

	fmt.Println("Contador:", contador)
}
```

Fluxo:

```text
Goroutine 1 → Lock → contador++ → Unlock
Goroutine 2 → espera
Goroutine 3 → espera
                  ↓
             Lock liberado
                  ↓
Goroutine 2 → Lock → contador++ → Unlock
```

> `Mutex` não elimina a concorrência. Ele controla o acesso ao estado compartilhado.

---

# 15. WaitGroup — sincronização de goroutines

O `WaitGroup` não torna as goroutines mais eficientes e não protege variáveis.

Ele serve para a goroutine principal **esperar outras goroutines terminarem**.

```go
var wg sync.WaitGroup

for i := 0; i < 3; i++ {
	wg.Add(1)

	go func() {
		defer wg.Done()
		processar()
	}()
}

wg.Wait()
```

A lógica é:

```text
Add(1)
   ↓
goroutine inicia
   ↓
Done()
   ↓
contador -1
   ↓
contador == 0
   ↓
Wait() libera
```

Métodos principais:

```go
wg.Add(n) // registra goroutines
wg.Done() // sinaliza que uma terminou
wg.Wait() // espera todas terminarem
```

> `WaitGroup` sincroniza término. `Mutex` protege estado compartilhado. `Channel` faz comunicação entre goroutines.

---

# 16. Channel x Mutex x WaitGroup

```text
Channel
   ↓
Comunicação entre goroutines

Mutex
   ↓
Proteção de estado compartilhado

WaitGroup
   ↓
Espera pelo término de goroutines
```

Eles podem ser usados juntos:

```text
              jobs
                ↓
             channel
                ↓
       ┌────────┼────────┐
       ↓        ↓        ↓
      W1       W2       W3
       │        │        │
       └────────┼────────┘
                ↓
            WaitGroup
                ↓
          todos terminaram
```

---

# 17. Concorrência controlada — Worker Pool

Criar uma goroutine para cada trabalho nem sempre é a melhor opção.

```text
10.000 jobs
     ↓
10.000 goroutines
```

Uma alternativa é limitar a quantidade de workers:

```text
10.000 jobs
     ↓
   jobs chan
     ↓
┌────┬────┬────┬────┐
│ W1 │ W2 │ W3 │ W4 │
└────┴────┴────┴────┘
```

Assim:

```text
10.000 jobs ≠ 10.000 goroutines

10.000 jobs → 10 workers
```

O Worker Pool serve principalmente para **controlar concorrência e recursos**.

---

# 18. `go test -race`

Go possui um detector de race condition.

```bash
go test -race ./...
```

Ele ajuda a encontrar acessos concorrentes incorretos à memória compartilhada.

Exemplo de problema:

```go
var contador int

go func() {
	contador++
}()

go func() {
	contador++
}()
```

O detector pode identificar que existe acesso concorrente sem sincronização.

---

# 19. go doc 

São ferramentas de documentação com objetivos diferentes.

## `go doc`

Documenta o código Go:

```bash
go doc fmt.Println
```

É usado para consultar:

- pacotes
- funções
- structs
- interfaces
- métodos
- tipos



Resumo:

```text
go doc       → documentação do código Go
```

Os dois podem coexistir no mesmo projeto.

---

# 20. Atualização da ordem de estudo

Depois dos conceitos de concorrência, a sequência fica:

```text
Mutex / Race Condition
        ↓
Producer / Consumer
        ↓
Filas
        ↓
APIs
        ↓
Banco
        ↓
Sistemas distribuídos
        ↓
Kafka / RabbitMQ
        ↓
Docker / Kubernetes
        ↓
AWS / CI/CD
        ↓
Testes
        ↓
System Design
```


# 21. Próximo assunto ⏭️

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

# 22. Próximas fases

## APIs

- `net/http`
- REST
- middleware
- JSON
- status codes
- timeouts
- graceful shutdown
- gRPC
- protobuf
- streaming
- interceptors

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
# REGRA DE OURO DA ENTREVISTA

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

# BLOCO EXTRA — OWASP / SEGURANÇA

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

## SQL Injection

O problema acontece quando dados fornecidos pelo usuário acabam sendo interpretados como parte do SQL.

Eu evito construir SQL concatenando entrada do usuário.

Não faria:

```python
query = f"SELECT * FROM users WHERE name = '{name}'"
```

Prefiro utilizar ORM, como SQLAlchemy, ou queries parametrizadas.

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

## XSS — Cross-Site Scripting

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

## Broken Access Control

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

## Authentication x Authorization

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

## CSRF ( Ele impede o envio do cookie em requisições cross-site (de outros sites)

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

## SSRF

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

## Cryptographic Failures

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

## Security Misconfiguration

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

## Vulnerable and Outdated Components

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

## Security Logging and Monitoring

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

## Input Validation

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

No FastAPI, Pydantic ajuda bastante na validação estrutural dos dados recebidos.

Mas validação de formato não substitui autorização ou regra de negócio.

---

## Secrets

Não colocaria:

```python
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

## Princípio do menor privilégio

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

# Como eu penso em segurança

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

# Resposta curta se perguntarem sobre OWASP

> “Eu conheço os principais riscos do OWASP e tento aplicar isso no dia a dia. Penso bastante em controle de acesso, autenticação, SQL Injection, XSS, validação de entrada, secrets, configuração segura e dependências vulneráveis. Não vejo OWASP como uma lista para decorar; uso como referência para pensar nos riscos da aplicação.”

# MAPA RÁPIDO — OWASP (ô-uasp)

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