### Minha sugestão de sequência


**FASE 1 — Go fundamental**
1. `struct`, interfaces e métodos
2. pointers
3. slices e maps
4. errors
5. `defer`
6. `panic/recover`
7. packages e organização
8. generics
9. interfaces idiomáticas
10. `go mod`, `go test`, `go vet`, `go fmt`, `go doc`

**FASE 2 — Concorrência Go**  
11. goroutines  - Eu poderia criar uma goroutine para cada pagamento, mas isso não significa que essa seja uma boa estratégia em escala. O worker pool limita a quantidade de operações concorrentes, evitando sobrecarregar recursos como CPU, memória, conexões, banco ou serviços externos. Os channels permitem distribuir os jobs entre os workers sem precisar necessariamente compartilhar memória protegida por mutex.
	Worker 1 ─┐
	Worker 2 ─┤
	Worker 3 ─┼──► results
	Worker 4 ─┤
	Worker 5 ─┘
	Várias goroutines podem escrever simultaneamente no mesmo channel.

12. channels  - Em Go, **channels são seguros para uso concorrente**
	O runtime do Go sincroniza o acesso ao channel.
	CHANNEL
	   ↓
	Comunicação/sincronização entre goroutines
	   ↓
	Seguro para uso concorrente
	
13. buffered vs unbuffered channels  
14. `select`  
15. `sync.WaitGroup`  - O `WaitGroup` serve para a goroutine principal **esperar outras goroutines terminarem**.
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
		   
	wg.Add(3)     // tenho 3 goroutines para esperar
	wg.Done()     // uma terminou
	wg.Wait()     // espera todas terminarem
    
16. `sync.Mutex` / `RWMutex`  - Quando temos **memória compartilhada** que precisa ser protegida.
	MUTEX
	   ↓
	Proteção de estado/memória compartilhada
	   ↓
	Exclusão mútua
	
17. `sync.Once`  
18. `sync.Pool`  
19. race condition  
20. deadlock  
21. goroutine leak- O que acontece primeiro é: **a goroutine fica bloqueada no envio**.Se nenhum worker estiver pronto para receber:
	Produtor
	   │
	   │ jobs <- pagamento
	   ▼
	 [ BLOQUEADO ]
       │
       │ espera um receiver
       ▼
    Worker recebe
       │
       ▼
    envio continua
se por um buffer:
		Producer
		   │
		   ▼
┌─────────────────┐
│ jobs buffer             │
└────────┬────────┘
         ▼
      Workers
Quando o buffer encher, o produtor volta a bloquear.

Isso é interessante porque começa a aparecer outro conceito importante:
**backpressure**.
**O que acontece quando os 100 slots do buffer estiverem ocupados e os 10 workers também estiverem ocupados?**

Essa pergunta é importante porque conecta:

**channel → blocking → worker pool → backpressure → controle de carga.**

	             Producer
	                 │
	        ┌────────┴────────┐
	        │                 │
	     jobs <-           ctx.Done()
	        │                 │
	        ▼                 ▼
       buffer cheio?        cancelado?
	        │                 │
	     espera             retorna
	 
Context
   +
Channels
   +
Worker Pool
   +
Backpressure
   +
Graceful cancellation
E quando não houver mais trabalho: em alguma aprte do codigo:
 close(jobs)
 jobs
 │
 ├── pagamento 1
 ├── pagamento 2
 ├── pagamento 3
 │
 └── close()
       ↓
   worker termina
   
**Quem envia normalmente é quem fecha o channel.**
O worker não sabe se outro produtor ainda vai enviar alguma coisa.
O produtor/orquestrador é quem sabe:
Agora temos **duas formas de terminar**:
                 Worker
                   │
             ┌─────┴─────┐
             │                    │
          jobs fechado  ctx cancelado
             │                    │
             ▼                  ▼
          return               return
   close(jobs)  -->Ele **não destrói o channel** e não encerra goroutines automaticamente.
  O worker continua recebendo os valores que já estavam no channel. Quando o channel estiver **fechado e vazio**, o `range` termina.
    
      
**FASE 3 — Context**
cancelamento propagado:
batch
  ↓
pool
  ↓
worker
  ↓
retry
  ↓
gateway

Se o batch chegar ao deadline:

             CONTEXT
                │
             CANCEL!
                │
       ┌────────┼────────┐
       ▼        ▼        ▼
    Worker1  Worker2  Worker3
       │        │        │
       ▼        ▼        ▼
     Retry    Retry    Retry
       │        │        │
       ▼        ▼        ▼
    Gateway  Gateway  Gateway

Todo mundo que estiver respeitando aquele contexto pode interromper o trabalho.
Batch: 10 segundos 
Pagamento 1: 2s 
Pagamento 2: 3s 
Pagamento 3: 8s 
Pagamento 4: 15s ← não pode ficar eternamente esperando

ctxBatch
   │
   ├── ctxPagamento1
   ├── ctxPagamento2
   ├── ctxPagamento3
   └── ctxPagamento4
   
cancelou ctxBatch
        ↓
pagamento 1 ─┐
pagamento 2 ─┤
pagamento 3 ─┼──► todos recebem cancelamento
pagamento 4 ─┘

filho pode ser cancelado individualmente; pai pode cancelar todos os filhos.
Eu utilizo context para controlar o ciclo de vida das operações. Existe um contexto do batch e, para cada pagamento, posso derivar um contexto com um timeout específico. Dessa forma, se um pagamento exceder seu limite, ele pode ser cancelado sem necessariamente cancelar os demais. Porém, se o contexto pai do batch for cancelado, o cancelamento é propagado para as operações filhas."


FASE 4 — Resiliência
                    GO
                     │
        ┌────────────┴────────────┐
        │                                              │
	   Concorrência             Sistemas distribuídos
	    │                                               │
	goroutines                                  serviços
	channels                                      APIs
	worker pool                                   filas
	   mutex                                       retries
	   WaitGroup                              timeout
	   select                                   circuit breaker
	    │                                              │
        └────────────┬────────────┘
                     │
                RESILIÊNCIA
                

Retry- Erro permanente ↓ não fazer retry, Erro permanente ↓ não fazer retry
"Tenho mais uma tentativa E ainda tenho tempo?"
  ↓
Backoff - a cada tentativa aumenta o tempo d eespera exponencialmente
O backoff precisa respeitar o deadline do contexto. Se ele crescer indefinidamente, podemos chegar a um ponto em que o tempo de espera do retry já ultrapassa o tempo restante da operação. Por isso normalmente usamos um limite máximo de backoff e verificamos o `context` antes e durante a espera
  ↓
Jitter - adicioantempo amais aleatoriamente para que todos nao voltem d euma vez...
API 1 ── 1.1s ── retry
API 2 ── 1.7s ── retry
API 3 ── 0.9s ── retry
API 4 ── 1.4s ── retry
  ↓
Timeout
  ↓
Circuit Breaker
  ↓
Bulkhead / Worker Pool
  ↓
Idempotência


FASE 5 — APIs
net/http
REST
middleware
JSON
status codes
timeouts
graceful shutdown
gRPC
protobuf
streaming
interceptors

FASE 6 — Banco
PostgreSQL
database/sql
connection pool
transactions
isolation
indexes
N+1
prepared statements
context + SQL

FASE 7 — Sistemas distribuídos
microservices
service discovery
timeouts
retry
idempotency
eventual consistency
distributed transactions
outbox pattern
saga
DLQ
backpressure
observability

FASE 8 — Mensageria
Producer
Consumer
Partition
Offset
Consumer Group
At-least-once
At-most-once
Exactly-once
DLQ
Retry queue
Ordering

FASE 9 — Docker + Kubernetes
Pod
Deployment
Service
ConfigMap
Secret
Liveness
Readiness
Horizontal Pod Autoscaler
Configuração
Graceful shutdown


FASE 10 — AWS + CI/CD
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

FASE 11 — Testes em Go
testing
table-driven tests
test doubles
mock
integration test
httptest
benchmark
race detector

FASE 12 — System Design
                 ┌──────────────┐
                 │ API Gateway  │
                 └──────┬───────┘
                        ↓
                ┌───────────────┐
                │ Payment API   │
                └───────┬───────┘
                        ↓
                 ┌─────────────┐
                 │   Queue     │
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
E aí começam as perguntas realmente interessantes:

- E se o PSP cair?
- E se responder em 10 segundos?
- E se a resposta for perdida?
- E se o worker morrer?
- E se a mensagem for processada duas vezes?
- Como garantir idempotência?
- Onde colocar retry?
- Quando abrir o circuit breaker?
- Como evitar avalanche?
- Como monitorar?
- Como escalar?
- Como fazer deploy sem derrubar pagamentos?