---

### Como eu estudei concorrência e resiliência em Go construindo (não lendo)

Duas coisas que eu queria dominar de verdade antes de entrevista de backend sênior pra fintech:

1. Concorrência — goroutines, channels, o pacote context.
    
2. Resiliência — circuit breaker, retry.
    

Decidi não estudar teoria isolada. Decidi construir um sistema pequeno que me obrigasse a usar tudo isso junto, com as peças se encaixando na ordem certa — porque a ordem em que você constrói também ensina.

**1. O gateway instável**

Comecei pelo main.go e criei um gateway de pagamento simulado, propositalmente instável: às vezes devolve 503, às vezes trava em timeout, às vezes recusa o cartão de verdade (erro de negócio). Isso já força uma decisão importante desde o início: nem toda falha é igual, e o sistema precisa reagir diferente pra cada uma.

**2. Circuit breaker, depois retry**

Com o gateway quebrando de propósito, montei o circuit breaker primeiro, e só depois o retry. Essa ordem de construção reflete a ordem de execução real: o circuit breaker envolve o retry, não o contrário. Ele só conta "1 falha" depois que o retry já esgotou as tentativas — evita abrir o circuito por uma falha isolada que o retry resolveria sozinho.

**3. O worker pool**

Só depois de ter gateway + circuit breaker + retry prontos parti pra montagem do pool, com 4 workers (goroutines).

**4. Nasce o context global**

No main, disparo 15 pagamentos. É aí que nasce o context global do batch — com cancelamento programado: depois de 20s, tudo é cancelado em cascata, sem precisar de nenhuma flag manual.

**5. Os 2 channels**

O pool cria dois channels: um (jobs) leva cada pagamento até um worker livre, outro (results) traz o resultado de volta pro main. Dos 4 workers, cada pagamento é pego por apenas 1 — é o próprio channel que arbitra isso, sem eu precisar coordenar nada manualmente.

Um WaitGroup segura as 4 goroutines dos workers até todas terminarem, e nasce também a goroutine produtora, responsável por alimentar o channel de jobs.

**6. O ponto que mais me obrigou a entender concorrência de verdade**

Os 4 workers compartilham o MESMO circuit breaker. Isso muda tudo: agora é memória compartilhada de verdade, com estado (closed / open / half-open) sendo lido e escrito por 4 goroutines ao mesmo tempo. Por isso o circuit breaker precisa de mutex.

E o retry entra por dentro dele exatamente por causa disso: impede que o pool fique martelando um servidor já quebrado, e impede que um worker altere o estado compartilhado enquanto outro ainda está no meio de uma tentativa.

**7. O fim do batch**

Quando os 15 pagamentos terminam, o produtor fecha o channel de jobs, os resultados voltam pelo channel de results, o WaitGroup libera, e o main imprime o resultado final do batch.

Cada execução é diferente — o gateway usa math/rand — então às vezes o circuit breaker chega a abrir de verdade (CLOSED → OPEN → HALF-OPEN → CLOSED), às vezes o retry resolve tudo antes disso. Isso é proposital: dá pra ver ao vivo os dois mecanismos atuando em camadas diferentes.

Repo aberto pra quem quiser rodar, quebrar e sugerir melhoria: [https://github.com/rafaellima1412/payment-resilience](https://github.com/rafaellima1412/payment-resilience)

#golang #backend #fintech #payments #softwareengineering