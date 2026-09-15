# Projeto Wall-E Cluster AI: Especificação Técnica e Arquitetura

Este documento consolida toda a especificação técnica para a criação de um robô autónomo pet (estilo Wall-E) utilizando um cluster de **10 microcontroladores ESP32-C3** interligados por fios, operando como uma rede neural artificial distribuída (500 neurónios no total, com 50 neurónios simulados por chip).

---

## 1. Visão Geral do Sistema (O que o robô faz?)
O robô opera como um organismo biológico descentralizado. Cada ESP32-C3 atua como um gânglio nervoso especializado numa função sensorial ou motora, processando localmente a informação (Edge AI) e enviando apenas os resultados consolidados para um chip coordenador.

*   **Navegação e Desvio de Obstáculos:** Módulos dedicados processam sensores de distância laser/ultrassom para criar mapas 2D simples em tempo real.
*   **Expressividade Corporal:** Movimentos fluidos de pescoço, braços e olhos em binóculo simulam as emoções do personagem sem a necessidade de um ecrã.
*   **Audição Direcional e Voz:** Reconhecimento da direção de sons (palmas, chamadas) para girar o corpo e reprodução local de áudios robóticos (.wav/.mp3).
*   **Sistema Sensorial de Toque:** Sensores capacitivos sob a carcaça detetam quando o utilizador faz "festas", alterando o estado de humor do robô.

---

## 2. Arquitetura da Rede Neural Distribuída

A rede neural é segmentada fisicamente através de hardware. O processamento matemático (multiplicação de matrizes de pesos e funções de ativação) ocorre de forma paralela.

*   **Camada de Entrada (Nós 1 a 4):** Leitura de sensores brutos (distância, toque, giroscópio) e compressão em ativações intermédias através de 50 neurónios por chip.
*   **Camada Oculta / Especializada (Nós 5 a 9):** Processamento de áudio, visão elementar (seguimento de cores) e cinemática motora.
*   **Camada de Saída (Nó 10 - Maestro):** Recolhe as ativações computadas pelos outros 9 nós e executa a lógica de decisão final do comportamento e humor do Wall-E.

---

## 3. Esquema de Ligação Física (Barramento SPI de Alta Velocidade)

Para simular o equivalente a um "NVLink" em microcontroladores, utiliza-se o barramento **Hardware SPI com DMA (Direct Memory Access)** operando entre 10 MHz e 20 MHz.

### Mapeamento de Pinos da Torre Vertical

*   **Pinos Partilhados (Passam direto por todos os andares da torre):**
    *   **5V / 3.3V:** Linha de alimentação lógica paralela.
    *   **GND:** Referência elétrica comum (obrigatório).
    *   **GPIO 4 (SCK):** Sinal de Clock gerado pelo Mestre.
    *   **GPIO 5 (SDI/MOSI):** Entrada de Dados nos Escravos (Mestre envia).
    *   **GPIO 6 (SDO/MISO):** Saída de Dados dos Escravos (Mestre recebe).

*   **Pinos Exclusivos (Chip Select - CS):**
    *   O pino **GPIO 7** de cada chip escravo (Nós 1 a 9) **deve ser fisicamente cortado ou entortado** no empilhamento vertical para evitar curto-circuito de dados.
    *   Fios externos independentes ligam o GPIO 7 de cada escravo a um pino de controlo dedicado no Mestre (Nó 10):
        *   Mestre GPIO 0  ──> Escravo 1 (CS)
        *   Mestre GPIO 1  ──> Escravo 2 (CS)
        *   Mestre GPIO 2  ──> Escravo 3 (CS)
        *   Mestre GPIO 3  ──> Escravo 4 (CS)
        *   Mestre GPIO 8  ──> Escravo 5 (CS)
        *   Mestre GPIO 9  ──> Escravo 6 (CS)
        *   Mestre GPIO 10 ──> Escravo 7 (CS)
        *   Mestre GPIO 20 ──> Escravo 8 (CS)
        *   Mestre GPIO 21 ──> Escravo 9 (CS)

---

## 4. Estrutura do Hardware e Chassis

O cluster pode ser empilhado verticalmente (formando uma torre compacta) utilizando barras de pinos longas (*stacking headers*) soldadas a uma placa de circuito perfurada universal (Backplane) que serve como base rígida dentro do robô.

### Recomendações de Chassis (Tipo Esteira/Lagarta):
1.  **Chassis Metálicos com Suspensão Independente:** Absorvem impactos mecânicos, protegendo as soldaduras e conexões do barramento SPI contra vibrações crónicas dos motores.
2.  **Chassis com Múltiplos Motores (4x Motores TT):** Fornecem o torque necessário para mover o robô com estabilidade transportando a bateria pesada, os 10 chips e os servos.

---

## 5. Sistema de Alimentação Elétrica (Isolamento de Ruído)

É estritamente proibido alimentar a eletrónica de controlo (ESP32) na mesma linha de potência dos motores para evitar reinicializações por queda de tensão (*brownout*).

*   **Fonte Principal:** Bateria LiPo 2S (7.4V) ou 3S (11.1V) de alta descarga (mínimo 20C).
*   **Linha de Inteligência Artificial (Limpa):** Regulador Buck Step-Down ajustado para **5V (mínimo 3A a 5A)** dedicado exclusivamente à torre de ESP32-C3. Um capacitor eletrolítico de **1000uF / 16V** deve ser soldado na base da torre para filtragem final.
*   **Linha de Potência (Bruta):** Regulador Buck Step-Down independente ajustado para **6V (mínimo 5A a 8A)** dedicado aos servos de movimento (pescoço/braços) e drivers dos motores das esteiras.
*   **Massa Comum (GND):** Todos os GNDs do sistema (IA, Motores e Bateria) devem estar interligados para garantir a referência de sinal comum.

---
