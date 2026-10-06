# FAB Auth Manager - Resumo

## O que é

**FAB Auth Manager** (Flask-AppBuilder Auth Manager) é o sistema de **autenticação e autorização robusto** do Airflow, baseado no framework Flask-AppBuilder.

## Principais Características

### 🔐 Autenticação

- Login com **usuário e senha** (armazenados no banco)
- Suporte a **OAuth** (Google, GitHub, Azure AD, etc.)
- Integração com **LDAP/Active Directory**
- Suporte a **OpenID Connect**
- **Multi-tenancy** (múltiplos domínios)

### 👥 Autorização (RBAC - Role-Based Access Control)

Sistema de **roles (papéis) e permissões** granular:

**Roles Padrões:**

- **Admin** - Acesso total ao sistema
- **Op** (Operator) - Pode executar e gerenciar DAGs
- **User** - Pode visualizar e executar DAGs específicos
- **Viewer** - Apenas visualização (somente leitura)
- **Public** - Acesso público limitado

**Permissões Granulares:**

- Por **DAG** (usuário X só vê DAGs Y e Z)
- Por **recurso** (Connections, Variables, Pools)
- Por **ação** (can_read, can_edit, can_create, can_delete)

### 🎯 Funcionalidades Principais

1. **Interface Web Completa**
    
    - Gerenciamento visual de usuários
    - Criação e edição de roles customizadas
    - Atribuição de permissões específicas
2. **CLI para Usuários**
    
    ```bash
    airflow users create --username admin --role Admin --email admin@example.com
    airflow users list
    airflow users delete --username user
    ```
    
3. **Auditoria**
    
    - Log de todas as ações dos usuários
    - Rastreamento de quem fez o quê e quando
4. **Segurança Avançada**
    
    - Senhas com hash seguro (bcrypt)
    - Sessões com timeout configurável
    - Proteção CSRF
    - Rate limiting

## Quando Usar

✅ **Use FAB Auth Manager quando:**

- Ambiente de **produção** ou **staging**
- Múltiplos usuários com diferentes níveis de acesso
- Necessidade de **auditoria** e compliance
- Integração com **SSO** (Single Sign-On)
- Controle **granular** de permissões por DAG/recurso

❌ **Não precisa quando:**

- Desenvolvimento solo/local
- Prototipagem rápida
- Ambiente totalmente isolado e confiável

## Diferença do Simple Auth Manager

|Recurso|FAB Auth Manager|Simple Auth Manager|
|---|---|---|
|**Gerenciamento de usuários**|Interface web + CLI|Apenas configuração|
|**Senhas**|Definidas pelo admin|Auto-geradas|
|**RBAC**|Completo e customizável|Básico (4 roles fixas)|
|**OAuth/LDAP**|✅ Sim|❌ Não|
|**Auditoria**|✅ Completa|❌ Não|
|**Produção**|✅ Recomendado|❌ Não recomendado|

## Instalação no Airflow 3

```bash
# Instalar o provider
pip install apache-airflow-providers-fab

# Configurar
export AIRFLOW__CORE__AUTH_MANAGER=airflow.providers.fab.auth_manager.fab_auth_manager.FabAuthManager

# Migrar banco de dados
airflow fab-db migrate

# Criar admin
airflow users create --username admin --role Admin --password admin --email admin@example.com
```

## Resumo em Uma Frase

**FAB Auth Manager é o sistema completo e profissional de segurança do Airflow, oferecendo controle total sobre quem pode acessar o quê, com suporte a integrações corporativas como LDAP e SSO.**