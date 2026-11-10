# 🦀 Ownership, Borrowing e Lifetimes: o trio que faz o Rust funcionar sem drama (de memória)

![[capa.png]]


Ownership, Borrowing e Lifetimes são três conceitos que só fazem sentido completo quando vistos juntos. Cada um responde uma parte da mesma pergunta: **quem pode usar esse dado, como pode usar, e até quando isso é válido?**

Vamos conhecer o trio:

## 🏠 Ownership: o dono possessivo, mas justo



Ownership é aquele amigo que empresta a casa de praia, mas com uma regra clara: **só uma pessoa é dona por vez**. Se você "der" a casa pra outra pessoa, ela é sua agora — e você, o antigo dono, não pode mais nem tocar na maçaneta.

```rust
let s1 = String::from("praia");
let s2 = s1; // s1 "mudou" de dono pra s2

println!("{}", s1); // 🚨 ERRO! s1 não é mais dono de nada
```

Rust não tá sendo chato por maldade. Ele só não quer que duas pessoas "donas" da mesma casa acabem discutindo sobre quem paga a conta de luz (leia-se: double free, use-after-free, e outros pesadelos de C/C++).

**Moral:** em Rust, posse é levada a sério. Só existe UM dono por vez. Ponto final.

## 🤝 Borrowing: "posso usar, mas prometo devolver"


Se Ownership é o dono possessivo, Borrowing é o vizinho educado que pede emprestado a furadeira e devolve no mesmo dia, inteirinha.

```rust
let s1 = String::from("praia");
let tamanho = calcula_tamanho(&s1); // empresta, não rouba!

println!("{} tem {} letras", s1, tamanho); // s1 continua vivinha
```

Aqui a regra do jogo é simples e Rust fiscaliza como uma sogra:

- Você pode ter **várias referências imutáveis** ao mesmo tempo (todo mundo pode *olhar* a furadeira)
- OU **uma única referência mutável** (só uma pessoa pode *usar* a furadeira por vez)
- **Nunca os dois ao mesmo tempo** — porque ninguém quer duas pessoas furando a parede simultaneamente

```rust
let mut furadeira = String::from("Bosch");

let ref1 = &furadeira; // ok, só olhando
let ref2 = &furadeira; // ok, também só olhando
// let ref3 = &mut furadeira; // 🚨 ERRO! não dá pra emprestar E usar ao mesmo tempo
```

**Moral:** pedir emprestado é ótimo, desde que ninguém tente usar a ferramenta enquanto o outro também tá usando.

## ⏳ Lifetimes: o aluguel com data de check-out


E aí entra o terceiro membro do trio: Lifetimes. Ele não define *quem* é o dono nem *como* a coisa é usada — ele garante que **nenhuma referência continue "hospedada" depois que a casa já foi devolvida**.

Pensa num aluguel de temporada: você não pode ficar na casa depois que o contrato venceu, mesmo que ainda tenha a chave na mão.

```rust
fn casa_mais_cara<'a>(c1: &'a str, c2: &'a str) -> &'a str {
    if c1.len() > c2.len() { c1 } else { c2 }
}
```

Aquele `'a` ali não é decoração — é o compilador garantindo, em tempo de compilação, que a referência retornada **não vai "morar" além do tempo em que a casa original existe**. Sem essa garantia, você corre o risco do temido *dangling reference*: uma referência que aponta pra uma casa que já foi demolida.

```rust
let referencia;
{
    let casa = String::from("casa de praia");
    referencia = &casa; // 🚨 ERRO! "casa" é derrubada no fim deste escopo
}
println!("{}", referencia); // referencia apontaria pra uma casa que já não existe
```

**Moral:** toda referência tem prazo de check-out, e o Rust não deixa você ficar hospedado de graça depois que ele venceu.

## 🎬 Resumindo o trio

| Conceito | Pergunta que ele responde | Analogia |
|---|---|---|
| **Ownership** | Quem é o dono disso? | A casa de praia só tem um dono por vez |
| **Borrowing** | Posso usar sem tomar posse? | Pedir a furadeira emprestada ao vizinho |
| **Lifetimes** | Até quando essa referência é válida? | Aluguel de temporada com data de check-out |

No fim das contas, esse trio é o motivo pelo qual Rust consegue te dar segurança de memória **sem garbage collector**. Parece regra chata de sogra, mas é essa disciplina toda que evita que seu programa vire aquele segfault silencioso às 3h da manhã em produção.

E o melhor: o compilador do Rust briga com você em tempo de compilação — não deixa isso virar surpresa desagradável no runtime. 😅

---

💬 E você, já apanhou do borrow checker essa semana? Conta aqui nos comentários qual desses três já te deu mais dor de cabeça (o meu voto vai pro lifetimes, sempre).

#Rust #Programação #SystemsProgramming #DevLife #100DaysOfCode
