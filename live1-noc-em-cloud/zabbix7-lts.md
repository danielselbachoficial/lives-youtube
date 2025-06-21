📊 NOC em Cloud - Manual Completo Zabbix 7.0
Manual Técnico: Implementação completa de monitoramento com Zabbix 7.0 + PostgreSQL no Debian 12

📑 Índice
Objetivo e Pré-requisitos
Preparação do Ambiente
Instalação dos Componentes
Configuração do Banco de Dados
Configuração do Zabbix Server
Configuração do Nginx
Configuração do PHP
Inicialização dos Serviços
Verificações Pós-Instalação
Configuração Web (Setup Wizard)
Primeiro Acesso
Configuração de Firewall
Troubleshooting
Scripts de Automação
Próximos Passos
🎯 1. Objetivo e Pré-requisitos
Objetivo
Implementar um sistema de monitoramento Zabbix completo para NOC (Network Operations Center) em ambiente cloud, proporcionando visibilidade total da infraestrutura.

Pré-requisitos
Componente	Especificação
Sistema Operacional	Debian 12 (Bookworm)
RAM	Mínimo 2GB (Recomendado 4GB+)
CPU	Mínimo 2 vCPUs
Storage	Mínimo 20GB (Recomendado 50GB+)
Acesso	Root ou usuário com sudo
Conectividade	Internet para download de pacotes
Portas	80, 443, 8080, 10051, 5432
Arquitetura da Solução
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Zabbix Web    │    │  Zabbix Server  │    │   PostgreSQL    │
│   (Nginx+PHP)   │◄──►│   (Backend)     │◄──►│   (Database)    │
└─────────────────┘    └─────────────────┘    └─────────────────┘
         │                       │                       │
         ▼                       ▼                       ▼
    Port 8080              Port 10051              Port 5432
