# 📊 NOC em Cloud - Manual Completo Grafana 12.0.1

> **Manual Técnico**: Implementação completa de visualização com Grafana 12.0.1 no Ubuntu Server 24.04 LTS

[![Grafana](https://img.shields.io/badge/Grafana-12.0.1-orange?style=for-the-badge&logo=grafana)](https://grafana.com/)
[![PostgreSQL](https://img.shields.io/badge/PostgreSQL-16-blue?style=for-the-badge&logo=postgresql)](https://www.postgresql.org/)
[![Ubuntu](https://img.shields.io/badge/Ubuntu-24.04_LTS-orange?style=for-the-badge&logo=ubuntu)](https://ubuntu.com/)

---

## Índice

1. [Introdução e Contexto](#1-introdução-e-contexto)
2. [Requisitos de Sistema e Dimensionamento](#2-requisitos-de-sistema-e-dimensionamento)
3. [Preparação do Ambiente Ubuntu Server 24.04 LTS](#3-preparação-do-ambiente-ubuntu-server-2404-lts)
4. [Instalação do Grafana via Repositório Oficial](#4-instalação-do-grafana-via-repositório-oficial)
5. [Inicialização, Ativação e Verificação do Serviço](#5-inicialização-ativação-e-verificação-do-serviço)
6. [Configuração de Firewall com UFW](#6-configuração-de-firewall-com-ufw)
7. [Boas Práticas Básicas de Segurança](#7-boas-práticas-básicas-de-segurança)
8. [Testes de Desempenho e Monitoramento Inicial](#8-testes-de-desempenho-e-monitoramento-inicial)
9. [Resolução de Problemas Mais Comuns](#9-resolução-de-problemas-mais-comuns)
10. [Considerações Finais e Próximos Passos](#10-considerações-finais-e-próximos-passos)

---

## 1. Introdução e Contexto

Instalar o Grafana em uma máquina virtual Ubuntu Server 24.04 LTS que possua apenas um endereço IP público expõe o serviço diretamente à internet. 

A estrutura deste manual privilegia uma narrativa completa, conduzindo o leitor desde a criação do ambiente até a verificação pós-instalação, sem pular etapas implícitas. Todo comando foi extraído ou validado na [documentação oficial do Grafana](https://grafana.com/docs/grafana/latest/setup-grafana/installation/debian/) e testado em Ubuntu Server 24.04 LTS (Noble Numbat).

---

## 2. Requisitos de Sistema e Dimensionamento

Ainda que o Grafana seja leve, dimensionar corretamente CPU, RAM e armazenamento evita gargalos.

| Perfil de Uso | vCPU | RAM | Armazenamento | Conexões Simultâneas Esperadas | Observação de Desempenho |
|---------------|------|-----|--------------|--------------------------------|--------------------------|
| Demonstração básica (dashboard simples) | 1 | 1 GB | 10 GB SSD | até 10 | Resposta em <150 ms, picos aceitos |
| Aula intermediária (dashboard complexo) | 2 | 2 GB | 20 GB SSD | 10–25 | Picos de CPU ~40%, memória estável |
| Turma grande / futuros plugins | 4 | 4 GB | 20 GB+ SSD | 25+ | Gráfico fluído mesmo sob stress |

### Requisitos Mínimos do Sistema (Ubuntu 24.04 LTS)
- **Sistema Operacional**: Ubuntu Server 24.04 LTS (64-bit)
- **Memória**: Mínimo 255 MB, recomendado 512 MB+
- **CPU**: Qualquer arquitetura suportada pelo Ubuntu (x86_64, ARM64)
- **Armazenamento**: Mínimo 1 GB de espaço livre
- **Rede**: Acesso à internet para download de pacotes

---

## 3. Preparação do Ambiente Ubuntu Server 24.04 LTS

A etapa de preparação garante que o sistema operacional esteja limpo, atualizado e com dependências mínimas satisfeitas. Apesar de breves, esses passos são cruciais para evitar conflitos de pacotes ou bibliotecas ausentes.

### 3.1 Atualização do Sistema

Atualize os índices de pacotes e aplique correções pendentes:

```bash
sudo apt update && sudo apt upgrade -y
```

### 3.2 Instalação de Dependências

Instale as dependências essenciais exigidas pela documentação oficial:

```bash
sudo apt install -y software-properties-common apt-transport-https wget
```

### 3.3 Configuração de Swap (Opcional)

Para VMs com pouca memória, considere ativar swap:

```bash
sudo fallocate -l 1G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```

Configure o parâmetro `vm.swappiness` para otimizar o uso de swap:

```bash
echo 'vm.swappiness=10' | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

### 3.4 Sincronização de Horário

Garanta que o sistema tenha horário sincronizado:

```bash
sudo timedatectl set-ntp true
timedatectl status
```

---

## 4. Instalação do Grafana via Repositório Oficial

A Grafana Labs mantém repositórios APT assinados que facilitam a instalação e a atualização futura. A sequência a seguir reflete exatamente a recomendação oficial para sistemas baseados em Debian/Ubuntu.

### 4.1 Importação da Chave GPG

```bash
sudo mkdir -p /etc/apt/keyrings/
wget -q -O - https://apt.grafana.com/gpg.key | gpg --dearmor | sudo tee /etc/apt/keyrings/grafana.gpg > /dev/null
```

### 4.2 Adição do Repositório

Adicione o repositório oficial para pacotes estáveis:

```bash
echo "deb [signed-by=/etc/apt/keyrings/grafana.gpg] https://apt.grafana.com stable main" | sudo tee -a /etc/apt/sources.list.d/grafana.list
```

### 4.3 Instalação do Grafana

Atualize o índice de pacotes e instale o Grafana:

```bash
sudo apt update
sudo apt install grafana
```

### 4.4 Verificação da Instalação

Confirme que o pacote foi instalado corretamente:

```bash
grafana --version
```

---

## 5. Inicialização, Ativação e Verificação do Serviço

Após instalado, o serviço precisa ser iniciado manualmente uma primeira vez e, em seguida, habilitado para inicializar com o sistema.

### 5.1 Inicialização do Serviço

```bash
sudo systemctl daemon-reload
sudo systemctl start grafana-server
sudo systemctl enable grafana-server.service
```

### 5.2 Verificação do Status

Confirme que o status é **active (running)**:

```bash
sudo systemctl status grafana-server
```

A saída deve mostrar algo similar a:

```
● grafana-server.service - Grafana instance
     Loaded: loaded (/lib/systemd/system/grafana-server.service; enabled; vendor preset: enabled)
     Active: active (running) since [timestamp]
```

### 5.3 Verificação de Conectividade

Verifique se o serviço está escutando na porta 3000:

```bash
sudo ss -tlnp | grep :3000
```

### 5.4 Primeiro Acesso

Quando a saída indicar que o servidor está escutando na porta 3000/tcp, abra o navegador e acesse:

```
http://SEU_IP_PUBLICO:3000
```

**Credenciais padrão:**
- **Usuário**: `admin`
- **Senha**: `admin`

> **Importante**: O sistema solicitará a alteração da senha no primeiro login.

---

## 6. Configuração de Firewall com UFW

O Ubuntu Server 24.04 LTS vem com UFW (Uncomplicated Firewall) que oferece uma interface simplificada para iptables. Por padrão, uma VM recém-criada pode ter todas as portas abertas, sendo necessário configurar regras específicas.

### 6.1 Verificação do Status do UFW

```bash
sudo ufw status
```

### 6.2 Configuração de Regras Básicas

1. **Permitir SSH (essencial para não perder acesso remoto):**

```bash
sudo ufw allow ssh
```

2. **Permitir porta 3000 para Grafana:**

```bash
sudo ufw allow 3000/tcp
```

3. **Ativar o firewall:**

```bash
sudo ufw enable
```

### 6.3 Verificação das Regras

```bash
sudo ufw status verbose
```

A saída deve mostrar:

```
Status: active
Logging: on (low)
Default: deny (incoming), allow (outgoing), disabled (routed)
New profiles: skip

To                         Action      From
--                         ------      ----
22/tcp                     ALLOW IN    Anywhere
3000/tcp                   ALLOW IN    Anywhere
22/tcp (v6)                ALLOW IN    Anywhere (v6)
3000/tcp (v6)              ALLOW IN    Anywhere (v6)
```

### 6.4 Regras Opcionais para Maior Segurança

Para limitar acesso SSH a IPs específicos:

```bash
sudo ufw delete allow ssh
sudo ufw allow from SEU_IP_ADMINISTRATIVO to any port 22
```

---

## 7. Boas Práticas Básicas de Segurança

Mesmo sem HTTPS, há rotinas simples que reduzem vetores de ataque comuns:

### 7.1 Configurações Básicas de Segurança

1. **Alterar senha padrão imediatamente** após primeiro login
2. **Desabilitar registro público** editando `/etc/grafana/grafana.ini`:

```bash
sudo nano /etc/grafana/grafana.ini
```

Localize e modifique:

```ini
[users]
# disable user signup / registration
allow_sign_up = false
```

3. **Reiniciar o serviço** após alterações:

```bash
sudo systemctl restart grafana-server
```

### 7.2 Monitoramento de Logs

Monitore logs em tempo real durante a aula:

```bash
sudo journalctl -u grafana-server -f
```

### 7.3 Backup de Configuração

Faça backup da configuração antes da aula:

```bash
sudo cp /etc/grafana/grafana.ini /etc/grafana/grafana.ini.backup
sudo tar -czf grafana-backup-$(date +%Y%m%d).tar.gz /var/lib/grafana/
```

---

## 8. Testes de Desempenho e Monitoramento Inicial

Para confirmar se a VM suporta a carga esperada, execute breves testes de stress enquanto observa CPU e RAM.

### 8.1 Instalação de Ferramentas de Monitoramento

```bash
sudo apt install -y htop stress-ng
```

### 8.2 Teste de Carga Básico

1. **Inicie o monitor de recursos:**

```bash
htop
```

2. **Em outro terminal, execute teste de stress:**

```bash
stress-ng --cpu 1 --vm 1 --vm-bytes 500M --timeout 60s
```

3. **Monitore o desempenho** navegando simultaneamente por dashboards

### 8.3 Verificação de Recursos

```bash
# Verificar uso de memória
free -h

# Verificar uso de disco
df -h

# Verificar processos do Grafana
ps aux | grep grafana
```

---

## 9. Resolução de Problemas Mais Comuns

### 9.1 Tabela de Diagnóstico

| Sintoma | Diagnóstico Sugerido | Solução |
|---------|---------------------|---------|
| Navegador não encontra IP:3000 | Firewall bloqueando ou serviço inativo | Verificar UFW (`sudo ufw status`) e status do serviço |
| `grafana-server` falha ao iniciar | Erro de configuração ou dependências | Verificar logs: `sudo journalctl -u grafana-server -n 50` |
| Lentidão ou timeouts | Recursos insuficientes | Verificar RAM/CPU com `htop`, considerar upgrade |
| Erro de permissão | Problemas de ownership | `sudo chown -R grafana:grafana /var/lib/grafana` |

### 9.2 Comandos de Diagnóstico

```bash
# Verificar status detalhado do serviço
sudo systemctl status grafana-server -l

# Verificar logs específicos
sudo tail -f /var/log/grafana/grafana.log

# Verificar conectividade de rede
sudo netstat -tlnp | grep grafana

# Verificar configuração
sudo grafana-cli admin reset-admin-password admin
```

### 9.3 Reinicialização Completa

Em caso de problemas persistentes:

```bash
sudo systemctl stop grafana-server
sudo systemctl daemon-reload
sudo systemctl start grafana-server
sudo systemctl status grafana-server
```

---

## 10. Considerações Finais e Próximos Passos

### 10.1 Resumo da Instalação

A instalação descrita aqui entrega um Grafana funcional em questão de minutos. Ao conservar estritamente as recomendações oficiais, a probabilidade de incompatibilidades futuras é mínima, e o caminho para upgrades se mantém simples.

### 10.2 Comandos de Manutenção

```bash
# Atualizar Grafana
sudo apt update && sudo apt upgrade grafana

# Verificar versão instalada
grafana --version

# Parar serviço após aula (segurança)
sudo systemctl stop grafana-server
```

### 10.3 Próximos Passos para Produção

Para ambientes de produção, considere:

- **HTTPS com certificados SSL/TLS**
- **Proxy reverso (Nginx/Apache)**
- **Autenticação externa (LDAP/OAuth)**
- **Backup automatizado**
- **Monitoramento de métricas do próprio Grafana**
- **Rate limiting e proteção DDoS**

### 10.4 Recursos Adicionais

- [Documentação Oficial do Grafana](https://grafana.com/docs/grafana/latest/)
- [Grafana Community](https://community.grafana.com/)
- [Grafana GitHub Repository](https://github.com/grafana/grafana)

---
