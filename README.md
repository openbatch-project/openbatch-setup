# OpenBatch Setup

Este repositório contém os scripts e playbooks do **Ansible** responsáveis pela infraestrutura como código (IaC) do projeto principal [OpenBatch](https://github.com/marcusmartinss/openbatch). Ele provisiona todo o ambiente necessário para a execução, desde a configuração de rede até a aplicação em si.

## Pré-requisitos de Automação

* Máquina de controle com **Ansible** instalado.
* Acesso SSH às máquinas alvo (Manager e Workers).
* Usuário com privilégios `sudo` nas máquinas alvo.

## 1. Configuração do Inventário

Crie o arquivo de inventário local a partir do exemplo fornecido:

```bash
cp inventory.example.ini inventory.ini
```

Em seguida, edite o arquivo `inventory.ini` preenchendo os IPs e variáveis de acordo com o seu ambiente:

```ini
[slurm_controller]
# Defina o IP de conexão (ansible_host) e o IP estático desejado (static_ip)
manager0 ansible_host=192.168.x.x interface_name=eth0 static_ip=192.168.122.10

[slurm_nodes]
worker0 ansible_host=192.168.x.x interface_name=eth0 static_ip=192.168.122.11
worker1 ansible_host=192.168.x.x interface_name=eth0 static_ip=192.168.122.12

[all:vars]
ansible_user=seu_usuario_ssh
# Defina uma senha forte para o banco de dados do Slurm
db_password="sua_senha_segura"
```

## 2. Execução dos Playbooks

A implantação é dividida em 3 fases para facilitar o troubleshooting e garantir estabilidade:

### Fase 1: Rede Base
Configura IPs estáticos e resolução de nomes (`/etc/hosts`) em todos os nós do cluster.

```bash
ansible-playbook 01-network-config.yaml --ask-become-pass
```

> **Nota:** A conexão SSH pode cair momentaneamente durante a troca de IP se a rede estiver sendo reconfigurada para a mesma interface conectada.

### Fase 2: Cluster SLURM
Instala e configura o Munge, Slurm (Controller, DBD e Daemons), MariaDB, além de configurar as restrições de Cgroups e limites de segurança (limits.conf).

```bash
ansible-playbook 02-slurm-config.yaml --ask-become-pass
```

### Fase 3: Aplicação OpenBatch
Instala o Node.js, ClamAV, compila o painel frontend e configura o serviço via systemd no Manager para inicializar o OpenBatch.

```bash
ansible-playbook 03-deploy-openbatch.yaml --ask-become-pass
```
