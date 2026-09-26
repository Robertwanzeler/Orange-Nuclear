# Orange-Nuclear 

Ambiente e bancada de pesquisa para simulação de redes 5G e controle inteligente em arquitetura **Open RAN (O-RAN)**, integrando o simulador de rede **ns-3** ao controlador **FlexRIC** (Near-RT RIC).

---

## 📌 Visão Geral

O projeto **`orange_nuclear`** reúne ferramentas e cenários de teste para experimentação e validação de algoritmos em redes celulares abertas e desagregadas. A bancada compreende:

* **Simulação de Acesso via Rádio (RAN):** Pilha do **ns-3** com extensões 5G mmWave/NR e agente E2 (`mmwave-LENA-oran`).
* **Controlador Near-RT RIC:** **FlexRIC** da EURECOM com suporte ao protocolo E2AP e Service Models (E2SM-KPM para métricas e E2SM-RC para controle).
* **Casos de Uso e Verticais:** Suporte a fluxos de tráfego de vigilância (eMBB), redes massivas de sensores/IoT (mMTC) e mobilidade veicular (V2X / CARLA).
* **Camada de Inteligência:** Estrutura para treino e avaliação de algoritmos de alocação de recursos (PRB slicing), economia de energia e resolução de conflitos via Aprendizado por Reforço (DRL / MARL).

---

## 📂 Como Instalar e Replicar o Ambiente

Se você está configurando este ambiente em uma máquina nova (Ubuntu 22.04 LTS ou 24.04 LTS), consulte o nosso roteiro passo a passo:

👉 **[Guia de Instalação e Replicação do Ambiente](./Guia%20de%20Instalação%20e%20Replicação.md)**

O guia cobre:
1. Instalação de dependências do sistema (`gcc`, `g++`, `cmake`, `ninja`, `libsctp-dev`, `swig`).
2. Compilação do compilador ASN.1 (`asn1c`) patchado para O-RAN.
3. Compilação e instalação das bibliotecas do **FlexRIC**.
4. Compilação da extensão **ns-3** com conector O-RAN.
5. Configuração de variáveis de ambiente (`LD_LIBRARY_PATH`).

---

## 🚀 Teste Rápido de Comunicação E2

Com o ambiente devidamente instalado, o teste fim a fim de validação é feito em dois terminais:

**1. Terminal 1 — Subir o Near-RT RIC:**
```bash
cd ~/orange_nuclear/flexric/build/examples/ric
./nearRT-RIC
```
**2. Terminal 2 — Disparar o nó simulador ns-3:**
```
cd ~/orange_nuclear/ns-O-RAN-flexric/mmwave-LENA-oran
./ns3 run scratch/scenario-zero-with_parallel_loging
```
