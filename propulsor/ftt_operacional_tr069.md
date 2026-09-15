# FTT Operacional baseado em TR-069 — Arquitetura

## 1. Objetivo

Substituir "OS fechada = sucesso" por "processo funcionou corretamente na primeira vez", cruzando três fontes:

```
FTT Operacional = OS (CRM/ERP) + TR-069 (GenieACS) + Evento de negócio (churn, reabertura, chamado)
```

O CPE nunca decide sozinho se houve FTT. Ele só confirma ou refuta o que o processo alega ter acontecido.

---

## 2. Parâmetros TR-069 a coletar

Baseado no que `coletor_cpe.py` já extrai, mais o que falta para sustentar as regras de FTT.

### 2.1 Já coletado (manter)
| Categoria | Parâmetro TR-098 | Equivalente TR-181 | Uso no FTT |
|---|---|---|---|
| Identificação | `DeviceInfo.SerialNumber` | `Device.DeviceInfo.SerialNumber` | Chave de correlação CPE ↔ matrícula/cliente |
| Identificação | `DeviceInfo.SoftwareVersion` | `Device.DeviceInfo.SoftwareVersion` | Detectar problema por versão de firmware |
| Identificação | `DeviceInfo.UpTime` | `Device.DeviceInfo.UpTime` | Detectar reboots não solicitados |
| WAN | `WANIPConnection.ConnectionStatus` | `Device.IP.Interface.Status` | FTT Instalação (CPE online = pré-requisito) |
| WAN | `ExternalIPAddress` | `IPv4Address.IPAddress` | Confirmar provisionamento correto |
| Óptico | `X_GPON_InterfaceConfig.RXPower` | `Device.Optical.Interface.OpticalSignalLevel` | Gatilho de degradação de sinal |
| Óptico | `X_GPON_InterfaceConfig.TXPower` | `Device.Optical.Interface.TransmitOpticalLevel` | Diagnóstico de causa (CPE vs OLT) |

### 2.2 Precisa adicionar
| Categoria | Parâmetro | Por quê |
|---|---|---|
| Contadores | `WANEthernetInterfaceConfig.Stats.{Bytes,Packets}{Sent,Received}` / `Device.Ethernet.Interface.Stats.*` | Detectar uso real pós-instalação (cliente sem tráfego = onboarding falho, não só "CPE online") |
| Erros de link | `Stats.ErrorsSent`, `Stats.ErrorsReceived`, `DiscardPacketsReceived` | Diferenciar "conectado mas degradado" de "conectado e saudável" |
| Reboot | `ManagementServer.UDPConnectionRequestAddress` + histórico de `Inform events` (`0 BOOTSTRAP`, `1 BOOT`) | Reboot logo após instalação = reincidência técnica |
| Diagnóstico ativo | `IPPingDiagnostics` (RTT, packet loss) | Rodar sob demanda no fechamento da OS, não só passivo |
| Diagnóstico ativo | `DownloadDiagnostics` / `UploadDiagnostics` | Confirmar throughput real contratado, evidência objetiva de "instalação OK" |
| Wi-Fi | `WLANConfiguration.TotalAssociations`, `RadioEnabled` | Reclamação de "wifi não pega" tem evidência técnica própria, separada da WAN |

**Cuidado prático:** ative `IPPingDiagnostics`/`DownloadDiagnostics` apenas sob evento (fechamento de OS, reabertura), nunca em polling constante — CPE doméstico não aguenta diagnóstico ativo contínuo em escala de 10k+ dispositivos.

---

## 3. Modelo de eventos (linha do tempo por cliente)

Cada cliente vira uma sequência de eventos com timestamp, não uma foto única:

```
t0  OS Instalação criada
t1  CPE Inform (0 BOOTSTRAP)          ← TR-069
t2  ConnectionStatus = Connected      ← TR-069
t3  RXPower dentro da faixa esperada  ← TR-069
t4  OS Instalação fechada             ← CRM
t5  [janela de 7 dias]
    - Stats.BytesReceived > limiar mínimo diário   ← TR-069
    - nenhum Inform(0 BOOTSTRAP) inesperado          ← TR-069
    - nenhum chamado aberto                          ← CRM
t6  → FTT Onboarding = 1
```

Se em qualquer ponto entre t1 e t6 houver: chamado técnico, reboot não programado, RXPower fora da faixa por > X horas, ou reabertura de OS → **FTT = 0** e o evento que quebrou a cadeia vira a causa registrada (não um score cego).

---

## 4. Regras de FTT por camada

| Indicador | Condição TR-069 (evidência) | Condição CRM/OS (processo) | FTT = 1 quando |
|---|---|---|---|
| **FTT Instalação** | `ConnectionStatus=Connected` + RXPower dentro da faixa do vendor, dentro de N min após provisionamento | OS de instalação fechada como concluída | Ambas verdadeiras E sem segunda visita em 48h |
| **FTT Onboarding** | Tráfego (`BytesSent/Received`) acima de limiar mínimo em pelo menos 5 dos 7 primeiros dias, sem `Inform(BOOTSTRAP)` inesperado | Cliente ativado no CRM | Sem chamado aberto na janela de 7 dias |
| **FTT Suporte** | Diagnóstico (ping/RXPower) **antes** ruim, diagnóstico **depois** dentro da faixa, sem nova queda em 24h | OS de chamado fechada | Diagnóstico pós-atendimento confirma correção, não só "técnico marcou resolvido" |
| **FTT Reabertura** | CPE ainda com o mesmo sintoma técnico (mesmo RXPower degradado / mesmo erro de interface) do chamado anterior | Nova OS aberta referenciando OS anterior em < 30 dias | Se sintoma técnico é o mesmo → reincidência confirmada (não um chamado novo genérico) |
| **FTT Técnico** | Histórico do CPE (quantas visitas, quantos reboots) por técnico/OS | Atribuição técnico ↔ OS | Agrega os indicadores acima por técnico, não é uma regra nova |

---

## 5. Score e matriz FTT × TR-069 × Churn

Score simples e auditável (evite modelo caixa-preta no início — comece determinístico, migre pra modelo estatístico depois de ter histórico):

```
FTT_score(cliente, período) = 
    (1 se FTT_Instalação) 
  × (1 se FTT_Onboarding)
  × (1 se, para cada chamado no período, FTT_Suporte = 1 e FTT_Reabertura = 0)
```

Resultado: 0 ou 1 por cliente/período — depois agrega em taxa (% FTT) por time, técnico, região, modelo de CPE.

| FTT | Comportamento técnico (TR-069) | Ação |
|---|---|---|
| Alto | Estável (sem reboot, tráfego normal, sinal dentro da faixa) | Baixo risco — manter |
| Alto | Instável (reboots frequentes mas sem chamado) | Risco oculto — cliente não reclama mas vai reclamar. Monitorar proativamente |
| Baixo | Estável após correção | Risco médio — depende de quantas intervenções foram necessárias |
| Baixo | Reincidência (mesmo sintoma técnico repetido) | Risco alto — prioridade de ação preventiva |

Esse segundo caso ("Alto FTT mas instável") é o mais valioso do seu modelo: é o cliente que o CRM não vê como problema mas o TR-069 já denuncia. É aí que o ISP Churn Zero ganha valor real — agir antes da reclamação.

---

## 6. Arquitetura técnica (encaixando no que já existe)

```
CPE (ONU/Router)
   │  TR-069 (Inform, Diagnostics)
   ▼
GenieACS (NBI) ──── já em uso, coletor_cpe.py fala com ele
   │
   ▼
ETL / Coletor de eventos  ← evolução do coletor_cpe.py
   - assina Inform events via NBI ou webhook do GenieACS
   - normaliza TR-098/TR-181 num schema único
   - grava série temporal por device_id (não só snapshot)
   ▼
Banco de eventos (Postgres/TimescaleDB)
   - tabela cpe_events (device_id, timestamp, tipo, payload)
   - tabela os_events (os_id, cliente_id, tipo, timestamp)  ← vem do CRM
   ▼
Engine de regras FTT
   - consome os dois streams
   - aplica as regras da seção 4 por janela de tempo
   - grava resultado em ftt_score (cliente_id, indicador, valor, evidência)
   ▼
Dashboard / API
   - taxa de FTT por time/técnico/modelo de CPE
   - matriz FTT × Churn
```

Ponto de atenção arquitetural: o `device_id` do GenieACS (`OUI-ProductClass-SerialNumber`) precisa de uma tabela de mapeamento explícita para `cliente_id`/matrícula — não dá pra inferir por string matching. Isso é o mesmo tipo de decisão que você já acertou ao mover `matricula` para `UserModel`: aqui o mapeamento CPE↔cliente também merece ser uma entidade própria, não um campo solto.

---

## 7. Ordem de implementação sugerida

1. Mapeamento `device_id` ↔ `cliente_id` (pré-requisito de tudo)
2. Captura de eventos TR-069 em série temporal (evoluir `coletor_cpe.py` de snapshot pra listener/poller contínuo)
3. FTT Instalação (é o mais simples — dado já existe)
4. FTT Onboarding (precisa dos contadores de tráfego, que ainda faltam)
5. FTT Suporte/Reabertura (precisa de diagnóstico ativo sob demanda)
6. Matriz FTT × Churn (só depois de ter 60-90 dias de histórico real)
