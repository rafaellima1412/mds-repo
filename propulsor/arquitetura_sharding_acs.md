# Arquitetura de Sharding do ACS — 10k → 100k CPEs

## 1. Princípio

Nem todo componente escala igual ao replicar o ACS. Separar o que **replica linearmente** (cwmp) do que **precisa de dimensionamento próprio** (Mongo, pipeline de eventos, fs) evita a armadilha de "copiei tudo e mesmo assim saturou".

```
Escala linear ao replicar     Precisa de estratégia própria
────────────────────────      ──────────────────────────────
genieacs-cwmp (por shard)     MongoDB (compartilhado)
                               Ingestor/consumidor (fila)
                               genieacs-fs (firmware em massa)
                               Roteamento CPE → shard
```

---

## 2. Divisão em shards

### 2.1 Critério de divisão
Escolha **um** critério fixo, estável no tempo (não deve mudar depois que o CPE já foi provisionado):

| Critério | Vantagem | Cuidado |
|---|---|---|
| Por região/POP | Alinha com topologia de rede real, facilita troubleshooting | Requer que a base de clientes já esteja organizada por região no CRM |
| Por faixa de OLT/PON | Fácil de automatizar no provisionamento (a OLT já sabe onde o CPE está) | Desalinhamento se a rede for reorganizada fisicamente |
| Por leva/data de instalação | Simples de implementar já | Não equilibra carga geograficamente, pode concentrar tráfego |

**Recomendação para o seu caso:** por região/POP — combina com a forma como TR-069 normalmente já é operado (URL de ACS provisionada via opção 43 do DHCP ou parâmetro de fábrica, específica por praça).

### 2.2 Estrutura por shard
Cada shard é uma cópia completa do stack de ACS, exceto o banco:

```
Shard N
├── genieacs-cwmp   (porta 7547, atende só os CPEs do shard)
├── genieacs-nbi    (API northbound, consultas administrativas)
├── genieacs-fs     (firmware — ver seção 5)
└── genieacs-ui     (opcional por shard, ou um único painel central lendo do Mongo compartilhado)

Todos os shards → mesmo MongoDB (cluster dedicado, replicado)
```

---

## 3. Roteamento CPE → shard (sem load balancer dinâmico)

Em vez de um LB decidindo em tempo real, o CPE já nasce sabendo pra qual ACS falar:

- **DHCP Option 43 / TR-069 ACS URL de fábrica**, configurada por praça/OLT no momento do provisionamento — cada região aponta pra um `acs-regiaoX.suaempresa.com.br`
- **Tabela única de roteamento** (fonte de verdade): `device_id` ou `serial_number` → `shard_id`. Vive no seu backend (Propulsor de Vendas ou serviço próprio), não no GenieACS
- **DNS por região**: cada `acs-regiaoX` resolve pro shard correspondente. Simples de trocar se precisar migrar um shard de servidor sem re-provisionar CPE

Isso elimina o problema de CGNAT/`ip_hash` de raiz — não existe decisão de roteamento por IP de origem, é decisão fixa por identidade do CPE.

### 3.1 O que fazer quando um shard satura
1. Provisiona um shard novo (`acs-regiaoX-b`)
2. **Só os CPEs novos** dessa região vão pro shard novo (via config de provisionamento)
3. CPEs existentes continuam no shard antigo até re-provisionamento natural (troca de equipamento, reset de fábrica) — migração forçada só se necessário

---

## 4. MongoDB — dimensionamento próprio

O banco é compartilhado por todos os shards, então recebe a soma da carga — não escala ao replicar o ACS.

- **Replica Set dedicado** (mínimo 3 nós) — não coloque o Mongo na mesma máquina de um shard de ACS
- **Índices por `_id` (device_id) e por campos consultados com frequência** (ex: `_lastInform`, tags usadas nas regras de FTT) — sem isso, consultas da NBI degradam conforme a coleção de devices cresce
- **Monitorar**: IOPS de disco, tempo de resposta de write (cada Inform gera update no documento do device), tamanho da working set em memória
- **Gatilho de ação**: se latência de escrita subir de forma sustentada mesmo com shards de cwmp saudáveis, o gargalo migrou pro banco — hora de otimizar índice, aumentar réplica, ou (último recurso) fazer sharding do próprio MongoDB por região, espelhando os shards de ACS

---

## 5. `genieacs-fs` (firmware) — capacidade própria

Atualização de firmware em massa é rajada, não é tráfego constante como o Inform. Se você fizer uma campanha de atualização em 20k CPEs de uma vez, isso é um pico de banda/I/O que não tem relação com quantos shards de cwmp existem.

- Sirva firmware via **CDN ou proxy de cache** (nginx com cache local) na frente do `genieacs-fs`, não direto do serviço
- Escalone campanhas de atualização por lote (ex: 5% da base por vez) — reduz pico, facilita rollback se uma versão de firmware tiver problema

---

## 6. Ingestor/consumidor — scale-out independente

Esse é o componente que sustenta o FTT (arquitetura anterior). Ele escuta eventos de **todos os shards** — não replica junto com o ACS, precisa da própria estratégia:

```
Shard 1 ─┐
Shard 2 ─┼─► Ingestor (stateless, várias réplicas atrás de LB simples)
Shard N ─┘        │
                   ▼
              Fila (Kafka/RabbitMQ/Redis Streams)
                   │
                   ▼
         Consumidores (N réplicas, processam regras de FTT)
                   │
                   ▼
         Banco de eventos (Postgres/Timescale)
```

- **Ingestor**: pode ter LB comum na frente (sem o problema de sessão do cwmp, porque aqui é HTTP stateless simples — webhook do GenieACS ou polling da NBI)
- **Consumidor**: escala horizontalmente por partição da fila — mais réplicas = mais throughput, sem relação com quantos shards de ACS existem
- **Gatilho de scale-out**: profundidade da fila crescendo (lag do consumidor) — não CPU do ACS

---

## 7. Gatilhos de crescimento (quando fazer o quê)

| Sintoma observado | Onde agir |
|---|---|
| CPU/event-loop lag do `genieacs-cwmp` alto num shard específico | Abrir novo shard naquela região, migrar provisionamento de CPEs novos |
| Latência de escrita no Mongo subindo mesmo com shards saudáveis | Otimizar índice / aumentar réplica / considerar sharding do Mongo |
| Fila do pipeline de eventos com lag crescente | Adicionar réplicas do consumidor (não mexe no ACS) |
| Picos de banda durante campanha de firmware | Cache/CDN na frente do `genieacs-fs`, escalonar por lote |

---

## 8. Evolução sugerida a partir do `docker-compose.yml` atual

1. **Fase atual (10k)**: 1 shard, Mongo no mesmo docker-compose — válido como está, só monitore os gatilhos da seção 7
2. **Fase 30k-50k**: separar Mongo pra fora do compose, em replica set próprio; abrir 2º shard só se o gatilho de CPU/latência disparar, não por antecipação
3. **Fase 50k-100k**: shards por região consolidados, tabela de roteamento CPE→shard formalizada como serviço (não hardcoded), pipeline de eventos com scale-out automático baseado em lag de fila

Não pule direto pra "montar 10 shards" — a ordem acima evita construir capacidade que você ainda não precisa e não vai conseguir validar com carga real.
