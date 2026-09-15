CPEs (ONU/Router)
   │  TR-069 (porta 7547)
   ▼
NGINX (LB) ──┬── GenieACS instância 1
             └── GenieACS instância 2
                       │
                       ▼
                  MongoDB (compartilhado)
                       ▲
                       │  polling NBI (delta por _lastInform)
                  ingestor  ← cursor de polling salvo no Redis
                       │
                       ▼
                Kafka (tópico device-events)
                       │
                       ▼
                  consumer
                   ├──► PostgreSQL (histórico / funil de sucesso)
                   ├──► Redis (cache do último estado de cada CPE)
                   └──► Kafka (tópico alertas-ftt, só quando detecta
                        problema: FTT quebrado, degradação, reincidência)
                                   │
                                   ▼
                        n8n (Kafka Trigger no tópico alertas-ftt)
                                   │
                                   ▼
                        Slack / e-mail / ticket no CRM