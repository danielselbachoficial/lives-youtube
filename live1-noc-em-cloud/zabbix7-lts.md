# 📊 NOC em Cloud - Manual Completo Zabbix 7.0

> **Manual Técnico**: Implementação completa de monitoramento com Zabbix 7.0 + PostgreSQL no Debian 12

[![Zabbix](https://img.shields.io/badge/Zabbix-7.0-red?style=for-the-badge&logo=zabbix)](https://www.zabbix.com/)
[![PostgreSQL](https://img.shields.io/badge/PostgreSQL-15-blue?style=for-the-badge&logo=postgresql)](https://www.postgresql.org/)
[![Debian](https://img.shields.io/badge/Debian-12-red?style=for-the-badge&logo=debian)](https://www.debian.org/)
[![Nginx](https://img.shields.io/badge/Nginx-1.22-green?style=for-the-badge&logo=nginx)](https://nginx.org/)

---

## 📑 **Índice**

- [🎯 Objetivo e Pré-requisitos](#-objetivo-e-pré-requisitos)
- [🚀 Preparação do Ambiente](#-preparação-do-ambiente)
- [🔧 Instalação dos Componentes](#-instalação-dos-componentes)
- [🗄️ Configuração do Banco de Dados](#️-configuração-do-banco-de-dados)
- [⚙️ Configuração do Zabbix Server](#️-configuração-do-zabbix-server)
- [🌐 Configuração do Nginx](#-configuração-do-nginx)
- [🔧 Configuração do PHP](#-configuração-do-php)
- [🚀 Inicialização dos Serviços](#-inicialização-dos-serviços)
- [✅ Verificações Pós-Instalação](#-verificações-pós-instalação)
- [🌐 Configuração Web (Setup Wizard)](#-configuração-web-setup-wizard)
- [🔐 Primeiro Acesso](#-primeiro-acesso)
- [🔥 Configuração de Firewall](#-configuração-de-firewall)
- [🛠️ Troubleshooting](#️-troubleshooting)
- [🤖 Scripts de Automação](#-scripts-de-automação)
- [📈 Próximos Passos](#-próximos-passos)

---

## 🎯 **Objetivo e Pré-requisitos**

### **Objetivo**
Implementar um sistema de monitoramento Zabbix completo para NOC (Network Operations Center) em ambiente cloud, proporcionando visibilidade total da infraestrutura.

### **Pré-requisitos**

|
 Componente 
|
 Especificação 
|
|
------------
|
---------------
|
|
**
Sistema Operacional
**
|
 Debian 12 (Bookworm) 
|
|
**
RAM
**
|
 Mínimo 2GB (Recomendado 4GB+) 
|
|
**
CPU
**
|
 Mínimo 2 vCPUs 
|
|
**
Storage
**
|
 Mínimo 20GB (Recomendado 50GB+) 
|
|
**
Acesso
**
|
 Root ou usuário com sudo 
|
|
**
Conectividade
**
|
 Internet para download de pacotes 
|
|
**
Portas
**
|
 80, 443, 8080, 10051, 5432 
|

### **Arquitetura da Solução**
┌─────────────────┐ ┌─────────────────┐ ┌─────────────────┐ │ Zabbix Web │ │ Zabbix Server │ │ PostgreSQL │ │ (Nginx+PHP) │◄──►│ (Backend) │◄──►│ (Database) │ └─────────────────┘ └─────────────────┘ └─────────────────┘ │ │ │ ▼ ▼ ▼ Port 8080 Port 10051 Port 5432


---

## 🚀 **Preparação do Ambiente**

### **a) Instalar Dependências Básicas**
```bash
# Atualizar sistema e instalar ferramentas essenciais
apt update && apt install -y sudo wget curl
```
### **b) Configurar Repositório Zabbix**
```bash
# Download e instalação do repositório oficial
wget https://repo.zabbix.com/zabbix/7.0/debian/pool/main/z/zabbix-release/zabbix-release_latest_7.0+debian12_all.deb
dpkg -i zabbix-release_latest_7.0+debian12_all.deb
apt update
```

## 🔧 Instalação dos Componentes
### **c) Instalar Zabbix Server e Dependências**
```bash
# Instalação completa do Zabbix com PostgreSQL
apt install -y zabbix-server-pgsql zabbix-frontend-php php8.2-pgsql \
               zabbix-nginx-conf zabbix-sql-scripts zabbix-agent2 \
               postgresql postgresql-contrib
```

### **d) Instalar Plugins do Zabbix Agent 2**
```bash
# Plugins para monitoramento avançado
apt install -y zabbix-agent2-plugin-mongodb \
               zabbix-agent2-plugin-mssql \
               zabbix-agent2-plugin-postgresql
```

## 🗄️ Configuração do Banco de Dados
### **e) Preparar PostgreSQL**
```bash
# Inicializar e habilitar PostgreSQL
systemctl start postgresql
systemctl enable postgresql

# Navegar para diretório seguro
cd /var/lib/postgresql
```

### **f) Criar Usuário e Banco**
```bash
# Criar usuário zabbix (será solicitada senha)
sudo -u postgres createuser --pwprompt zabbix

# Criar banco de dados
sudo -u postgres createdb -O zabbix zabbix

# Verificar criação
sudo -u postgres psql -c "\du" | grep zabbix
sudo -u postgres psql -c "\l" | grep zabbix
```

### **g) Importar Schema do Zabbix**
```bash
# Importar estrutura inicial (inserir senha quando solicitado)
zcat /usr/share/zabbix-sql-scripts/postgresql/server.sql.gz | sudo -u zabbix psql zabbix
```

## ⚙️ Configuração do Zabbix Server
### **h) Editar Configuração Principal**
```bash
# Fazer backup da configuração
cp /etc/zabbix/zabbix_server.conf /etc/zabbix/zabbix_server.conf.backup

# Editar arquivo de configuração
nano /etc/zabbix/zabbix_server.conf
```

### Configurações obrigatórias:
```bash
DBHost=localhost
DBName=zabbix
DBUser=zabbix
DBPassword=sua_senha_aqui
DBPort=5432
```

### 🌐 Configuração do Nginx

### **i) Configurar Servidor Web**
```bash
# Editar configuração do Nginx para Zabbix
nano /etc/zabbix/nginx.conf
```
