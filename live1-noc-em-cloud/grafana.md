# 📊 NOC em Cloud - Manual Completo Grafana 12.0.1

> **Manual Técnico**: Implementação completa de visualização com Grafana 12.0.1 + PostgreSQL no Ubuntu Server 24.04 LTS

[![Grafana](https://img.shields.io/badge/Grafana-12.0.1-orange?style=for-the-badge&logo=grafana)](https://grafana.com/)
[![PostgreSQL](https://img.shields.io/badge/PostgreSQL-16-blue?style=for-the-badge&logo=postgresql)](https://www.postgresql.org/)
[![Ubuntu](https://img.shields.io/badge/Ubuntu-24.04_LTS-orange?style=for-the-badge&logo=ubuntu)](https://ubuntu.com/)

---

## 📑 **Índice**

- [🎯 Objetivo e Pré-requisitos](#-objetivo-e-pré-requisitos)
- [🚀 Preparação do Ambiente](#-preparação-do-ambiente)
- [🔧 Instalação dos Componentes](#-instalação-dos-componentes)
- [🗄️ Configuração do Banco de Dados](#️-configuração-do-banco-de-dados)
- [⚙️ Configuração do Grafana Server](#️-configuração-do-grafana-server)
- [🚀 Inicialização dos Serviços](#-inicialização-dos-serviços)
- [✅ Verificações Pós-Instalação](#-verificações-pós-instalação)
- [🌐 Configuração Web (Setup Inicial)](#-configuração-web-setup-inicial)
- [🔐 Primeiro Acesso](#-primeiro-acesso)
- [📊 Configuração de Data Sources](#-configuração-de-data-sources)
- [🔧 Configurações Avançadas](#-configurações-avançadas)
- [🎯 Próximos Passos Recomendados](#-próximos-passos-recomendados)
- [📚 Recursos Úteis](#-recursos-úteis)
- [🐛 Troubleshooting Comum](#-troubleshooting-comum)

---

## 🎯 **Objetivo e Pré-requisitos**

### **Objetivo**
Implementar uma plataforma de visualização Grafana 12.0.1 completa para NOC (Network Operations Center) em ambiente cloud, proporcionando dashboards avançados e análise de dados em tempo real via HTTP direto.

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
 Ubuntu Server 24.04 LTS (Noble Numbat) 
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
 3000, 5432 
|

### **Arquitetura da Solução**
┌─────────────────┐ ┌─────────────────┐ │ Grafana Server │ │ PostgreSQL │ │ (Direct Access) │◄──►│ (Database) │ └─────────────────┘ └─────────────────┘ │ │ ▼ ▼ Port 3000 Port 5432


---

## 🚀 **Preparação do Ambiente**

### **a) Instalar Dependências Básicas**
```bash
# Atualizar sistema e instalar ferramentas essenciais
sudo apt update && sudo apt upgrade -y
sudo apt install -y wget curl gnupg2 software-properties-common apt-transport-https lsb-release

b) Configurar Repositório Grafana
bash
Copiar

# Adicionar chave GPG oficial do Grafana
sudo mkdir -p /etc/apt/keyrings/
wget -q -O - https://apt.grafana.com/gpg.key | gpg --dearmor | sudo tee /etc/apt/keyrings/grafana.gpg > /dev/null

# Adicionar repositório oficial para Ubuntu 24.04
echo "deb [signed-by=/etc/apt/keyrings/grafana.gpg] https://apt.grafana.com stable main" | sudo tee /etc/apt/sources.list.d/grafana.list

# Atualizar lista de pacotes
sudo apt update
🔧 Instalação dos Componentes
c) Instalar Grafana 12.0.1 e Dependências
bash
Copiar

# Verificar versões disponíveis
apt list -a grafana

# Instalação do Grafana OSS (Open Source) versão específica
sudo apt install -y grafana=12.0.1

# Manter versão específica (evitar atualizações automáticas)
sudo apt-mark hold grafana

# Instalar PostgreSQL
sudo apt install -y postgresql postgresql-contrib

# Instalar ferramentas úteis para monitoramento
sudo apt install -y htop iotop nethogs net-tools
d) Verificar Versão Instalada
bash
Copiar

# Confirmar versão do Grafana
grafana-server --version

# Verificar status inicial
sudo systemctl status grafana-server
🗄️ Configuração do Banco de Dados
e) Preparar PostgreSQL
bash
Copiar

# Inicializar e habilitar PostgreSQL
sudo systemctl start postgresql
sudo systemctl enable postgresql

# Verificar versão do PostgreSQL
sudo -u postgres psql -c "SELECT version();"
f) Criar Usuário e Banco para Grafana
bash
Copiar

# Criar usuário grafana (será solicitada senha - anote-a!)
sudo -u postgres createuser --pwprompt grafana

# Criar banco de dados
sudo -u postgres createdb -O grafana grafana

# Verificar criação
sudo -u postgres psql -c "\du" | grep grafana
sudo -u postgres psql -c "\l" | grep grafana
g) Configurar Acesso ao PostgreSQL
bash
Copiar

# Editar configuração de autenticação (PostgreSQL 16)
sudo nano /etc/postgresql/16/main/pg_hba.conf

# Adicionar linha para acesso local do Grafana (adicione antes das outras regras local)
# local   grafana         grafana                                 md5

# Ou usar comando para adicionar automaticamente
sudo sed -i '/^local.*all.*all.*peer/i local   grafana         grafana                                 md5' /etc/postgresql/16/main/pg_hba.conf

# Reiniciar PostgreSQL
sudo systemctl restart postgresql

# Testar conexão
sudo -u grafana psql -d grafana -c "SELECT 1;"
⚙️ Configuração do Grafana Server
h) Gerar Secret Key
bash
Copiar

# Gerar secret key aleatória de 32 caracteres
SECRET_KEY=$(openssl rand -hex 16)
echo "Secret Key gerada: $SECRET_KEY"

# Ou gerar usando /dev/urandom
SECRET_KEY=$(head -c 32 /dev/urandom | base64 | tr -d "=+/" | cut -c1-32)
echo "Secret Key gerada: $SECRET_KEY"

# Ou usar um comando mais simples
SECRET_KEY=$(cat /dev/urandom | tr -dc 'a-zA-Z0-9' | fold -w 32 | head -n 1)
echo "Secret Key gerada: $SECRET_KEY"
i) Editar Configuração Principal
bash
Copiar

# Fazer backup da configuração
sudo cp /etc/grafana/grafana.ini /etc/grafana/grafana.ini.backup

# Editar arquivo de configuração
sudo nano /etc/grafana/grafana.ini
Configurações Essenciais para Grafana 12.0.1:
ini
Copiar

#################################### Server ####################################
[server]
# Protocolo (http)
protocol = http

# Endereço IP para bind (0.0.0.0 para todas as interfaces)
http_addr = 0.0.0.0

# Porta HTTP
http_port = 3000

# Domínio público (substitua pelo seu IP/domínio)
domain = SEU_IP_PUBLICO

# URL raiz completa
root_url = http://SEU_IP_PUBLICO:3000/

# Habilitar gzip
enable_gzip = true

#################################### Database ####################################
[database]
# Tipo do banco de dados
type = postgres

# Host do banco
host = localhost:5432

# Nome do banco
name = grafana

# Usuário do banco
user = grafana

# Senha do banco (substitua pela sua senha)
password = SUA_SENHA_AQUI

# SSL mode
ssl_mode = disable

# Configurações de conexão
max_idle_conn = 2
max_open_conn = 0
conn_max_lifetime = 14400

#################################### Security ####################################
[security]
# Chave secreta para cookies (use a gerada anteriormente)
secret_key = SUA_SECRET_KEY_GERADA

# Permitir embedding em iframes
allow_embedding = true

# Configurações básicas
admin_user = admin
admin_password = admin

#################################### Users ####################################
[users]
# Permitir registro de usuários
allow_sign_up = false

# Permitir criação de organizações
allow_org_create = false

# Atribuição automática de role
auto_assign_org_role = Viewer

# Verificação de email obrigatória
verify_email_enabled = false

# Configurações de usuário padrão
default_theme = dark

#################################### Anonymous Auth ####################################
[auth.anonymous]
# Habilitar acesso anônimo (opcional)
enabled = false

#################################### Logging ####################################
[log]
# Modo de log
mode = console file

# Nível de log
level = info

# Formato de log
format = text

#################################### Paths ####################################
[paths]
# Diretório de dados
data = /var/lib/grafana

# Diretório de logs
logs = /var/log/grafana

# Diretório de plugins
plugins = /var/lib/grafana/plugins

# Diretório de provisionamento
provisioning = /etc/grafana/provisioning

#################################### Alerting ####################################
[alerting]
# Habilitar alerting
enabled = true

# Configurações de execução
execute_alerts = true

#################################### Unified Alerting ####################################
[unified_alerting]
# Habilitar unified alerting
enabled = true

# Configurações de execução
execute_alerts = true

# Configurações de avaliação
evaluation_timeout = 30s
max_attempts = 3

#################################### Explore ####################################
[explore]
# Habilitar Explore
enabled = true

#################################### Plugins ####################################
[plugins]
# Permitir plugins não assinados
allow_loading_unsigned_plugins = 

# Configurações de marketplace
plugin_admin_enabled = true

#################################### Live ####################################
[live]
# Configurações de streaming
max_connections = 100
allowed_origins = *
j) Configurar Permissões
bash
Copiar

# Definir proprietário correto dos arquivos
sudo chown -R grafana:grafana /var/lib/grafana
sudo chown -R grafana:grafana /var/log/grafana
sudo chown -R grafana:grafana /etc/grafana

# Definir permissões corretas
sudo chmod 755 /var/lib/grafana
sudo chmod 755 /var/log/grafana
sudo chmod 640 /etc/grafana/grafana.ini
🚀 Inicialização dos Serviços
k) Iniciar e Habilitar Serviços
bash
Copiar

# Iniciar todos os serviços
sudo systemctl start postgresql
sudo systemctl start grafana-server

# Habilitar inicialização automática
sudo systemctl enable postgresql
sudo systemctl enable grafana-server

# Verificar status
sudo systemctl status postgresql grafana-server
✅ Verificações Pós-Instalação
Status dos Serviços
bash
Copiar

# Verificar se todos os serviços estão ativos
sudo systemctl is-active grafana-server postgresql

# Verificar logs do Grafana
sudo journalctl -u grafana-server -n 20 --no-pager
Verificar Conectividade
bash
Copiar

# Verificar portas em uso
sudo ss -tlnp | grep -E "(3000|5432)"

# Testar conexão local do Grafana
curl -I http://localhost:3000

# Testar conexão com banco
sudo -u grafana psql -d grafana -c "SELECT version();"
Verificar Logs
bash
Copiar

# Logs do Grafana
sudo tail -f /var/log/grafana/grafana.log

# Verificar se não há erros
sudo journalctl -u grafana-server --since "5 minutes ago"
🌐 Configuração Web (Setup Inicial)
Acesso à Interface
URL de acesso: http://SEU_IP_PUBLICO:3000
Primeiro Login
Acesse a URL no navegador
Será apresentada a tela de login do Grafana 12.0.1
Use as credenciais padrão
🔐 Primeiro Acesso
Credenciais Padrão
Usuário: admin
Senha: admin
Configuração Inicial
Alterar senha padrão (será solicitado no primeiro login)
Configurar timezone para America/Sao_Paulo
Selecionar tema (Dark recomendado para NOC)

🔧 Configurações Avançadas
Instalar Plugins Úteis
bash
Copiar

# Instalar plugins via CLI
sudo grafana-cli plugins install grafana-clock-panel
sudo grafana-cli plugins install grafana-worldmap-panel
sudo grafana-cli plugins install grafana-piechart-panel

# Reiniciar Grafana após instalar plugins
sudo systemctl restart grafana-server
Configurar Firewall (UFW)
bash
Copiar

# Permitir acesso à porta do Grafana
sudo ufw allow 3000/tcp

# Verificar regras
sudo ufw status
Backup Automático
bash
Copiar

# Criar diretório de backup
sudo mkdir -p /backup/grafana

# Criar script de backup
sudo tee /usr/local/bin/grafana-backup.sh << 'EOF'
#!/bin/bash
BACKUP_DIR="/backup/grafana"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p $BACKUP_DIR

# Backup do banco de dados
sudo -u postgres pg_dump grafana > $BACKUP_DIR/grafana_db_$DATE.sql

# Backup dos arquivos de configuração
tar -czf $BACKUP_DIR/grafana_config_$DATE.tar.gz /etc/grafana/

# Backup dos dashboards e dados
tar -czf $BACKUP_DIR/grafana_data_$DATE.tar.gz /var/lib/grafana/

# Manter apenas backups dos últimos 7 dias
find $BACKUP_DIR -name "*.sql" -mtime +7 -delete
find $BACKUP_DIR -name "*.tar.gz" -mtime +7 -delete

echo "Backup concluído: $DATE"
EOF

# Tornar executável
sudo chmod +x /usr/local/bin/grafana-backup.sh

# Agendar backup diário às 2h da manhã
echo "0 2 * * * /usr/local/bin/grafana-backup.sh" | sudo crontab -
Monitoramento do Sistema
bash
Copiar

# Instalar node-exporter para monitoramento
sudo apt install -y prometheus-node-exporter

# Habilitar node-exporter
sudo systemctl enable prometheus-node-exporter
sudo systemctl start prometheus-node-exporter

# Verificar se está rodando
sudo systemctl status prometheus-node-exporter
Comandos Úteis para Secret Key
bash
Copiar

# Método 1: OpenSSL (mais comum)
openssl rand -hex 16

# Método 2: /dev/urandom com base64
head -c 32 /dev/urandom | base64 | tr -d "=+/" | cut -c1-32

# Método 3: /dev/urandom simples
cat /dev/urandom | tr -dc 'a-zA-Z0-9' | fold -w 32 | head -n 1

# Método 4: Usando pwgen (se instalado)
sudo apt install pwgen
pwgen -s 32 1

# Método 5: Usando Python
python3 -c "import secrets; print(secrets.token_urlsafe(24))"
🎯 Próximos Passos Recomendados
Configurar Data Sources específicos para seus sistemas
Importar dashboards da comunidade Grafana
Criar dashboards personalizados para seu NOC
Configurar alertas básicos
Implementar backup automatizado
Configurar usuários e permissões
Instalar plugins adicionais conforme necessidade
📚 Recursos Úteis
Documentação Oficial: 

grafana.com
Dashboards Comunidade: 

grafana.com
Plugins: 

grafana.com
Alerting: 

grafana.com
🐛 Troubleshooting Comum
Grafana não inicia
bash
Copiar

# Verificar logs
sudo journalctl -u grafana-server -n 50

# Verificar configuração
sudo grafana-server -config /etc/grafana/grafana.ini
Problemas de conexão com PostgreSQL
bash
Copiar

# Testar conexão manual
sudo -u grafana psql -h localhost -d grafana -U grafana

# Verificar configuração do pg_hba.conf
sudo cat /etc/postgresql/16/main/pg_hba.conf | grep grafana
Erro de Secret Key
bash
Copiar

# Gerar nova secret key
NEW_SECRET=$(openssl rand -hex 16)
echo "Nova Secret Key: $NEW_SECRET"

# Substituir no arquivo de configuração
sudo sed -i "s/secret_key = .*/secret_key = $NEW_SECRET/" /etc/grafana/grafana.ini

# Reiniciar Grafana
sudo systemctl restart grafana-server
Problemas com Live Features
bash
Copiar

# Verificar configurações de Live no Grafana
grep -A 10 "$$live$$" /etc/grafana/grafana.ini

# Verificar se WebSocket está funcionando
curl -i -N -H "Connection: Upgrade" -H "Upgrade: websocket" http://localhost:3000/api/live/ws
⚠️ IMPORTANTE: Este manual foi otimizado para acesso HTTP direto na porta 3000. Certifique-se de que a porta esteja liberada no firewall da sua cloud.
