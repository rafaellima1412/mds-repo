# AWS vs Open Source: entendendo o fluxo de CI/CD ponta a ponta

## A dúvida inicial

Tudo começou com uma analogia simples, construída em cima de algo real: um projeto
com `buildspec.yml` na AWS de um lado, e o mesmo conceito usando GitLab do outro.

| Responsabilidade | AWS | GitLab/Open Source |
|---|---|---|
| Git | — | GitLab |
| Orquestração | CodePipeline | GitLab CI/CD |
| Execução | CodeBuild | GitLab Runner |
| Registry | ECR | GitLab Registry |
| Runtime | ECS | Docker |
| Configuração | buildspec.yml | .gitlab-ci.yml |
| Deploy | ECS | Docker |

A pergunta que puxou todo o resto: **será que existem mais ferramentas no mercado
que cobrem essas mesmas responsabilidades?**

## 1. Expandindo o mapa do mercado

A tabela cresceu para incluir GitHub, Azure, GCP e as principais ferramentas open
source de cada categoria:

| Responsabilidade | AWS | GitLab | GitHub | Outros open source |
|---|---|---|---|---|
| Orquestração (CI/CD) | CodePipeline | GitLab CI/CD | GitHub Actions | Jenkins, CircleCI, Argo CD |
| Execução (Runner) | CodeBuild | GitLab Runner | GitHub Runner | Jenkins Agent, Buildkite Agent |
| Registry | ECR | GitLab Registry | GHCR (ghcr.io) | Docker Hub, Harbor |
| Runtime (containers) | ECS / EKS | Docker / Kubernetes | Docker / Kubernetes | Kubernetes, Nomad |
| Provisionamento (IaC) | CloudFormation | — | — | **Terraform**, OpenTofu, Pulumi |

Ponto de virada aqui: entender que faltava uma camada inteira na tabela original —
**provisionamento de infraestrutura** (quem cria a VM/rede/banco, não quem builda
o código).

## 2. GitLab CI/CD → GitHub Actions, na prática

Partindo de um `.gitlab-ci.yml` real (test → build → push → deploy), a tradução
para GitHub Actions revelou diferenças de sintaxe, mas a mesma lógica de fundo:

| Conceito | GitLab CI | GitHub Actions |
|---|---|---|
| Runner próprio | `tags: [projeto]` | `runs-on: self-hosted` |
| Docker-in-docker | `services: docker:27-dind` manual | `docker/build-push-action` resolve por baixo dos panos |
| Login no registry | `docker login` manual | `docker/login-action` + `GITHUB_TOKEN` automático |
| Dependência entre stages | ordem implícita por `stage` | `needs: [job]` explícito |

**Pegadinha real encontrada:** o GHCR (`ghcr.io`) cria pacotes como **privados por
padrão**, mesmo em repositório público — é preciso ajustar a visibilidade
manualmente, senão o `docker pull` no deploy falha por permissão.

## 3. A confusão central: "Terraform substitui o pipeline?"

Aqui vale separar duas coisas com precisão, porque elas se misturaram no meio da
conversa.

**O que já era conhecido, vindo do CloudFormation:** o conceito de ferramenta
declarativa — você descreve o estado desejado, e ela **cria ou atualiza** a
infraestrutura pra bater com isso. Esse princípio é o mesmo nas duas
ferramentas, e não precisou ser reaprendido.

**O que era realmente novo:** não o "cria ou atualiza" em si, mas a **fronteira
de responsabilidade** entre o Terraform e o pipeline de CI/CD. Como as duas
coisas "rodam" e "fazem alguma coisa acontecer", surgiu a hipótese de que
`terraform apply` poderia também disparar testes ou refazer o build da
aplicação — e não, ele nunca faz isso. Assim como uma stack CloudFormation só
reconcilia os recursos que ela declara (nunca re-executa um deploy de aplicação
feito por fora dela), o Terraform tem exatamente essa mesma fronteira:

> **Terraform não builda, não testa, não faz deploy de aplicação. Ele só garante
> que a infraestrutura declarada existe — criando ou atualizando, nunca mais
> que isso.**

| Pergunta que a ferramenta responde | Ferramenta |
|---|---|
| "Esse código funciona?" | pytest |
| "Empacota isso num container" | docker build |
| "Guarda o container em algum lugar" | docker push |
| "Esse servidor/rede existe?" | **Terraform** — só isso |
| "Roda o container no servidor" | docker run |

## 4. De CloudFormation para Terraform: tradução de conceitos

Como CloudFormation já era conhecido, a ponte foi direta:

| CloudFormation | Terraform |
|---|---|
| Template (`.yml`/`.json`) | Arquivos `.tf` |
| `Resources:` | `resource "tipo" "nome" { }` |
| `Parameters:` | `variable "nome" { }` |
| `Outputs:` | `output "nome" { }` |
| Stack | State (`terraform.tfstate`) |
| `aws cloudformation deploy` | `terraform apply` |
| Change Set | `terraform plan` |
| Intrínsecos (`!Ref`, `!GetAtt`) | Referência direta: `aws_instance.exemplo.id` |

**Diferença filosófica importante:** no CloudFormation, o state vive dentro da
AWS, invisível. No Terraform, o state é **seu problema** — fica num arquivo
(`terraform.tfstate`) que, em time, precisa ir para um backend remoto (ex: S3 +
DynamoDB para lock), senão duas pessoas rodando `apply` ao mesmo tempo corrompem
o estado.

## 5. Onde o Terraform roda, de fato

Essa pergunta tinha duas partes que precisavam ser separadas:

- **De onde o comando `terraform apply` executa**: pode ser local, dentro do
  próprio pipeline CI/CD (mais comum em produção — um job a mais no
  `.gitlab-ci.yml`), ou em uma ferramenta dedicada (Terraform Cloud, Atlantis).
- **Onde o state fica guardado**: precisa ser um lugar compartilhado e
  persistente (backend remoto), independente de onde o comando rodou.

Fluxo típico usado como exemplo:

```
Pipeline → job "terraform-apply" roda numa VM temporária/efêmera
        → Terraform cria a infraestrutura (ex: uma VM)
        → script de inicialização (user_data) instala o que for necessário
        → Terraform "vai embora" — não fica residente em lugar nenhum
```

## 6. O que realmente fica em cada lugar (correção de um erro comum)

Depois de tentar juntar tudo numa "stack única" (`gitlab-ci + docker + runner +
terraform` no mesmo servidor), veio a correção mais importante do processo:

**Terraform não fica instalado no servidor final.** Ele passa uma vez (ou a cada
mudança de infra), cria o que precisa, e não é mais necessário depois disso.

| Peça | Papel | Onde vive |
|---|---|---|
| `.gitlab-ci.yml` / `.github/workflows/*.yml` | Define o que será executado | No repositório Git (código, não instalação) |
| GitLab Runner / GitHub Actions Runner | Executa os jobs | Instalado no servidor que vai rodar os jobs |
| Docker | Executa o container da app | Instalado no servidor final |
| Terraform | Cria a infraestrutura (1x) | Roda fora, cria o servidor, **some depois** |

## 7. Terraform na prática: cada peça da sintaxe

| Elemento            | Papel                                            | Exemplo                                               |
| ------------------- | ------------------------------------------------ | ----------------------------------------------------- |
| `resource`          | O que deve existir                               | `resource "aws_instance" "runner_vm" { ... }`         |
| `variable`          | Valores parametrizáveis (inclusive sensíveis)    | `variable "runner_token" { sensitive = true }`        |
| `output`            | O que mostrar depois do apply                    | `output "ip" { value = aws_instance.vm.public_ip }`   |
| `terraform.tfstate` | O que o Terraform *acha* que existe              | Gerado automaticamente, nunca escrito à mão           |
| `terraform plan`    | Simulação — mostra o que vai mudar, sem executar | `+` cria, `~` altera, `-` destrói                     |
| `terraform apply`   | Executa de verdade e atualiza o state            | —                                                     |
| Referência direta   | Como um recurso "conhece" o outro, sem `!Ref`    | `aws_security_group.sg.id` dentro de outro `resource` |

O Terraform monta a ordem de criação sozinho, a partir dessas referências — é o
chamado **grafo de dependências implícito**: não é preciso dizer "crie isso antes
daquilo", ele deduz pela referência entre os blocos.

## 8. Aplicando tudo: caso real do projeto

Repositório de estudo: comparação prática entre uma pipeline AWS e uma
implementação open source (FastAPI + pytest + Docker + GitLab CI/CD).

Dois bugs reais identificados no `.gitlab-ci.yml` durante a revisão:

1. **`2>$null` no `deploy-local`**: como o deploy roda localmente numa máquina
   Windows (runner com shell executor), a sintaxe é válida em PowerShell — mas
   ela só esconde a *mensagem* de erro, não muda o *exit code* do comando. Na
   primeira execução (container ainda não existe), isso pode quebrar o job.
   Correção mais robusta: checar se o container existe antes de tentar
   removê-lo, em vez de só suprimir o erro.

2. **Executor do runner**: jobs que usam `image:` (Docker executor) só funcionam
   se o runner estiver configurado para isso — num runner shell executor puro
   (comum em instalação rápida no Windows), a diretiva `image:` é ignorada.

## 9. Saindo da AWS: alternativas de infraestrutura

Com o objetivo de manter o projeto fiel ao espírito "open source", o mapa de
provedores de nuvem fora da AWS:

| Provedor | Diferencial | Free tier |
|---|---|---|
| **Oracle Cloud** | Free tier permanente de verdade (Always Free) | 2 OCPUs + 12 GB RAM (Ampere A1) — reduzido pela metade em jun/2026 |
| Hetzner | Mais barato, favorito da comunidade self-hosted | Não |
| DigitalOcean | Mais didático, muito tutorial | Não |
| Vultr / Linode | Alternativas equivalentes ao DigitalOcean | Não |

E, fora do modelo "IaaS puro" (VM crua), existe uma camada mais alta de
abstração que vale registrar:

```
IaaS (você gerencia tudo)      → EC2, Compute Instance Oracle, Droplet
Orquestração gerenciada        → ECS, Cloud Run, App Platform
PaaS "git push e pronto"       → Railway, Render, Fly.io
```

Quanto mais alto nessa escala, menos Terraform se escreve — mas também menos se
aprende sobre a "cozinha" por trás, que é justamente o que este projeto de
estudo busca expor.

## 10. Segurança: VPN como camada adicional

Ponto levantado ao notar que o Security Group do Terraform deixava a porta 8000
aberta para `0.0.0.0/0`: existe uma camada de rede privada que evita expor a
aplicação publicamente.

| Necessidade | AWS | Open Source |
|---|---|---|
| VPN ponto-a-ponto | AWS Client VPN | WireGuard, OpenVPN |
| Rede mesh entre máquinas | AWS Transit Gateway | Tailscale, Netmaker |

**WireGuard** é hoje o padrão de fato no mundo open source (mais simples e leve
que OpenVPN). **Tailscale** constrói em cima dele e resolve toda a parte manual
de configuração — instala um agente, conecta numa rede privada virtual, sem abrir
porta nenhuma para a internet.

## Conclusão: o fluxo completo, ponta a ponta

```
1. Código-fonte          → Git (GitLab/GitHub)
2. Orquestração           → .gitlab-ci.yml / workflow.yml (arquivo no repo)
3. Execução dos jobs      → Runner/Agent (instalado no servidor)
4. Testes                 → pytest, dentro de uma imagem/container
5. Build da imagem        → docker build
6. Registry                → GitLab Registry / GHCR / ECR
7. Infraestrutura (1x)    → Terraform (roda fora, cria o servidor, some depois)
8. Runtime                → Docker no servidor final (ou ECS/Kubernetes)
9. Rede segura (opcional) → VPN (WireGuard/Tailscale) entre você e o servidor
```

Cada peça tem uma responsabilidade única e não se sobrepõe às outras — entender
essas fronteiras foi o que resolveu, uma por uma, cada dúvida que surgiu ao
longo do caminho.
