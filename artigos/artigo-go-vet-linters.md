# `go vet` não verifica se sua struct implementa uma interface — e isso é proposital

Uma dúvida comum depois de conhecer a linha abaixo:

```go
var _ input.CreateTripConfigUseCase = (*createTripConfigUseCase)(nil)
```

é: "dá pra automatizar essa checagem com `go vet` ou algum linter, em vez de depender de alguém lembrar de escrever essa linha em cada use case?"

A resposta passa por entender o que cada ferramenta do ecossistema Go realmente faz — porque não é papel de todas elas a mesma coisa.

## O que `go vet` verifica (e o que não verifica)

`go vet` é a ferramenta estática oficial, distribuída junto com o toolchain do Go:

```bash
go vet ./...
```

Ela cobre um conjunto específico de erros estruturais e de uso incorreto da linguagem:

- Formatação incompatível em `Printf`/`Sprintf` (argumento não bate com o verbo)
- Comparações redundantes
- Cópia indevida de structs que contêm `sync.Mutex`
- Tags de struct malformadas
- Uso incorreto de `context.Context` como campo de struct

**Conformidade de interface não está nessa lista.** Isso não é uma lacuna do `go vet` — é o próprio compilador quem faz essa verificação, no momento exato em que um valor concreto é atribuído a uma variável de tipo interface. É por isso que a trava `var _ Interface = (*Struct)(nil)` funciona sem precisar de nenhuma ferramenta externa: ela só força essa checagem do compilador a acontecer mais cedo, no lugar onde a struct é definida.

## Forçando o padrão via CI: analisador customizado

Se o objetivo é garantir, de forma automática em pipeline, que **todo** construtor `New*UseCase` siga uma convenção — por exemplo, sempre retornar a interface e não o tipo concreto — isso está fora do escopo do `go vet` padrão. A ferramenta certa para isso é escrever um analisador próprio usando `golang.org/x/tools/go/analysis`, o mesmo framework sobre o qual o `go vet` é construído.

Estrutura mínima de um analyzer:

```go
package usecasereturn

import (
    "go/ast"
    "strings"

    "golang.org/x/tools/go/analysis"
)

var Analyzer = &analysis.Analyzer{
    Name: "usecasereturn",
    Doc:  "verifica se construtores New*UseCase retornam interface, não tipo concreto",
    Run:  run,
}

func run(pass *analysis.Pass) (interface{}, error) {
    for _, file := range pass.Files {
        ast.Inspect(file, func(n ast.Node) bool {
            fn, ok := n.(*ast.FuncDecl)
            if !ok || !strings.HasPrefix(fn.Name.Name, "New") {
                return true
            }
            if fn.Type.Results == nil || len(fn.Type.Results.List) == 0 {
                return true
            }
            if _, isPointer := fn.Type.Results.List[0].Type.(*ast.StarExpr); isPointer {
                pass.Reportf(fn.Pos(), "%s retorna tipo concreto (*T); prefira retornar a interface", fn.Name.Name)
            }
            return true
        })
    }
    return nil, nil
}
```

Esse analyzer pode ser plugado num binário próprio (via `multichecker`) e rodado como parte do CI, falhando o build quando a convenção não for seguida. É mais trabalho de configuração inicial, mas escala melhor que revisão manual em times grandes.

## Linters prontos que já cobrem parte do problema

Antes de escrever algo do zero, vale considerar ferramentas já existentes:

| Ferramenta | Função |
|---|---|
| `staticcheck` | Conjunto amplo de checagens estáticas, incluindo vários padrões estruturais além do que `go vet` cobre |
| `golangci-lint` | Executor que agrega dezenas de linters (`govet`, `staticcheck`, `unused`, `ireturn`, entre outros) numa única passada, configurável via `.golangci.yml` |
| `ireturn` | Sinaliza funções que retornam interfaces — útil para o cenário inverso ao discutido aqui, caso o time queira limitar em quais camadas isso é permitido |

Execução unificada, comum na maioria dos projetos Go atuais:

```bash
go install github.com/golangci-lint/golangci-lint/cmd/golangci-lint@latest
golangci-lint run ./...
```

O conjunto de linters ativos, exceções e severidade fica centralizado no `.golangci.yml`, versionado junto com o repositório.

## Resumo

- `go vet` verifica bugs de uso da linguagem — não conformidade com interface. Essa checagem é do compilador, disparada no ponto de uso ou forçada antecipadamente com `var _ Interface = (*Struct)(nil)`.
- Garantir uma convenção de projeto (como "construtores sempre retornam interface") em escala, via CI, exige um analisador customizado com `golang.org/x/tools/go/analysis` — o mesmo mecanismo por trás do `go vet`.
- Antes de construir algo do zero, vale avaliar se `staticcheck` ou `golangci-lint`, com a configuração certa, já cobrem parte da necessidade.

Automatizar esse tipo de checagem tem retorno maior quanto maior o time: o custo de configurar um analyzer ou um `.golangci.yml` bem ajustado se paga rápido quando substitui revisão manual repetitiva em cada pull request.
