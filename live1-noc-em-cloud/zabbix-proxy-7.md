

[![Zabbix](https://img.shields.io/badge/Zabbix-7.0-red?style=for-the-badge&logo=zabbix)](https://www.zabbix.com/)
[![MySQL](https://img.shields.io/badge/MySQL-8.0-blue?style=for-the-badge&logo=mysql)](https://www.mysql.com/)
[![Ubuntu](https://img.shields.io/badge/Ubuntu-24.04-orange?style=for-the-badge&logo=ubuntu)](https://ubuntu.com/)

# Manual de Instalação do 📊 Zabbix Proxy 7.0 LTS (MariaDB, Proxy e Agent)

## 1. Instalar MariaDB

### Instalação do pacote MariaDB
```bash
apt -y install mariadb-server
```

### Configuração do conjunto de caracteres
```bash
vim /etc/mysql/mariadb.conf.d/50-server.cnf
```

**Instruções para edição:**
1. Execute o comando `:set number` e aperte ENTER
2. Edite a linha 95 do arquivo:

```ini
# se usar UTF-8 de 4 bytes, especifique [utf8mb4]
character-set-server  = utf8mb4
collation-server      = utf8mb4_general_ci
```

### Reiniciar o serviço
```bash
systemctl restart mariadb
```

---

## 2. Configurações Iniciais para MariaDB

### Executar script de segurança
```bash
mysql_secure_installation
```

**Processo de configuração:**

```
NOTE: RUNNING ALL PARTS OF THIS SCRIPT IS RECOMMENDED FOR ALL MariaDB
      SERVERS IN PRODUCTION USE!  PLEASE READ EACH STEP CAREFULLY!

In order to log into MariaDB to secure it, we'll need the current
password for the root user. If you've just installed MariaDB, and
haven't set the root password yet, you should just press enter here.

Enter current password for root (enter for none):
OK, successfully used password, moving on...

Setting the root password or using the unix_socket ensures that nobody
can log into the MariaDB root user without the proper authorisation.

You already have your root account protected, so you can safely answer 'n'.
```

### Opções de configuração:

#### Autenticação unix_socket
```
# Alternar para autenticação [unix_socket] ou não
# A autenticação [unix_socket] está habilitada para o usuário root por padrão, mesmo se você selecionar [No]
Switch to unix_socket authentication [Y/n] n
 ... skipping.
```

#### Senha do root
```
# definir senha root do MariaDB ou não
# A autenticação [unix_socket] está habilitada por padrão, mas
# se você definir a senha root, também é possível fazer login com autenticação por senha.
# se não definir a senha root, apenas o usuário root do SO pode fazer login como usuário root do MariaDB
Change the root password? [Y/n] n
 ... skipping.
```

#### Remover usuários anônimos
```
# remover usuários anônimos
Remove anonymous users? [Y/n] y
 ... Success!
```

#### Desabilitar login root remoto
```
# desabilitar login root remotamente
Disallow root login remotely? [Y/n] y
 ... Success!
```

#### Remover banco de dados de teste
```
# remover banco de dados de teste
Remove test database and access to it? [Y/n] y
 - Dropping test database...
 ... Success!
 - Removing privileges on test database...
 ... Success!
```

#### Recarregar tabelas de privilégios
```
# recarregar tabelas de privilégios
Reload privilege tables now? [Y/n] y
 ... Success!

Cleaning up...

All done!  If you've completed all of the above steps, your MariaDB
installation should now be secure.

Thanks for using MariaDB!
```

---

## 3. Criar um Banco de Dados para o Zabbix Proxy

### Acessar o MariaDB e criar o banco
```bash
mysql
```

```sql
create database zabbix_proxy character set utf8mb4 collate utf8mb4_bin;
grant all privileges on zabbix_proxy.* to zabbix@'localhost' identified by 'password';
exit
```

### Importar esquema do banco
```bash
cat /usr/share/zabbix-sql-scripts/mysql/proxy.sql | mysql -uzabbix -p zabbix_proxy
```

---

## 4. Instalar Zabbix Proxy

### Download e instalação dos repositórios
```bash
wget https://repo.zabbix.com/zabbix/7.0/ubuntu/pool/main/z/zabbix-release/zabbix-release_7.0-1+ubuntu24.04_all.deb
dpkg -i zabbix-release_7.0-1+ubuntu24.04_all.deb
apt update
apt -y install zabbix-proxy-mysql zabbix-sql-scripts
```

---

## 5. Configurar Zabbix Proxy

### Editar arquivo de configuração
```bash
vim /etc/zabbix/zabbix_proxy.conf
```

### Parâmetros de configuração:
```ini
# linha 14: adicionar modo proxy
# 0 = modo ativo, 1 = modo passivo
ProxyMode=0

# linha 32: especificar servidor Zabbix
Server=10.0.0.30

# linha 42: especificar nome do host do Zabbix Proxy
Hostname=prox.srv.world

# linha 160: especificar host do BD
DBHost=localhost

# linha 173: especificar nome do BD
DBName=zabbix_proxy

# linha 188: especificar usuário do BD
DBUser=zabbix

# linha 197: adicionar senha do usuário do BD
DBPassword=password
```

### Iniciar e habilitar o serviço
```bash
systemctl start zabbix-proxy
systemctl enable zabbix-proxy
```

---

## 6. Instalar o Zabbix Agent 2

### Download e instalação
```bash
wget https://repo.zabbix.com/zabbix/7.0/ubuntu/pool/main/z/zabbix-release/zabbix-release_7.0-1+ubuntu24.04_all.deb
dpkg -i zabbix-release_7.0-1+ubuntu24.04_all.deb
apt update
apt -y install zabbix-agent2
```

### Configurar o Agent
```bash
vim /etc/zabbix/zabbix_agent2.conf
```

### Parâmetros de configuração:
```ini
# linha 80: especificar servidor Zabbix
Server=10.0.0.30

# linha 133: especificar servidor Zabbix
ServerActive=10.0.0.30

# linha 144: alterar para o nome do seu host
Hostname=node01.srv.world
```

### Reiniciar o serviço
```bash
systemctl restart zabbix-agent2
```

---

## ✅ Resumo da Instalação

1. **MariaDB** - Banco de dados instalado e configurado com segurança
2. **Banco Zabbix Proxy** - Banco específico criado com usuário dedicado
3. **Zabbix Proxy** - Instalado e configurado em modo ativo
4. **Zabbix Agent 2** - Instalado nos servidores de monitoramento

> **Nota:** Certifique-se de ajustar os endereços IP e nomes de host conforme sua infraestrutura antes de implementar em produção.
