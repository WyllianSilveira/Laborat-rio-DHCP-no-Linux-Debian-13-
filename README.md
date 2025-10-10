# 🛠️ Laboratório: Servidor DHCP com Debian 13

Este repositório contém a documentação de um laboratório prático que simula a configuração de um servidor **DHCP** no **Debian 13**, utilizando **máquinas virtuais no VirtualBox** em um ambiente isolado.

## 🌐 Descrição do Ambiente

- O host é um **Windows 11**, conectado à internet via Wi-Fi.
- O **servidor Debian 13** foi configurado com o serviço `isc-dhcp-server`.
- Um **cliente Windows 10** obtém IP automaticamente pela rede interna.
- Rede interna: `10.200.0.0/24` (Classe A - isolada).

## ⚙️ Objetivos

- Configurar o `isc-dhcp-server` para distribuição automática de IPs.
- Validar o funcionamento do DHCP com um cliente Windows.
- Simular uma rede realista para fins educacionais ou de testes.

## 📂 Conteúdo do Repositório

- Diagrama da arquitetura da rede (`diagrama-lan-dhcp-debian.png`)
- Prints e arquivos de configuração (`/imagens`)
- Documentação detalhada do processo (`Laboratório_DHCP.md`)

## 🧑‍💻 Autor

**Wyllian Silveira Calixto**  
[github.com/WyllianSilveira](https://github.com/WyllianSilveira)

