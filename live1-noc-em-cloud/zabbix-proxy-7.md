# Manual de Instalação do 📊 Zabbix Proxy 7.0 LTS

[![Zabbix](https://img.shields.io/badge/Zabbix-7.0-red?style=for-the-badge&logo=zabbix)](https://www.zabbix.com/)
[![MySQL](https://img.shields.io/badge/MySQL-8.0-blue?style=for-the-badge&logo=mysql)](https://www.mysql.com/)

## 📋 Pré-requisitos

- Debian 12
- Acesso root ou sudo
- Conexão com internet
- Servidor Zabbix principal já configurado

## 🔧 1. Instalar Dependências

```bash
# Instalar dependências básicas
apt update
apt upgrade -y
apt-get install -y wget curl net-tools sudo gnupg2 lsb-release apt-transport-https vim

# Configurar timezone para São Paulo
timedatectl set-timezone America/Sao_Paulo
```

## 🗄️ 2. Instalação e Configuração do Banco de Dados PostgreSQL

```bash
# Instalar o servidor e cliente PostgreSQL
sudo apt install -y postgresql postgresql-client
```

## 🏗️ 3. Criar Banco de Dados para o Proxy
```bash
# Conectar ao PostgreSQL
sudo -u postgres psql
```

Execute os comandos SQL abaixo no PostgreSQL:

```sql
CREATE USER zabbix WITH PASSWORD '@dmin123';
CREATE DATABASE zabbix_proxy
  WITH OWNER zabbix
  ENCODING 'UTF8'
  LC_COLLATE='C'
  LC_CTYPE='C'
  TEMPLATE template0;
GRANT ALL PRIVILEGES ON DATABASE zabbix_proxy TO zabbix;
\q
```

## 📦 4. Adicionar Repositório Oficial do Zabbix

```bash
# Baixar o pacote de repositório para Debian 12
wget https://repo.zabbix.com/zabbix/7.0/debian/pool/main/z/zabbix-release/zabbix-release_7.0-1+debian12_all.deb

# Instalar o pacote de repositório
sudo dpkg -i zabbix-release_7.0-1+debian12_all.deb

# Atualizar lista de pacotes
apt update
```

## 📦 5. Instalação dos Pacotes do Zabbix Proxy

```bash
# Instalar o Zabbix Proxy PostgreSQL e scripts SQL
apt install zabbix-proxy-pgsql zabbix-sql-scripts
```

## 📊 6. Importar Schema do Banco

```bash
# Importar estrutura do banco de dados
cat /usr/share/zabbix-sql-scripts/postgresql/proxy.sql | sudo -u zabbix psql zabbix_proxy
```

## ⚙️ 7. Configuração do zabbix_proxy.conf

```bash
# Editar arquivo de configuração com nano
nano /etc/zabbix/zabbix_proxy.conf

# Editar arquivo de configuração com vim
vim /etc/zabbix/zabbix_proxy.conf
```

### Configurações principais no arquivo:

```ini
DBHost=localhost
DBName=zabbix_proxy
DBUser=zabbix
DBPassword=@dmin123

Server=IP_OU_HOSTNAME_DO_SERVER
Hostname=zabbix-proxy  
LogFile=/var/log/zabbix/zabbix_proxy.log
LogFileSize=0
Timeout=4
```
Explicação narrativa dos campos críticos:

- DBHost aponta ao socket local PostgreSQL; mantém latência mínima.
- DBName, DBUser e DBPassword correspondem ao acesso criado na etapa 3.
- Server deve refletir o endereço IPv4/IPv6 ou FQDN do Zabbix Server. Esse valor é usado no handshake inicial.
- Hostname precisa ser igual ao nome cadastrado no frontend do Server; divergências causam recusa de conexão.
- LogFile e LogFileSize=0 preservam logs ilimitados em /var/log/zabbix.

## 🚀 8. Iniciar o Serviço

```bash
# Criar diretório de logs
mkdir -p /var/log/zabbix
chown zabbix:zabbix /var/log/zabbix

# Iniciar e Habilitar o serviço
systemctl start zabbix-proxy
systemctl enable zabbix-proxy

# Verificar status
systemctl status zabbix-proxy

# Monitore logs em tempo real:
sudo journalctl -u zabbix-proxy -f


```

## 🔍 9. Verificar Funcionamento

```bash
# Verificar logs
tail -f /var/log/zabbix/zabbix_proxy.log
```

## 📝 Notas Importantes

- **Segurança**: Altere a senha padrão `@dmin123` por uma senha mais segura
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
4. **Banco de dados** (Informar a senha criada na etapa 3): Teste a conexão MySQL `psql -h localhost -U zabbix -d zabbix_proxy`
