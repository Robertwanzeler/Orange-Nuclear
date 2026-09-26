# Guia de Instalação e Replicação do Ambiente: `orange_nuclear`
---
Este guia contém todos os comandos para instalar do zero o ecossistema **ns-3 + FlexRIC (O-RAN)** em um computador novo com Ubuntu, recriando exatamente a estrutura de diretórios do ambiente **`~/orange_nuclear`**.



## 1. Atualização do Sistema e Dependências
##### Instala os compiladores, gerenciadores de build, ferramentas de depuração e as bibliotecas de protocolo essenciais (como SCTP para a interface E2).

#### 1.1 Atualizar o sistema
``` 
sudo apt update && sudo apt upgrade -y
``` 
#### 1.2 Compiladores, ferramentas de build e debug
``` 
sudo apt install -y \
    build-essential \
    cmake \
    ninja-build \
    git \
    gcc \
    g++ \
    gdb \
    valgrind \
    pkg-config \
    bison \
    flex \
    autoconf \
    automake \
    libtool
``` 
##### 1.3 Bibliotecas de rede (SCTP obrigatório para O-RAN)
``` 
sudo apt install -y \
    libsctp-dev \
    lksctp-tools \
    net-tools \
    iproute2 \
    tcpdump
``` 
#### 1.4 Suporte a Python, SWIG e banco de dados
```
sudo apt install -y \
    python3 \
    python3-dev \
    python3-pip \
    python3-setuptools \
    swig \
    libsqlite3-dev \
    libxml2-dev
```     
## 2. Instalação do Compilador ASN.1 (asn1c) Patchado para O-RAN
##### O padrão E2AP do O-RAN requer a versão do asn1c mantida pela EURECOM para serialização e decodificação das mensagens de controle

#### 2.1 Criar diretório temporário para compilar o asn1c
``` 
mkdir -p ~/build-support
cd ~/build-support
``` 
#### 2.2 Clonar e compilar o asn1c patchado
``` 
git clone https://gitlab.eurecom.fr/oai/asn1c.git
cd asn1c
git checkout vlm_master
autoreconf -iv
./configure
make -j$(nproc)
``` 
#### 2.3 Instalar globalmente no sistema
```
sudo make install
sudo ldconfig

``` 
## 3. Criação da Pasta Raiz orange_nuclear

##### Cria a pasta principal onde ficarão todos os módulos e subpastas do projeto:

``` 
mkdir -p ~/orange_nuclear
cd ~/orange_nuclear

``` 
## 4. Instalação e Compilação do FlexRIC (~/orange_nuclear/flexric)
#####O Near-RT RIC e seus Service Models (KPM, RC, MAC) ficam dentro de ~/orange_nuclear/flexric.
``` 
cd ~/orange_nuclear
``` 
#### 4.1 Clonar o repositório do FlexRIC
```
git clone https://gitlab.eurecom.fr/mosaic5g/flexric.git
cd flexric
``` 
#### 4.2 Criar a pasta de build
```
mkdir -p build && cd build
``` 
#### 4.3 Configurar o build com CMake e Ninja
```
cmake -GNinja \
      -DCMAKE_BUILD_TYPE=Release \
      -DCMAKE_C_FLAGS_RELEASE="-O3" \
      -DCMAKE_CXX_FLAGS_RELEASE="-O3" ..
``` 
#### 4.4 Compilar o Near-RT RIC e Service Models
```
ninja
``` 
#### 4.5 Instalar bibliotecas no sistema e atualizar cache
```
sudo ninja install
sudo ldconfig
``` 
#### 4.6 Criar pasta local de bibliotecas (compatível com a sua estrutura)
```
mkdir -p ~/orange_nuclear/flexric_install_v1/lib
mkdir -p ~/orange_nuclear/flexric_lib
``` 

## 5. Instalação e Compilação do ns-3 (~/orange_nuclear/ns-O-RAN-flexric)
##### Instala o simulador de rede com a extensão mmWave/NR e o conector E2 para se comunicar com o FlexRIC.
``` 
cd ~/orange_nuclear
``` 
#### 5.1 Clonar o ns-O-RAN-flexric recursivamente (traz todos os submódulos)
```
git clone --recursive https://github.com/wines-lab/ns-O-RAN-flexric.git
cd ns-O-RAN-flexric
``` 
#### 5.2 Garantir que os submódulos estejam atualizados
```
git submodule update --init --recursive
``` 
#### 5.3 Entrar na pasta do simulador (mmwave-LENA-oran)
```
cd mmwave-LENA-oran
``` 
#### 5.4 Limpar configurações anteriores
```
./ns3 clean
``` 
#### 5.5 Configurar os módulos necessários para o O-RAN
``` 
./ns3 configure --build-profile=optimized \
                --disable-tests \
                --disable-examples \
                --enable-modules=nr,mmwave,oran-interface,lte,energy
``` 
#### 5.6 Compilar o ns-3
```
./ns3 build -j$(nproc)
``` 

## 6. Configuração das Variáveis de Ambiente no ~/.bashrc
##### Para que qualquer terminal reconheça os caminhos da pasta ~/orange_nuclear e as bibliotecas do FlexRIC:

#### 6.1 Adicionar variáveis ao ~/.bashrc
```
cat << 'EOF' >> ~/.bashrc

# ==========================================
# Configurações do Ambiente orange_nuclear
# ==========================================
export ORANGE_ROOT="$HOME/orange_nuclear"
export LD_LIBRARY_PATH="/usr/local/lib:$ORANGE_ROOT/flexric/build/src/sm/kpm_sm:$ORANGE_ROOT/flexric/build/src/sm/rc_sm:$ORANGE_ROOT/flexric_lib:$LD_LIBRARY_PATH"
EOF
``` 
#### 6.2 Aplicar as variáveis no terminal atual
```
source ~/.bashrc
sudo ldconfig
``` 

## 7. Teste de Validação Fim a Fim (E2 Handshake)
##### Para confirmar que o FlexRIC e o ns-3 estão se comunicando perfeitamente:

####Terminal 1: Iniciar o Near-RT RIC
``` 
cd ~/orange_nuclear/flexric/build/examples/ric
./nearRT-RIC
``` 
#### Terminal 2: Rodar a simulação no ns-3
``` 
cd ~/orange_nuclear/ns-O-RAN-flexric/mmwave-LENA-oran
./ns3 run scratch/scenario-zero-with_parallel_loging
``` 
##### Validação Esperada:
##### No Terminal 1, você deve observar a mensagem de conexão aceita via SCTP e a troca de mensagens de E2 Setup Request (do nó ns-3) e E2 Setup Response (do FlexRIC)

## 8. Comandos Rápidos de Manutenção
##### Liberar porta SCTP presa se o RIC travar em segundo plano:
```
sudo fuser -k 36421/sctp 36422/sctp || true
``` 
#### Recompilar o ns-3 após alterações de código:
``` 
cd ~/orange_nuclear/ns-O-RAN-flexric/mmwave-LENA-oran
./ns3 build -j$(nproc)
``` 
#### Recompilar o FlexRIC:
```
cd ~/orange_nuclear/flexric/build
ninja && sudo ninja install && sudo ldconfig
``` 






















