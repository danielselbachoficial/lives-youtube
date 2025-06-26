📊 NOC em Cloud - Manual Completo Zabbix Proxy 7.0 LTS
Manual Técnico: Implementação completa de Zabbix Proxy 7.0 LTS + MySQL no Ubuntu Server 24.04 LTS


www.zabbix.com
 

www.mysql.com
 

ubuntu.com
 

www.zabbix.com

📑 Índice
🎯 Objetivo e Pré-requisitos
🚀 Preparação do Ambiente
🔧 Instalação dos Componentes
🗄️ Configuração do Banco de Dados
⚙️ Configuração do Zabbix Proxy
🚀 Inicialização dos Serviços
✅ Verificações Pós-Instalação
🔗 Configuração no Zabbix Server
🔥 Configuração de Firewall
📊 Monitoramento e Troubleshooting
🎯 Objetivo e Pré-requisitos
Objetivo
Implementar um Zabbix Proxy para distribuir a carga de monitoramento e melhorar a performance em ambientes distribuídos, proporcionando coleta de dados local e sincronização com o Zabbix Server central.

Pré-requisitos
Componente	Especificação
Sistema Operacional	Ubuntu Server 24.04 LTS
RAM	Mínimo 1GB (Recomendado 2GB+)
CPU	Mínimo 1 vCPU (Recomendado 2+)
Storage	Mínimo 10GB (Recomendado 20GB+)
Acesso	Root ou usuário com sudo
Conectividade	Internet + acesso ao Zabbix Server
Portas	10051 (proxy), 3306 (mysql)
Arquitetura da Solução
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│  Zabbix Server  │◄──►│  Zabbix Proxy   │◄──►│     MySQL       │
│   (Central)     │    │   (Collector)   │    │   (Local DB)    │
└─────────────────┘    └─────────────────┘    └─────────────────┘
         │                     │                      │
         │                     │                      │
         ▼                     ▼                      ▼
     Port 10051             Port 10051             Port 3306
                                │
                                ▼
                        ┌─────────────────┐
                        │   Monitored     │
                        │   Hosts/Agents  │
                        └─────────────────┘
