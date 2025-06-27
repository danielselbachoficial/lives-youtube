# Configuração de Acesso API do Grafana com Zabbix

Este guia detalha os passos para configurar um user role, user group e user no Zabbix para integração com a API do Grafana.

---

## 1. Criar User Role para API do Grafana

**Caminho:** `Users` > `User roles` > `Create user role`

### Detalhes do Role:

*   **Name:** `grafana.api`
*   **User Type:** `user`

### Permissões:

#### Access to UI elements
*   [x] `Dashboards`

#### Monitoring
*   [x] `Latest data`
*   [x] `Hosts`

#### Read-write access to services
*   `none`

#### Read-only access to services
*   `all`

#### Access to modules
*   [x] `Graph`
*   [x] `Host availability`
*   [x] `Item history`
*   [x] `Item navigator`
*   [x] `Item value`
*   [x] `Problems`

#### Default access to new modules
*   [x]

#### Access to API
*   **Enabled:** [x]

#### API methods - Allow list

```
host.get
hostgroup.get
template.get
item.get
problem.get
trigger.get
history.get
trend.get
event.get
proxy.get
graph.get
hostinterface.get
user.get
usermacro.get
```

#### Access to actions
*   [x] `Invoke \"Execute now\" on read-only hosts`

Clique em \"Add\".

---

## 2. Criar Grupo de Usuários

**Caminho:** `Users` > `User groups` > `Create user group`

### Detalhes do Grupo:

*   **name:** `Grafana Viewers`
*   **Permissions:** `read`

### Permissões de Templates:

*   `Templates`
*   `Templates/Applications`
*   `Templates/Cloud`
*   `Templates/Databases`
*   `Templates/Network devices`
*   `Templates/Operating systems`
*   `Templates/Power`
*   `Templates/SAN`
*   `Templates/Server hardware`
*   `Templates/Telephony`
*   `Templates/Video surveillance`
*   `Templates/Virtualization`

### Permissões de Hosts:

*   `hosts permissions: Selecionar os hosts`

---

## 3. Criar Usuário

**Caminho:** `Users` > `User` > `Create User`

### Detalhes do Usuário:

*   **username:** `<Seu_Nome_de_Usuário>`
*   **password:** `<Sua_Senha_Segura>`
*   **permissions:** `selecionar a user role criada`

---
