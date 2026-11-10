
Para indexar o conteúdo dos MDs (Memoriais Descritivos) no Elasticsearch, você precisa transformar o documento em uma estrutura pesquisável, permitindo localizar informações dentro dos memoriais, e não apenas pelo nome do arquivo.

Guarde esta distinção:

|Operação|Quem é consultado?|
|---|---|
|Cadastrar documento|PostgreSQL + Elasticsearch|
|Alterar documento|PostgreSQL + Elasticsearch|
|Excluir documento|PostgreSQL + Elasticsearch|
|Pesquisar por texto|Elasticsearch|
|Consultar detalhes oficiais|PostgreSQL|

Pense na hierarquia:

| Conceito      | Analogia                                                             |
| ------------- | -------------------------------------------------------------------- |
| Cluster       | Conjunto de servidores                                               |
| Node          | Servidor individual                                                  |
| Index         | Coleção de documentos                                                |
| Shard         | Partição de um índex                                                 |
| Primary Shard | Partição principal                                                   |
| Replica Shard | Cópia da partição principal                                          |
| Lucene<br>    | Motor de indexação e busca utilizado internamente pelo Elasticsearch |

Pense em um sistema de gestão:

|Campo|Tipo|Por quê?|
|---|---|---|
|Nome do produto|text|Pesquisar palavras|
|Descrição|text|Pesquisar conteúdo|
|E-mail|keyword|Comparar e-mail completo|
|CPF|keyword|Comparar identificador|
|Status|keyword|Filtrar por aprovado/reprovado|
|Categoria|keyword|Filtrar por categoria|
Guarde esta frase:

- `text` = quero encontrar palavras dentro de um conteúdo.
- `keyword` = quero encontrar um valor específico, sem dividi-lo em palavras.

Principais operadores:

| Operador   | Significado                    | Equivalente aproximado SQL |
| ---------- | ------------------------------ | -------------------------- |
| `must`     | Obrigatoriamente deve atender  | AND                        |
| `should`   | Preferencialmente deve atender | OR (com ressalvas)         |
| `must_not` | Não pode atender               | NOT                        |
| `filter`   | Filtra sem calcular relevância | WHERE                      |

Exemplo:

Imagine que você queira encontrar documentos que:

- Contenham a palavra "autenticação".
    
- Tenham status "aprovado".
    
- Sejam da categoria "segurança".
    

```JSON
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "conteudo": "autenticação"
          }
        }
      ],
      "filter": [
        {
          "term": {
            "status": "aprovado"
          }
        },
        {
          "term": {
            "categoria": "segurança"
          }
        }
      ]
    }
  }
}
```


## Compound Queries (consultas compostas)

São consultas que combinam outras consultas, inclusive Leaf Queries.
## Comparação

| Leaf                             | Compound                    |
| -------------------------------- | --------------------------- |
| Executa uma condição de pesquisa | Combina condições           |
| Pesquisa diretamente um campo    | Organiza outras consultas   |
| `match`, `term`, `range`         | `bool`, `nested`, `dis_max` |
Para guardar: Leaf procura. Compound combina pesquisas.

O Inverted Index (Índice Invertido)
A ideia é simples: em vez de armazenar apenas quais palavras existem em cada documento, o Elasticsearch cria uma estrutura que informa em quais documentos cada palavra aparece.

O Elasticsearch inverte essa estrutura:

Palavra → documentos onde aparece

## Inverted Index



Agora, quando você pesquisa por `Dell`, o Elasticsearch consulta o índice invertido e encontra diretamente os documentos 1 e 3.

Ele não precisa ler todos os documentos para descobrir onde a palavra aparece.

| palavra  | documentos |
| -------- | ---------- |
| notebook | 1, 2       |
| dell     | 1, 3       |
| inspiron | 1          |
| lenovo   | 2          |
| thinkpad | 2          |
| mouse    | 3          |
| wireless | 3          |

