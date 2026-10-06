
## 1. AIRFLOW__CORE__EXECUTION_API_SERVER_URL

Esta configuração define a URL do servidor de API de execução, usado no modo de separação de componentes (API Server).

**Exemplo de uso:**

```python
# No seu docker-compose, está configurado:
AIRFLOW__CORE__EXECUTION_API_SERVER_URL: 'http://airflow-apiserver:8080/execution/'

# Isso permite que o Scheduler se comunique com o API Server
# para executar tasks remotamente
```

**Exemplo de chamada da API:**

```bash
# Listar DAGs
curl -X GET "http://localhost:8080/api/v1/dags" \
  -H "Content-Type: application/json" \
  --user "airflow:airflow"

# Trigger uma DAG
curl -X POST "http://localhost:8080/api/v1/dags/example_dag/dagRuns" \
  -H "Content-Type: application/json" \
  --user "airflow:airflow" \
  -d '{"conf":{}}'
```

## 2. AIRFLOW__CORE__AUTH_MANAGER

Define o gerenciador de autenticação. No seu caso, está usando **FAB (Flask AppBuilder)**.

**Opções disponíveis:**

- `airflow.providers.fab.auth_manager.fab_auth_manager.FabAuthManager` (padrão)
- `airflow.auth.managers.simple.simple_auth_manager.SimpleAuthManager` (mais simples)

**Recursos do FAB:**

- Autenticação baseada em usuário/senha
- Integração com OAuth, LDAP, OpenID
- Controle de acesso baseado em roles (RBAC)

## 3. AIRFLOW__CORE__FERNET_KEY

**Finalidade:** Criptografar senhas/tokens sensíveis no banco de dados (connections, variables).

**Por que precisa:**

- Protege credenciais armazenadas
- Sem ela, o Airflow usa uma chave padrão (INSEGURO para produção)

**Configuração recomendada:**

```yaml
AIRFLOW__CORE__FERNET_KEY: '81HqDtbqAywKSOumSha3BhWNOdQ26slT6K0YaZeZyPs='
```

## 4. AIRFLOW__SCHEDULER__ENABLE_HEALTH_CHECK

**Finalidade:** Habilita endpoint de health check no Scheduler na porta 8974.

**Sem ele:**

- Ainda é possível fazer healthcheck, mas de forma diferente
- O docker-compose usa: `curl http://localhost:8974/health`
- Alternativa sem este parâmetro:

```yaml
healthcheck:
  test: ["CMD-SHELL", 'airflow jobs check --job-type SchedulerJob --hostname "$${HOSTNAME}"']
```

## 5. _PIP_ADDITIONAL_REQUIREMENTS

**Finalidade:** Instalar pacotes Python adicionais durante a inicialização do container.

**Exemplo de sample:**

```yaml
_PIP_ADDITIONAL_REQUIREMENTS: >-
  apache-airflow-providers-amazon==8.0.0
  apache-airflow-providers-google==10.0.0
  pandas==2.0.0
  requests==2.31.0
  boto3==1.28.0
```

**Ou no .env:**

```bash
_PIP_ADDITIONAL_REQUIREMENTS=apache-airflow-providers-amazon apache-airflow-providers-google pandas boto3
```

## 6. Arquivo airflow.cfg - Sample## 7. Níveis de Logs

**Níveis disponíveis (do menos ao mais detalhado):**

1. **CRITICAL** - Erros críticos que impedem o funcionamento
2. **ERROR** - Erros que precisam atenção
3. **WARNING** - Avisos sobre situações incomuns
4. **INFO** - Informações gerais (padrão)
5. **DEBUG** - Informações detalhadas para debug

**Configuração:**

```yaml
# No docker-compose ou .env
AIRFLOW__LOGGING__LOGGING_LEVEL: DEBUG

# Ou no airflow.cfg
[logging]
logging_level = DEBUG
```

## 8. Mudar Login e Senha do Airflow

**Opção 1: Via variáveis de ambiente (.env):**

```bash
_AIRFLOW_WWW_USER_USERNAME=admin
_AIRFLOW_WWW_USER_PASSWORD=minha_senha_segura123
```

**Opção 2: Via CLI após inicialização:**

```bash
docker-compose run airflow-cli users create \
    --username novo_admin \
    --firstname Admin \
    --lastname User \
    --role Admin \
    --email admin@example.com \
    --password nova_senha_segura
```

**Opção 3: Resetar senha de usuário existente:**

```bash
docker-compose run airflow-cli users reset-password \
    --username airflow \
    --password nova_senha
```

## 9. Container CLI - Explicação

**Finalidade:** Container para executar comandos do Airflow CLI de forma interativa.

**Características:**

- Perfil `debug` (só inicia com `--profile debug`)
- Permite executar comandos administrativos
- Não fica rodando continuamente

**Exemplos de uso:**

```bash
# Iniciar o CLI
docker-compose --profile debug run airflow-cli bash

# Dentro do container:
airflow dags list
airflow tasks list example_dag
airflow dags trigger example_dag
airflow users list
airflow connections list

# Ou executar comandos diretos:
docker-compose run airflow-cli airflow dags list
docker-compose run airflow-cli airflow variables set MY_VAR "my_value"
```

## 10. Container Init - Funcionamento Detalhado

**Finalidade:** Container de inicialização único que prepara o ambiente.

**O que ele faz:**

1. **Verifica recursos do sistema** (CPU, memória, disco)
2. **Cria diretórios necessários** (logs, dags, plugins, config)
3. **Executa migrations do banco** (_AIRFLOW_DB_MIGRATE)
4. **Cria usuário admin inicial** (_AIRFLOW_WWW_USER_CREATE)
5. **Ajusta permissões** dos volumes para AIRFLOW_UID
6. **Gera arquivo de configuração padrão**

**Quando deve ser iniciado:**

- **Primeira vez** que sobe o ambiente
- **Após upgrades** de versão do Airflow
- **Após mudanças** no esquema do banco

**Como é iniciado:**

```bash
# Automaticamente com docker-compose up
docker-compose up -d

# Manualmente apenas o init
docker-compose up airflow-init
```

**Sobre healthcheck e dependências:**

Sim! Os outros containers dependem do sucesso do init:

```yaml
depends_on:
  airflow-init:
    condition: service_completed_successfully
```

Isso significa:

- Os containers **só iniciam** após o init completar com sucesso (exit code 0)
- **Não precisa** de healthcheck no init porque ele roda uma vez e termina
- Se o init falhar, os outros containers não iniciam

**Fluxo de inicialização:**

```
1. postgres inicia e passa no healthcheck
2. airflow-init roda e completa com sucesso
3. airflow-apiserver, scheduler, dag-processor e triggerer iniciam
```

**Logs importantes do init:**

```bash
# Ver logs do init
docker-compose logs airflow-init

# Reiniciar apenas o init
docker-compose up --force-recreate airflow-init
```

---

**Dicas finais:**

- Em **produção**, sempre defina `AIRFLOW__CORE__FERNET_KEY`
- Mude `AIRFLOW__CORE__LOAD_EXAMPLES: 'false'` em produção
- Use `_PIP_ADDITIONAL_REQUIREMENTS` com versões fixas
- Configure logs externos para persistência
- Implemente backup do volume `postgres-db-volume`

Precisa de mais detalhes sobre algum desses pontos?