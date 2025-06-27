# Manual de Instalação do 📊 Zabbix Proxy 7.0 LTS

[![Zabbix](https://img.shields.io/badge/Zabbix-7.0-red?style=for-the-badge&logo=zabbix)](https://www.zabbix.com/)
[![MySQL](https://img.shields.io/badge/MySQL-8.0-blue?style=for-the-badge&logo=mysql)](https://www.mysql.com/)
[![Ubuntu](https://img.shields.io/badge/Ubuntu-24.04-orange?style=for-the-badge&logo=ubuntu)](https://ubuntu.com/)



1. Instalar as dependências:
# Dependências
apt-get install -y wget curl net-tools 

#Configurar a timezone
timedatectl set-timezone America/Sao_Paulo

2. Adicionar Repositório Oficial do Zabbix

# Baixar o pacote de repositório para Ubuntu 24.04
wget https://repo.zabbix.com/zabbix/7.0/ubuntu/pool/main/z/zabbix-release/zabbix-release_7.0-2+ubuntu24.04_all.deb

# Instalar o pacote de repositório
dpkg -i zabbix-release_7.0-2+ubuntu24.04_all.deb

# Atualizar lista de pacotes
apt update

3. Instalar Zabbix Proxy e Dependências

# Instalar o Zabbix Proxy MySQL e scripts SQL
apt install zabbix-proxy-mysql zabbix-sql-scripts

# Instalar MySQL Server se ainda não estiver instalado
apt install mysql-server

4. Verificar Instalação

# Verificar se os pacotes foram instalados corretamente
dpkg -l | grep zabbix

# Verificar versão do Zabbix Proxy
zabbix_proxy --version

5. Configurar MySQL

# Configurar MySQL (se primeira instalação)
mysql_secure_installation

# Conectar ao MySQL
mysql -u root -p

6. Criar Banco de Dados para o Proxy

-- No MySQL
CREATE DATABASE zabbix_proxy CHARACTER SET utf8mb4 COLLATE utf8mb4_bin;
CREATE USER 'zabbix_proxy'@'localhost' IDENTIFIED BY '@dmin123';
GRANT ALL PRIVILEGES ON zabbix_proxy.* TO 'zabbix_proxy'@'localhost';
FLUSH PRIVILEGES;
EXIT;

7. Importar Schema do Banco

# Importar estrutura do banco de dados
cat /usr/share/zabbix-sql-scripts/mysql/proxy.sql | mysql -u zabbix_proxy -p zabbix_proxy

8. Configurar o Zabbix Proxy

# Editar arquivo de configuração
nano /etc/zabbix/zabbix_proxy.conf

Configurações principais:

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

# Configurações de performance
ProxyOfflineBuffer=24
ConfigFrequency=3600
DataSenderFrequency=1

# Log
LogFile=/var/log/zabbix/zabbix_proxy.log
LogFileSize=10

9. Iniciar o Serviço

# Criar diretório de logs
mkdir -p /var/log/zabbix
chown zabbix:zabbix /var/log/zabbix

# Habilitar e iniciar o serviço
systemctl enable zabbix-proxy
systemctl start zabbix-proxy

# Verificar status
systemctl status zabbix-proxy


10. Verificar Funcionamento

# Verificar logs
tail -f /var/log/zabbix/zabbix_proxy.log

# Verificar se está escutando na porta
ss -tlnp | grep 10051
