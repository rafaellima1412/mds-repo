
O Floci é um emulador de AWS local gratuito e de código aberto para desenvolvimento, testes e CI.

Ele oferece serviços em forma de AWS em sua máquina sem exigir uma conta na nuvem, um token de autenticação ou portões de recursos pagos. Aponte seu AWS SDK, CLI, Terraform, CDK, OpenTofu ou conjunto de testes em `http://localhost:4566`e mantenha seus fluxos de trabalho existentes.

## Início Rápido

[](https://github.com/floci-io/floci?utm_source=chatgpt.com#quick-start)

A maneira mais rápida de correr Floci é com a [CLI](https://github.com/floci-io/floci-cli) oficial[](https://github.com/floci-io/floci-cli)

```shell
floci start
```

Não há Docker? Cada [liberação](https://github.com/floci-io/floci/releases) também envia um standalone `floci-aws`binário para Linux, macOS e Windows; ver [binário nativo](https://floci.io/floci/getting-started/installation/#native-binary).

Exportar as variáveis de ambiente da AWS:

```shell
eval $(floci env)
```

Use suas ferramentas AWS existentes normalmente:

```shell
aws s3 mb s3://my-bucket

aws dynamodb create-table \
  --table-name demo-table \
  --attribute-definitions AttributeName=pk,AttributeType=S \
  --key-schema AttributeName=pk,KeyType=HASH \
  --billing-mode PAY_PER_REQUEST

aws dynamodb list-tables
```

