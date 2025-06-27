# Manual de Instalação do 📊 Zabbix Proxy 7.0 LTS

[![Zabbix](https://img.shields.io/badge/Zabbix-7.0-red?style=for-the-badge&logo=zabbix)](https://www.zabbix.com/)
[![MySQL](https://img.shields.io/badge/MySQL-8.0-blue?style=for-the-badge&logo=mysql)](https://www.mysql.com/)
[![Ubuntu](https://img.shields.io/badge/Ubuntu-24.04-orange?style=for-the-badge&logo=ubuntu)](https://ubuntu.com/)

## 📋 Pré-requisitos

- Ubuntu 24.04 LTS
- Acesso root ou sudo
- Conexão com internet
- Servidor Zabbix principal já configurado

## 🔧 1. Instalar Dependências

```bash
# Instalar dependências básicas
apt-get install -y wget curl net-tools 

# Configurar timezone para São Paulo
timedatectl set-timezone America/Sao_Paulo
```

## 📦 2. Adicionar Repositório Oficial do Zabbix

```bash
# Baixar o pacote de repositório para Ubuntu 24.04
wget https://repo.zabbix.com/zabbix/7.0/ubuntu/pool/main/z/zabbix-release/zabbix-release_7.0-2+ubuntu24.04_all.deb

# Instalar o pacote de repositório
dpkg -i zabbix-release_7.0-2+ubuntu24.04_all.deb

# Atualizar lista de pacotes
apt update
```

## ⚙️ 3. Instalar Zabbix Proxy e Dependências

```bash
# Instalar o Zabbix Proxy MySQL e scripts SQL
apt install zabbix-proxy-mysql zabbix-sql-scripts

# Instalar MySQL Server se ainda não estiver instalado
apt install mysql-server
```

## ✅ 4. Verificar Instalação

```bash
# Verificar se os pacotes foram instalados corretamente
dpkg -l | grep zabbix

# Verificar versão do Zabbix Proxy
zabbix_proxy --version
```

## 🗄️ 5. Configurar MySQL

```bash
# Configurar MySQL (se primeira instalação)
mysql_secure_installation

# Conectar ao MySQL
mysql -u root -p
```

## 🏗️ 6. Criar Banco de Dados para o Proxy

Execute os comandos SQL abaixo no MySQL:

```sql
-- Criar banco de dados
CREATE DATABASE zabbix_proxy CHARACTER SET utf8mb4 COLLATE utf8mb4_bin;

-- Criar usuário
CREATE USER 'zabbix_proxy'@'localhost' IDENTIFIED BY '@dmin123';

-- Conceder privilégios
GRANT ALL PRIVILEGES ON zabbix_proxy.* TO 'zabbix_proxy'@'localhost';

-- Habilitar criação de funções sem privilégios SUPER
SET GLOBAL log_bin_trust_function_creators = 1;

-- Aplicar mudanças
FLUSH PRIVILEGES;

-- Sair do MySQL
EXIT;
```

## 📊 7. Importar Schema do Banco

```bash
# Importar estrutura do banco de dados
cat /usr/share/zabbix-sql-scripts/mysql/proxy.sql | mysql -u zabbix_proxy -p zabbix_proxy

# Verificar se as tabelas foram criadas corretamente
mysql -u zabbix_proxy -p zabbix_proxy -e "SELECT COUNT(*) as 'Total de Tabelas' FROM information_schema.tables WHERE table_schema = 'zabbix_proxy';"

# Verificar algumas tabelas específicas
mysql -u zabbix_proxy -p zabbix_proxy -e "SHOW TABLES LIKE 'hosts'; SHOW TABLES LIKE 'items'; SHOW TABLES LIKE 'proxy_%';"
```

## ⚙️ 8. Configurar o Zabbix Proxy

```bash
# Editar arquivo de configuração
nano /etc/zabbix/zabbix_proxy.conf
```

### Configurações principais no arquivo:

```ini
# IP do servidor Zabbix principal
Server=IP_DO_SEU_ZABBIX_SERVER

# Nome do proxy (deve ser único)
Hostname=NOME_DO_ZABBIX-PROXY

# Porta de escuta
ListenPort=10051

# Configurações do banco de dados
DBHost=localhost
DBName=zabbix_proxy
DBUser=zabbix_proxy
DBPassword=@dmin123

# Log
LogFile=/var/log/zabbix/zabbix_proxy.log
LogFileSize=10
```

## 🚀 9. Iniciar o Serviço

```bash
# Criar diretório de logs
mkdir -p /var/log/zabbix
chown zabbix:zabbix /var/log/zabbix

# Habilitar e iniciar o serviço
systemctl enable zabbix-proxy
systemctl start zabbix-proxy

# Verificar status
systemctl status zabbix-proxy
```

## 🔍 10. Verificar Funcionamento

```bash
# Verificar logs
tail -f /var/log/zabbix/zabbix_proxy.log

# Verificar se está escutando na porta
ss -tlnp | grep 10051
```

## 📝 Notas Importantes

- **Segurança**: Altere a senha padrão `@dmin123` por uma senha mais segura
- **Firewall**: Certifique-se de que a porta 10051 esteja liberada
- **Nome do Proxy**: O nome configurado em `Hostname` deve ser único na sua infraestrutura
- **Conectividade**: O proxy deve conseguir se comunicar com o servidor Zabbix principal

## 🔧 Comandos Úteis

```bash
# Reiniciar o serviço
systemctl restart zabbix-proxy

# Ver logs em tempo real
journalctl -u zabbix-proxy -f

# Verificar configuração
zabbix_proxy -c /etc/zabbix/zabbix_proxy.conf -T
```

## 🆘 Solução de Problemas

Se encontrar problemas, verifique:

1. **Logs do sistema**: `journalctl -u zabbix-proxy`
2. **Logs do Zabbix**: `/var/log/zabbix/zabbix_proxy.log`
3. **Conectividade de rede**: `telnet IP_ZABBIX_SERVER 10051`
4. **Banco de dados**: Teste a conexão MySQL
