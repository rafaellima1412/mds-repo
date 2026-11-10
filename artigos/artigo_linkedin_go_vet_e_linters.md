# `go vet` não verifica se sua struct implementa uma interface — e como elevar o nível do seu código Go com Linters

Uma dúvida comum depois de conhecer a linha abaixo:

```go
var _ input.CreateTripConfigUseCase = (*createTripConfigUseCase)(nil)
```

é: "dá pra automatizar essa checagem com `go vet` ou algum linter, em vez de depender de alguém lembrar de escrever essa linha em cada use case?"

A resposta passa por entender o que cada ferramenta do ecossistema Go realmente faz — porque não é papel de todas elas a mesma coisa.

---

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

---

## Na Prática: Armadilhas comuns detectadas por Linters (`errcheck`)

Embora o compilador cuide dos tipos e o `go vet` pegue erros básicos, ferramentas como o `golangci-lint` e o linter **`errcheck`** capturam falhas de execução e bugs sutis no dia a dia.

### 1. Tratar erros de conversão e parsing
Funções utilitárias como `fmt.Sscanf` retornam a quantidade de itens lidos e um erro. Ignorar esse retorno pode mascarar bugs em parsing de requisições:

```go
// ❌ Problemático: Erro não verificado (errcheck)
fmt.Sscanf(p, "%d", &page)

// ✅ Forma correta e idiomática:
page, err := strconv.Atoi(p)
if err != nil {
    page = 1 // Fallback para valor padrão
}
```

### 2. Cuidados em `defer` (ex: Flush de Logs)
Ao encerrar a aplicação, é comum descarregar o buffer do logger (como Uber Zap) usando `Sync()`. O linter alertará caso o retorno de erro do `Sync()` seja ignorado no `defer`:

```go
// ❌ Alerta do errcheck
defer logger.Log.Sync()

// ✅ Forma explícita e segura em Go
defer func() {
    _ = logger.Log.Sync()
}()
```

### 3. Evitando bugs em Type Assertions e Estruturas de Dados
Ao trabalhar com filas de prioridade (`container/heap`), ignorar o status da conversão de tipo (`ok`) ou chamar `heap.Pop()` mais de uma vez em condicionais altera o estado da estrutura e pode causar inconsistências:

```go
// ❌ Problemático: Remove dois elementos e ignora o status da type assertion
curr := heap.Pop(pq).(*Node)
if heap.Pop(pq) == nil { ... }

// ✅ Seguro e idiomático:
if pq.Len() == 0 {
    return input.TripSearchResponse{}, false
}

nodeVal := heap.Pop(pq)
curr, ok := nodeVal.(*Node)
if !ok || curr == nil {
    return input.TripSearchResponse{}, false
}
```

---

## Padronizando com `.golangci.yml`

Para evitar que erros de parsing, type assertions inseguras ou descaso com retornos de erro cheguem em produção, o ideal é criar um arquivo `.golangci.yml` na raiz do projeto:

```yaml
run:
  concurrency: 4
  timeout: 5m
  issues-exit-code: 1

linters:
  disable-all: true
  enable:
    - errcheck     # Checa retornos de erro não tratados
    - govet        # Análise estática oficial
    - staticcheck  # Boas práticas da comunidade
    - ineffassign  # Variáveis atribuídas mas não utilizadas
    - unused       # Código morto
    - gofmt        # Formatação padrão
    - revive       # Substituto moderno do golint
```

Para rodar localmente e corrigir problemas de formatação automaticamente:

```bash
golangci-lint run --fix
```

---

## Automação e CI com GitHub Actions

O ciclo se fecha automatizando as verificações em cada Pull Request via GitHub Actions com a ação oficial (`.github/workflows/lint.yml`):

```yaml
name: golangci-lint

on:
  push:
    branches: [ main ]
  pull_request:

jobs:
  golangci:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-go@v5
        with:
          go-version-file: 'go.mod'
          cache: false
      - uses: golangci/golangci-lint-action@v6
        with:
          version: v1.60.3
          args: --timeout=5m
```

---

## Resumo

- `go vet` verifica bugs de uso da linguagem — não a conformidade com interfaces (responsabilidade do compilador).
- Linters como `errcheck` e `staticcheck` evitam problemas reais como falhas silenciosas de parsing, type assertions fracas ou `defer` mal configurados.
- Centralizar essas regras em um `.golangci.yml` rodando no GitHub Actions garante a qualidade e manutenibilidade do repositório a longo prazo.