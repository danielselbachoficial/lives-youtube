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

Remover tudo e colar a informação abaixo:
```bash
server {
    listen 80;
    server_name SEU_IP_PUBLICO;  # Substitua pelo seu IP público
    
    root /usr/share/zabbix;
    index index.php;

    location = /favicon.ico {
        log_not_found off;
    }

    location / {
        try_files $uri $uri/ =404;
    }

    location /assets {
        access_log off;
        expires 10d;
    }

    location ~ /\.ht {
        deny all;
    }

    location ~ /(api\/|conf[^\.]|include|locale) {
        deny all;
        return 404;
    }

    location /vendor {
        deny all;
        return 404;
    }

    location ~ [^/]\.php(/|$) {
        fastcgi_pass unix:/var/run/php/php8.2-fpm.sock;
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_index index.php;

        fastcgi_param DOCUMENT_ROOT /usr/share/zabbix;
        fastcgi_param SCRIPT_FILENAME /usr/share/zabbix$fastcgi_script_name;
        fastcgi_param PATH_TRANSLATED /usr/share/zabbix$fastcgi_script_name;

        include fastcgi_params;
        fastcgi_param QUERY_STRING $query_string;
        fastcgi_param REQUEST_METHOD $request_method;
        fastcgi_param CONTENT_TYPE $content_type;
        fastcgi_param CONTENT_LENGTH $content_length;

        fastcgi_intercept_errors on;
        fastcgi_ignore_client_abort off;
        fastcgi_connect_timeout 60;
        fastcgi_send_timeout 180;
        fastcgi_read_timeout 180;
        fastcgi_buffer_size 128k;
        fastcgi_buffers 4 256k;
        fastcgi_busy_buffers_size 256k;
        fastcgi_temp_file_write_size 256k;
    }
}
```

### **j) Incluir Configuração no Nginx Principal**
```bash
# Criar link simbólico
ln -sf /etc/zabbix/nginx.conf /etc/nginx/sites-enabled/zabbix.conf

# Testar configuração
nginx -t
```

### 🔧 Configuração do PHP

### **k) Método Rápido - Aplicar Todas as Configurações**
```bash
# Fazer backup
cp /etc/php/8.2/fmp/php.ini /etc/php/8.2/fpm/php.ini.backup

# Aplicar todas as configurações necessárias
sed -i 's/max_execution_time = 30/max_execution_time = 300/' /etc/php/8.2/fpm/php.ini
sed -i 's/max_input_time = 60/max_input_time = 300/' /etc/php/8.2/fpm/php.ini
sed -i 's/;max_input_vars = 1000/max_input_vars = 10000/' /etc/php/8.2/fpm/php.ini
sed -i 's/memory_limit = 128M/memory_limit = 256M/' /etc/php/8.2/fpm/php.ini
sed -i 's/post_max_size = 8M/post_max_size = 16M/' /etc/php/8.2/fpm/php.ini
sed -i 's/;date.timezone =/date.timezone = America\/Sao_Paulo/' /etc/php/8.2/fpm/php.ini
```

Ou editar manualmente:

Parâmetro	com Valor Necessário
- max_execution_time= 300
- max_input_time= 300
- max_input_vars=	10000
- memory_limit=	256M
- post_max_size=	16M
- date.timezone=	America/Sao_Paulo

###🚀 Inicialização dos Serviços

#### **l) Iniciar e Habilitar Serviços**
```bash
# Reiniciar todos os serviços
systemctl restart zabbix-server zabbix-agent2 nginx php8.2-fpm postgresql

# Habilitar inicialização automática
systemctl enable zabbix-server zabbix-agent2 nginx php8.2-fpm postgresql
```

#### ✅ Verificações Pós-Instalação
Status dos Serviços
```bash
# Verificar se todos os serviços estão ativos
systemctl status zabbix-server zabbix-agent2 nginx php8.2-fpm postgresql
```

#### Verificar Configurações PHP
```bash
# Confirmar parâmetros PHP
php -r "
echo 'max_execution_time: ' . ini_get('max_execution_time') . PHP_EOL;
echo 'post_max_size: ' . ini_get('post_max_size') . PHP_EOL;
echo 'max_input_time: ' . ini_get('max_input_time') . PHP_EOL;
echo 'max_input_vars: ' . ini_get('max_input_vars') . PHP_EOL;
echo 'memory_limit: ' . ini_get('memory_limit') . PHP_EOL;
echo 'date.timezone: ' . ini_get('date.timezone') . PHP_EOL;
"
```

#### Testar Conectividade
```bash
# Verificar portas em uso
netstat -tlnp | grep -E "(80|8080|10051|5432)"

# Testar conexão com banco
sudo -u zabbix psql -d zabbix -c "SELECT version();"
```

### 🌐 Configuração Web (Setup Wizard)

### Acesso à Interface
URL: http://SEU_IP:8080/setup.php

#### Passo a Passo do Setup:

##### 1. Verificar Pré-requisitos
- ✅ Todos os requisitos devem estar em verde
- ❌ Se algum estiver vermelho, revisar configurações PHP

##### 2. Configurar Banco de Dados
- Database type: PostgreSQL
- Database host: localhost
- Database port: 5432
- Database name: zabbix
- Database schema: (deixar vazio)
- User: zabbix
- Password: [sua senha definida anteriormente]

##### 3. Configurações do Zabbix Server
- Host: localhost
- Port: 10051
- Name: Zabbix Server NOC

##### 4. Configurações Finais
- Fuso horário: America/Sao_Paulo
- Tema: Blue (recomendado)

### 🔐 Primeiro Acesso

#### Credenciais Padrão
- URL: http://SEU_IP:8080
- Usuário: Admin
- Senha: zabbix

- ⚠️ IMPORTANTE: Altere a senha padrão imediatamente após o primeiro login!

### 🔥 Configuração de Firewall
```bash
# Permitir acesso às portas necessárias
ufw allow 80/tcp      # HTTP
ufw allow 443/tcp     # HTTPS
ufw allow 8080/tcp    # Zabbix Web
ufw allow 10051/tcp   # Zabbix Server
```
