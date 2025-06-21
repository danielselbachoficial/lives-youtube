# 📊 NOC em Cloud - Manual Completo Grafana 11.3

> **Manual Técnico**: Implementação completa de visualização com Grafana 11.3 + PostgreSQL no Debian 12

[![Grafana](https://img.shields.io/badge/Grafana-11.3-orange?style=for-the-badge&logo=grafana)](https://grafana.com/)
[![PostgreSQL](https://img.shields.io/badge/PostgreSQL-15-blue?style=for-the-badge&logo=postgresql)](https://www.postgresql.org/)
[![Debian](https://img.shields.io/badge/Debian-12-red?style=for-the-badge&logo=debian)](https://www.debian.org/)
[![Nginx](https://img.shields.io/badge/Nginx-1.22-green?style=for-the-badge&logo=nginx)](https://nginx.org/)

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

---

## 🎯 **Objetivo e Pré-requisitos**

### **Objetivo**
Implementar uma plataforma de visualização Grafana completa para NOC (Network Operations Center) em ambiente cloud, proporcionando dashboards avançados e análise de dados em tempo real.

### **Pré-requisitos**

| Componente | Especificação |
|------------|---------------|
| **Sistema Operacional** | Debian 12 (Bookworm) |
| **RAM** | Mínimo 2GB (Recomendado 4GB+) |
| **CPU** | Mínimo 2 vCPUs |
| **Storage** | Mínimo 20GB (Recomendado 50GB+) |
| **Acesso** | Root ou usuário com sudo |
| **Conectividade** | Internet para download de pacotes |
| **Portas** | 80, 443, 3000, 5432 |

### **Arquitetura da Solução**
