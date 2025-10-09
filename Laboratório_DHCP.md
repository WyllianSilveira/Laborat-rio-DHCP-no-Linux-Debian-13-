# 🛠️ Laboratório DHCP no Linux (Debian 13)

# 📘 Descrição Geral

Este laboratório tem como objetivo demonstrar a configuração completa de um servidor **DHCP**  
em um ambiente Linux Debian 13, utilizando máquinas virtuais no VirtualBox para simular uma  
rede LAN interna.

O cenário foi projetado para representar uma topologia de rede realista, onde o Debian 13  
virtualizado atua como servidor DHCP responsável por fornecer configurações automáticas de  
endereçamento IP para clientes conectados à rede interna.

O ambiente foi implementado sobre um host **Windows 11**, que utiliza uma interface de rede  
física Intel(R) Wireless-AC 9560 para acesso à Internet e comunicação com o roteador doméstico,  
cujo endereço `192.168.0.1` também fornece serviço DHCP na rede externa.

Dentro do VirtualBox, foi criada uma interface em modo **Bridge** (`enp0s3`) para comunicação  
entre o host e o servidor Debian, e uma interface **interna** (`enp0s8`) que conecta o servidor  
Debian ao cliente Windows 10 virtualizado, formando uma sub-rede isolada Classe A (`10.200.0.0/24`)  
para os testes do serviço DHCP.

Essa arquitetura permite validar o funcionamento do serviço em um ambiente controlado,  
simulando a atuação de um servidor DHCP real em redes corporativas.

---

# 🧩 Objetivos do Projeto

- 🖥️⚙️ Instalar e configurar o serviço **DHCP** no Debian 13.  
- 🌐📡 Fornecer configurações automáticas de IP para clientes conectados à rede interna  
  (sub-rede `10.200.0.0/24`).  
- ✅💻 Validar o funcionamento do servidor DHCP através de uma estação cliente Windows 10  
  virtualizada, garantindo que o cliente obtenha corretamente as configurações de rede.


# 🧾 DOCUMENTAÇÃO – CONFIGURAÇÃO DO SERVIDOR DHCP (ISC-DHCP-SERVER) NO DEBIAN

# 🔹 1. Instalação e acesso ao diretório padrão de configuração

Antes de realizar as configurações, é necessário instalar o pacote do servidor DHCP no Debian.
O serviço utilizado neste laboratório é o ISC-DHCP-SERVER, um dos mais usados em ambientes Linux.

Execute os comandos abaixo para realizar a instalação:
````bash
apt update
apt install isc-dhcp-server -y
````
<br><br>
# 🔹 2. Acesso ao diretório padrão de configuração

```bash
cd /etc/default
ls
```

![Acesso ao diretório padrão](imagem/imagem.png)


O primeiro passo é navegar até o diretório `/etc/default`, onde estão localizados os arquivos de  
configuração padrão de diversos serviços do sistema.

Neste diretório, encontramos o arquivo `isc-dhcp-server`, responsável por armazenar as definições  
de inicialização e as interfaces de rede que o serviço **DHCP** utilizará.

É neste arquivo que especificamos **qual placa de rede o serviço vai usar para distribuir endereços IP**.
<br><br>

# 🔹 3. Editando o arquivo padrão do serviço

📸 Imagem:  
![Configuração padrão do serviço DHCP](imagem/arquivo_configuração_padrão_placa_dhcp.png)

Abrimos o arquivo de configuração padrão do serviço **DHCP**:

```bash
nano /etc/default/isc-dhcp-server
```
Dentro dele, configuramos a interface de rede responsável por responder aos pedidos DHCP:

INTERFACESv4="enp0s8"  

INTERFACESv6=""

INTERFACESv4: define a interface de rede IPv4 (no caso, enp0s8).

INTERFACESv6: vazio, pois não estamos usando DHCPv6.

💡 Essa configuração garante que o serviço ISC-DHCP-Server escute e distribua IPs apenas pela
interface enp0s8, evitando conflitos com outras interfaces do sistema.
<br><br><br>

# 🔹 4. Configurando a interface de rede no sistema

📸 Imagem:  
![Configuração da interface de rede](imagem/configurando_interface_do_dhcp.png)

Agora, configuramos o arquivo `/etc/network/interfaces` para definir os parâmetros de rede da interface  
usada pelo DHCP.

```bash
nano /etc/network/interfaces
```
Adicionamos a configuração manual da interface enp0s8, que será a interface servidora DHCP (rede interna):
# Interface DHCP
```bash
auto enp0s8
iface enp0s8 inet static
address 10.200.0.1
netmask 255.255.0.0
network 10.200.0.0
broadcast 10.200.0.255
```
🔸 Explicação dos parâmetros:

address: IP fixo do servidor DHCP (gateway da rede interna).

netmask: máscara de rede.

network: endereço da rede.

broadcast: endereço de broadcast da sub-rede.

💡 Essa interface servirá como gateway e ponto de distribuição de endereços IP para os clientes DHCP.


<br><br>
# 🔹 5. Configurando o arquivo principal do serviço DHCP

📸 **Imagem:**  
![Acesso ao diretório /etc/dhcp](imagem/Caminho_do_diretorio_configuração_dhcp.png)

Após definir a interface de rede do servidor, acessamos o diretório **/etc/dhcp**, onde estão armazenados os arquivos principais de configuração do serviço **ISC-DHCP-SERVER**.

# 🔸 Comandos executados

```bash
cd /etc/dhcp
ls
````
📘 Explicação

cd /etc/dhcp → acessa o diretório onde ficam os arquivos de configuração do servidor DHCP.

Na listagem, observamos:

dhcpd.conf → arquivo principal de configuração do servidor DHCP (versão IPv4)

dhcpd6.conf → configuração para DHCPv6 (endereçamento IPv6)

dhclient-exit-hooks.d → diretório com scripts de saída executados pelo cliente DHCP

O arquivo que será editado e configurado neste laboratório é o dhcpd.conf, utilizado pelo serviço para definir o comportamento da distribuição de endereços IP, parâmetros de rede e opções entregues aos clientes.


<br><br>
# 🔹 6. Editando o arquivo de configuração principal (dhcpd.conf)

📸 Imagem:<br>
![Arquivo de configuração DHCP](imagem/arquivo_com_configuração_dhcp.png)

🔸 Comando utilizado
````bash
nano /etc/dhcp/dhcpd.conf
````

Após editar, o conteúdo final do arquivo ficou assim:
````bash
ddns-update-style none;
option domain-name "";
option domain-name-servers 8.8.8.8;
default-lease-time 600;
max-lease-time 7200;
authoritative;

subnet 10.200.0.0 netmask 255.255.255.0 {
  range 10.200.0.100 10.200.0.200;
  option routers 10.200.0.1;
  option broadcast-address 10.200.0.255;
}

````

# 🧩 Explicação das Diretivas Globais

| Diretiva | Descrição |
|-----------|------------|
| `ddns-update-style none;` | Desativa atualizações automáticas de DNS dinâmico pelo servidor DHCP. |
| `option domain-name "";` | Define o nome de domínio a ser informado aos clientes (vazio neste lab). |
| `option domain-name-servers 8.8.8.8;` | Informa o servidor DNS que será usado pelos clientes (DNS público do Google). |
| `default-lease-time 600;` | Tempo padrão de concessão de IP (**10 minutos = 600s**). |
| `max-lease-time 7200;` | Tempo máximo de concessão (**2 horas = 7200s**). |
| `authoritative;` | Define que este servidor é o principal e autoritativo para a rede — evita conflitos caso exista outro servidor DHCP. |

---

# 🧩 Explicação do Bloco `subnet`

O bloco `subnet` define as configurações específicas para a rede interna controlada pelo servidor DHCP.

| Diretiva | Função |
|-----------|--------|
| `subnet 10.200.0.0 netmask 255.255.255.0 { }` | Define a sub-rede atendida pelo servidor DHCP. |
| `range 10.200.0.100 10.200.0.200;` | Faixa de endereços IP que o servidor poderá distribuir aos clientes. |
| `option routers 10.200.0.1;` | Define o **gateway padrão** (neste caso, o próprio servidor DHCP). |
| `option broadcast-address 10.200.0.255;` | Endereço de broadcast da sub-rede, usado para comunicação em grupo. |

---

# 💡 Observação Importante

- A rede utilizada (`10.200.0.0/24`) é uma **sub-rede privada de classe A**, criada exclusivamente para este laboratório.  
- O servidor DHCP está configurado com o **IP 10.200.0.1**, atuando como **gateway e ponto de distribuição** para os clientes conectados à interface interna `enp0s8`.

---

📘 **Resumo:**
Este bloco de configuração garante que todos os dispositivos conectados à rede interna recebam automaticamente:
- Endereço IP dentro do intervalo definido;
- Gateway de saída;
- Servidor DNS funcional.

Isso forma a base de um **ambiente de rede automatizado e gerenciável**, essencial em infraestruturas corporativas ou laboratoriais.

<br><br>
## 🔹 7. Inicializando e Verificando o Serviço DHCP

Após finalizar a configuração do arquivo `dhcpd.conf`, o próximo passo é **inicializar o serviço DHCP** e confirmar se ele foi executado corretamente.

📸 Imagem:<br>
![Verificando status do serviço DHCP](imagem/status_serviço_dhcp.png)

### 🧠 O que está sendo feito

1. **Inicia o serviço DHCP:**
   - O comando `service isc-dhcp-server start` ativa o daemon responsável por distribuir endereços IP na rede interna.

2. **Verifica o status do serviço:**
   - `service isc-dhcp-server status` mostra o estado atual do serviço, o tempo de execução e possíveis erros de inicialização.

3. **Confirma que o serviço está ativo (running):**
   - A linha `Active: active (running)` indica que o servidor DHCP foi iniciado com sucesso e está escutando na interface configurada (`enp0s8`).

---

### 📜 Script utilizado

```bash
service isc-dhcp-server start
service isc-dhcp-server status
````

💬 Explicação técnica

O serviço isc-dhcp-server foi iniciado com sucesso e está distribuindo endereços IP conforme definido em /etc/dhcp/dhcpd.conf.
A saída indica:

✅ Loaded: O serviço foi carregado corretamente no sistema.

✅ Active (running): Está em execução e operacional.

✅ Process: Demonstra o PID e o comando de inicialização (dhcpd -4 -q -cf /etc/dhcp/dhcpd.conf enp0s8).

Essa etapa confirma que o servidor DHCP está funcional e pronto para atender as requisições dos clientes conectados à rede interna.


<br><br>
### 🔹8 Verificando a Configuração Automática do Cliente na Rede

📸 **Imagem:**  
![Verificando a configuração automática do cliente](imagem/Imagem_outro_pc_terminal_Linux_configuração_rede_automatica.png)

🔍 **Explicação rápida:**  
A imagem mostra a execução do comando `ip a` no terminal de um cliente conectado à rede interna controlada pelo servidor DHCP Debian. O cliente obteve um endereço IP automaticamente.

- **Endereço IP obtido**: O cliente recebeu o endereço IP `10.200.0.100`, dentro da faixa configurada no servidor DHCP (`range 10.200.0.100 10.200.0.200`).
- **Máscara de rede**: A máscara de rede é `255.255.255.0`, correspondente à sub-rede `10.200.0.0/24`.
- **Status**: O comando `ip a` mostra que a interface está **UP** e funcionando corretamente.

💡 **Importante:**  
O cliente recebeu um endereço IP automaticamente do servidor DHCP (`10.200.0.100`), confirmando que o servidor Debian está configurado corretamente para distribuir IPs na rede interna.


<br><br>
## 🔹9 Verificando os Arquivos de Concessão do DHCP

📸 **Imagem:**  
![Verificando arquivos de concessão do DHCP](imagem/arquivo_maquinas_conectadas_no_server_dhcp.png)

🔍 **Explicação rápida:**  
Na imagem, é mostrado o comando `ls` sendo executado no diretório `/var/lib/dhcp`. Esse diretório armazena os arquivos de concessão do servidor DHCP, que controlam os endereços IP atribuídos aos clientes.

- **Arquivo `dhcpd.leases`**: O arquivo `dhcpd.leases` é onde o servidor DHCP armazena as informações de concessão de IP, como o endereço atribuído, o tempo de concessão e o cliente que recebeu o IP.
  - Esse arquivo é fundamental para o funcionamento do serviço DHCP, pois ele mantém um histórico das concessões de IP feitas aos clientes, o que permite a renovação de IPs e o gerenciamento de conflitos de IP.
  
- **Arquivo `dhcpd.leases~`**: O arquivo com o sufixo `~` é um arquivo de backup, geralmente gerado automaticamente pelo sistema ou por editores de texto.

💡 **Importante:**  
Os arquivos de concessão, como `dhcpd.leases`, são essenciais para monitorar e controlar os endereços IP distribuídos. Eles podem ser usados para solucionar problemas relacionados a concessões e renovação de IPs.


📂 **Arquivo analisado:** `/var/lib/dhcp/dhcpd.leases`
📸 **Imagem:**  
![Verificando arquivos de concessão do DHCP](imagem/dhcpdleaves.png)
📂 **Arquivo analisado:** `/var/lib/dhcp/dhcpd.leases`

🔍 **Explicação rápida:**  
O arquivo `dhcpd.leases` armazena as informações de concessões de endereços IP feitas pelo servidor DHCP. Cada concessão registrada neste arquivo contém:

- O endereço IP atribuído ao cliente;
- O horário de início e término da concessão;
- O endereço MAC do cliente (identificado por `hardware ethernet`);
- O identificador único do cliente DHCP (`uid`);
- O nome do host (se fornecido).

