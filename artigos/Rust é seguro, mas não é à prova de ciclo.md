

3 de out. de 2026 · @Rafaellimahero

Meu servidor em Rust caiu com stack overflow. Eu não mexi em uma linha de código: só editei um arquivo JSON.

Se você também acreditava que "Rust é memory safe" significa "Rust não quebra por memória", este relato é para você. As duas frases parecem iguais, mas não são.

## O projeto

É um jogo de combinações, no estilo "junte dois elementos e descubra um novo", organizado em trilhas, sequências de descobertas que contam uma história.

O backend é Rust com axum, rodando num Raspberry Pi 3 com 1 GB de RAM, servindo os jogadores pela rede e um telão para o evento. Todas as receitas ficam em um `combos.json`, carregado na inicialização.

O Raspberry Pi 3 não está ali por acaso. Ele é o servidor de verdade do jogo e, ao mesmo tempo, o meu laboratório: quero ver na prática até onde o Rust vai em hardware limitado, com pouca memória e CPU modesta, e testar a performance que a linguagem promete fora de uma máquina potente.

## O que aconteceu

Ajustei algumas receitas e, sem perceber, criei um ciclo:

- A + B gerava C
- D + C gerava A

Nenhum dos dois era elemento inicial. Para ter C, era preciso A; para ter A, era preciso C.

Na inicialização, uma função percorria as receitas de trás para frente, recursivamente, para descobrir quais elementos base cada trilha usa. Ela entrou nesse ciclo e nunca mais saiu. Resultado: `thread 'main' has overflowed its stack` e o processo abortado antes de aceitar a primeira conexão.

O detalhe mais irônico estava num comentário da própria função: "não deveria acontecer se os dados passaram na auditoria". Só que essa auditoria não existia.

## Por que o Rust não impediu

A segurança de memória do Rust impede _corrupção_ de memória: acesso a ponteiro inválido, uso depois de liberar, condição de corrida entre threads. Ela não impede _esgotamento_ da pilha. Recursão sem fim estoura a stack em Rust como em qualquer linguagem.

O que muda é a forma de quebrar. O Rust detecta o estouro por uma página de proteção e aborta de maneira controlada, sem comportamento indefinido. É seguro, mas o servidor cai do mesmo jeito.

Quatro detalhes que eu não tinha claros:

- **O compilador não tinha como avisar.** O `rustc` só alerta para recursão incondicional. A minha dependia dos dados: o código estava correto até chegar um JSON com ciclo.
- **Não dá para capturar.** Stack overflow não é `panic`, então `catch_unwind` não segura. O processo simplesmente morre.
- **Não existe garantia de otimização de chamada de cauda.** O LLVM às vezes otimiza em release, mas não há promessa. E com duas chamadas recursivas, nem se aplicaria.
- **A pilha é menor do que parece.** A thread principal costuma ter 8 MB no Linux, mas threads criadas, incluindo as do Tokio, têm 2 MB por padrão.

## A correção, em três camadas

**1. Percorrer o grafo sem recursão.** A função passou a usar uma pilha explícita e um conjunto de visitados. Mesmo que um ciclo escape, ela termina.

```rust
let mut visitados = HashSet::new();
let mut pilha = vec![elemento];
while let Some(atual) = pilha.pop() {
    if !visitados.insert(atual) {
        continue; // já passou por aqui: ciclo ou ramo repetido
    }
    // ... empilha os ingredientes de `atual`
}
```

**2. Validar os dados antes de subir.** Na inicialização, o servidor resolve todas as receitas a partir dos elementos base, de forma iterativa. O que sobra sem resolver é ciclo ou ingrediente órfão. Se houver qualquer erro, o processo sai com a lista exata dos ids problemáticos, em vez de cair com uma mensagem genérica.

**3. Testar antes do deploy.** Um `cargo test` valida o JSON real no meu PC antes de o arquivo ir para o Raspberry Pi. O erro aparece em segundos, não no dia do evento.

De bônus, a validação revelou outros problemas que o ciclo escondia: ingredientes que não existiam em lugar nenhum e elementos da trilha impossíveis de alcançar.

## O que fica

- **Arquivo de configuração é entrada externa**, mesmo quando é você quem escreve. Trate como trataria um formulário vindo da internet.
- **Dado em forma de grafo pede travessia iterativa**, com controle de visitados. Recursão em dado que você não controla é uma aposta.
- **Falhe cedo e com clareza.** É melhor o servidor se recusar a subir dizendo "id 4 depende de A, que depende de C" do que cair com uma mensagem genérica.
- **A garantia de uma linguagem tem um escopo.** Rust elimina uma classe inteira de bugs, e isso é enorme. Mas lógica e dados continuam sendo responsabilidade nossa.

E você, já foi pego por uma garantia que cobria menos do que você imaginava?

#Rust #RustLang #DesenvolvimentoDeSoftware #Backend #RaspberryPi