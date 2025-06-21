# 📊 NOC em Cloud - Manual Completo Grafana 12.0.1

> **Manual Técnico**: Implementação completa de visualização com Grafana 12.0.1 + PostgreSQL no Ubuntu Server 24.04 LTS

[![Grafana](https://img.shields.io/badge/Grafana-12.0.1-orange?style=for-the-badge&logo=grafana)](https://grafana.com/)
[![PostgreSQL](https://img.shields.io/badge/PostgreSQL-16-blue?style=for-the-badge&logo=postgresql)](https://www.postgresql.org/)
[![Ubuntu](https://img.shields.io/badge/Ubuntu-24.04_LTS-orange?style=for-the-badge&logo=ubuntu)](https://ubuntu.com/)
[![Nginx](https://img.shields.io/badge/Nginx-1.24-green?style=for-the-badge&logo=nginx)](https://nginx.org/)

---

## 📑 **Índice**

- [🎯 Objetivo e Pré-requisitos](#-objetivo-e-pré-requisitos)
- [🚀 Preparação do Ambiente](#-preparação-do-ambiente)
- [🔧 Instalação dos Componentes](#-instalação-dos-componentes)
- [🗄️ Configuração do Banco de Dados](#️-configuração-do-banco-de-dados)
- [⚙️ Configuração do Grafana Server](#️-configuração-do-grafana-server)
- [🌐 Configuração do Nginx](#-configuração-do-nginx)
- [🚀 Inicialização dos Serviços](#-inicialização-dos-serviços)
- [✅ Verificações Pós-Instalação](#-verificações-pós-instalação)
- [🌐 Configuração Web (Setup Inicial)](#-configuração-web-setup-inicial)
- [🔐 Primeiro Acesso](#-primeiro-acesso)
- [📊 Configuração de Data Sources](#-configuração-de-data-sources)
- [🔥 Configuração de Firewall](#-configuração-de-firewall)
- [🔧 Configurações Avançadas](#-configurações-avançadas)
- [🎯 Próximos Passos Recomendados](#-próximos-passos-recomendados)
- [📚 Recursos Úteis](#-recursos-úteis)
- [🐛 Troubleshooting Comum](#-troubleshooting-comum)

---

## 🎯 **Objetivo e Pré-requisitos**

### **Objetivo**
Implementar uma plataforma de visualização Grafana 12.0.1 completa para NOC (Network Operations Center) em ambiente cloud, proporcionando dashboards avançados, análise de dados em tempo real e recursos de alerting modernos.

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
 80, 443, 3000, 5432 
|

### **Arquitetura da Solução**
┌─────────────────┐ ┌─────────────────┐ ┌─────────────────┐ │ Grafana Web │ │ Grafana Server │ │ PostgreSQL │ │ (Nginx Proxy) │◄──►│ (Backend) │◄──►│ (Database) │ └─────────────────┘ └─────────────────┘ └─────────────────┘ │ │ │ ▼ ▼ ▼ Port 80/443 Port 3000 Port 5432


---

## 🚀 **Preparação do Ambiente**

### **a) Instalar Dependências Básicas**
```bash
# Atualizar sistema e instalar ferramentas essenciais
sudo apt update && sudo apt upgrade -y
sudo apt install -y wget curl gnupg2 software-properties-common apt-transport-https lsb-release ca-certificates
