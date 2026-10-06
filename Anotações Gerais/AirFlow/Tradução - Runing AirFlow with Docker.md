Fonte: https://airflow.apache.org/docs/apache-airflow/stable/howto/docker-compose/index.html#

---
# Executando o Airflow no Docker

Este guia de início rápido permitirá que você coloque rapidamente o Airflow em funcionamento com o CeleryExecutor no Docker.

**Atenção**: Este procedimento pode ser útil para aprendizado e exploração. No entanto, adaptá-lo para uso em situações do mundo real pode ser complicado e o arquivo docker compose não fornece nenhuma garantia de segurança necessária para sistemas de produção. Fazer mudanças neste procedimento exigirá conhecimento especializado em Docker & Docker Compose, e a comunidade do Airflow pode não conseguir ajudá-lo.

Por esse motivo, recomendamos usar o Kubernetes com o Helm Chart Oficial da Comunidade Airflow quando você estiver pronto para executar o Airflow em produção.

## Antes de Começar

Este procedimento pressupõe familiaridade com Docker e Docker Compose. Se você ainda não trabalhou com essas ferramentas, reserve um momento para passar pelo Docker Quick Start (especialmente a seção sobre Docker Compose) para se familiarizar com o funcionamento delas.

Siga estas etapas para instalar as ferramentas necessárias, caso ainda não tenha feito isso:

1. Instale o Docker Community Edition (CE) em sua estação de trabalho. Dependendo do seu sistema operacional, pode ser necessário configurar o Docker para usar pelo menos 4.00 GB de memória para que os containers do Airflow funcionem adequadamente. Consulte a seção Resources na documentação do Docker for Windows ou Docker for Mac para mais informações.
    
2. Instale o Docker Compose v2.14.0 ou mais recente em sua estação de trabalho.
    

Versões mais antigas do `docker-compose` não suportam todos os recursos exigidos pelo arquivo `docker-compose.yaml` do Airflow, portanto, verifique se sua versão atende aos requisitos mínimos de versão.

**Dica**: A quantidade padrão de memória disponível para o Docker no macOS geralmente não é suficiente para colocar o Airflow em funcionamento. Se não houver memória suficiente alocada, isso pode levar à reinicialização contínua do webserver. Você deve alocar pelo menos 4GB de memória para o Docker Engine (idealmente 8GB).

Você pode verificar se tem memória suficiente executando este comando:

```bash
docker run --rm "debian:bookworm-slim" bash -c 'numfmt --to iec $(echo $(($(getconf _PHYS_PAGES) * $(getconf PAGE_SIZE))))'
```

**Aviso**: Alguns sistemas operacionais (Fedora, ArchLinux, RHEL, Rocky) introduziram recentemente mudanças no Kernel que resultam no Airflow no Docker Compose consumindo 100% da memória quando executado dentro da implementação Docker da comunidade mantida pelas equipes do sistema operacional. Parece que instalar o Docker Desktop no Linux resolve o problema e permite executar o Airflow sem problemas.

## Obtendo o docker-compose.yaml

Para implantar o Airflow no Docker Compose, você deve obter o docker-compose.yaml:

```bash
curl -LfO 'https://airflow.apache.org/docs/apache-airflow/3.1.3/docker-compose.yaml'
```

**Importante**: A partir de julho de 2023, o Compose V1 parou de receber atualizações. Aconselhamos fortemente a atualização para uma versão mais recente do Docker Compose, o `docker-compose.yaml` fornecido pode não funcionar corretamente dentro do Compose V1.

Este arquivo contém várias definições de serviço:

- `airflow-scheduler` - O scheduler monitora todas as tarefas e DAGs, depois dispara as instâncias de tarefa uma vez que suas dependências sejam concluídas.
- `airflow-dag-processor` - O processador de DAG analisa arquivos DAG.
- `airflow-api-server` - O servidor de API está disponível em `http://localhost:8080`.
- `airflow-worker` - O worker que executa as tarefas dadas pelo scheduler.
- `airflow-triggerer` - O triggerer executa um loop de eventos para tarefas diferíveis.
- `airflow-init` - O serviço de inicialização.
- `postgres` - O banco de dados.
- `redis` - O broker redis que encaminha mensagens do scheduler para o worker.

Opcionalmente, você pode habilitar o flower adicionando a opção `--profile flower`, por exemplo `docker compose --profile flower up`, ou especificando-o explicitamente na linha de comando, por exemplo `docker compose up flower`.

- `flower` - O aplicativo flower para monitorar o ambiente. Está disponível em `http://localhost:5555`.

Todos esses serviços permitem que você execute o Airflow com CeleryExecutor.

Alguns diretórios no container são montados, o que significa que seus conteúdos são sincronizados entre seu computador e o container:

- `./dags` - você pode colocar seus arquivos DAG aqui.
- `./logs` - contém logs da execução de tarefas e do scheduler.
- `./config` - você pode adicionar um analisador de log personalizado ou adicionar `airflow_local_settings.py` para configurar a política de cluster.
- `./plugins` - você pode colocar seus plugins personalizados aqui.

Este arquivo usa a imagem mais recente do Airflow (apache/airflow). Se você precisar instalar uma nova biblioteca Python ou biblioteca do sistema, pode construir sua própria imagem.

## Inicializando o Ambiente

Antes de iniciar o Airflow pela primeira vez, você precisa preparar seu ambiente, ou seja, criar os arquivos e diretórios necessários e inicializar o banco de dados.

### Configurando o Usuário Airflow Correto

No **Linux**, o início rápido precisa saber o ID do usuário do host e precisa ter o ID do grupo definido como `0`. Caso contrário, os arquivos criados em `dags`, `logs`, `config` e `plugins` serão criados com propriedade do usuário `root`. Você deve configurá-los para o docker-compose:

```bash
mkdir -p ./dags ./logs ./plugins ./config
echo -e "AIRFLOW_UID=$(id -u)" > .env
```

Para outros sistemas operacionais, você pode receber um aviso de que `AIRFLOW_UID` não está definido, mas pode ignorá-lo com segurança. Você também pode criar manualmente um arquivo `.env` na mesma pasta que `docker-compose.yaml` com este conteúdo para se livrar do aviso:

```
AIRFLOW_UID=50000
```

### Inicializar airflow.cfg (Opcional)

Se você deseja inicializar `airflow.cfg` com valores padrão antes de iniciar o serviço do Airflow, execute:

```bash
docker compose run airflow-cli airflow config list
```

Isso preencherá `airflow.cfg` com valores padrão na pasta `config`.

Em sistemas com SELinux/AppArmor, você pode encontrar problemas de permissão. Se isso acontecer, edite seu arquivo `docker-compose.yaml` adicionando o sufixo `:z` a todos os volumes:

```yaml
volumes:
  - ${AIRFLOW_PROJ_DIR:-.}/dags:/opt/airflow/dags:z
  - ${AIRFLOW_PROJ_DIR:-.}/logs:/opt/airflow/logs:z
  - ${AIRFLOW_PROJ_DIR:-.}/config:/opt/airflow/config:z
  - ${AIRFLOW_PROJ_DIR:-.}/plugins:/opt/airflow/plugins:z
```

Se, após esta mudança, você ainda estiver enfrentando problemas de permissão ao criar o arquivo `airflow.cfg`, pode aplicar uma configuração muito permissiva à pasta `config/`:

```bash
sudo chmod -R 777 ./config
```

Observe que o comando acima é uma _solução alternativa_ que nunca deve ser usada em produção.

### Inicializar o Banco de Dados

Em **todos os sistemas operacionais**, você precisa executar migrações de banco de dados e criar a primeira conta de usuário. Para fazer isso, execute:

```bash
docker compose up airflow-init
```

Após a conclusão da inicialização, você deve ver uma saída relacionada a arquivos, pastas e plug-ins e, finalmente, uma mensagem como esta:

```
airflow-init-1 exited with code 0
```

A conta criada tem o login `airflow` e a senha `airflow`.

## Limpando o Ambiente

O ambiente docker-compose que preparamos é um de "início rápido". Ele não foi projetado para ser usado em produção e tem várias ressalvas - uma delas sendo que a melhor maneira de se recuperar de qualquer problema é limpá-lo e recomeçar do zero.

A melhor maneira de fazer isso é:

- Execute o comando `docker compose down --volumes --remove-orphans` no diretório onde você baixou o arquivo `docker-compose.yaml`
- Remova o diretório inteiro onde você baixou o arquivo `docker-compose.yaml`: `rm -rf '<DIRETÓRIO>'`
- Execute este guia desde o início, começando baixando novamente o arquivo `docker-compose.yaml`

## Executando o Airflow

Agora você pode iniciar todos os serviços:

```bash
docker compose up
```

**Nota**: docker-compose é sintaxe antiga. Por favor, verifique o Stackoverflow para mais detalhes.

Em um segundo terminal, você pode verificar a condição dos containers e certificar-se de que nenhum container está em uma condição não saudável:

```bash
$ docker ps
CONTAINER ID   IMAGE                  COMMAND                  CREATED          STATUS                    PORTS                              NAMES
247ebe6cf87a   apache/airflow:3.1.3   "/usr/bin/dumb-init …"   3 minutes ago    Up 3 minutes (healthy)    8080/tcp                           compose_airflow-worker_1
ed9b09fc84b1   apache/airflow:3.1.3   "/usr/bin/dumb-init …"   3 minutes ago    Up 3 minutes (healthy)    8080/tcp                           compose_airflow-scheduler_1
7cb1fb603a98   apache/airflow:3.1.3   "/usr/bin/dumb-init …"   3 minutes ago    Up 3 minutes (healthy)    0.0.0.0:8080->8080/tcp             compose_airflow-api_server_1
74f3bbe506eb   postgres:16            "docker-entrypoint.s…"   18 minutes ago   Up 17 minutes (healthy)   5432/tcp                           compose_postgres_1
0bd6576d23cb   redis:latest           "docker-entrypoint.s…"   10 hours ago     Up 17 minutes (healthy)   0.0.0.0:6379->6379/tcp             compose_redis_1
```

## Acessando o Ambiente

Após iniciar o Airflow, você pode interagir com ele de 3 maneiras:

- executando comandos CLI
- através de um navegador usando a interface web
- usando a API REST

### Executando Comandos CLI

Você também pode executar comandos CLI, mas deve fazê-lo em um dos serviços `airflow-*` definidos. Por exemplo, para executar `airflow info`, execute o seguinte comando:

```bash
docker compose run airflow-worker airflow info
```

Se você tiver Linux ou Mac OS, pode facilitar seu trabalho e baixar scripts wrapper opcionais que permitirão executar comandos com um comando mais simples:

```bash
curl -LfO 'https://airflow.apache.org/docs/apache-airflow/3.1.3/airflow.sh'
chmod +x airflow.sh
```

Agora você pode executar comandos mais facilmente:

```bash
./airflow.sh info
```

Você também pode usar `bash` como parâmetro para entrar no shell bash interativo no container ou `python` para entrar no container python:

```bash
./airflow.sh bash
```

```bash
./airflow.sh python
```

### Acessando a Interface Web

Uma vez que o cluster tenha iniciado, você pode fazer login na interface web e começar a experimentar com DAGs.

O webserver está disponível em: `http://localhost:8080`. A conta padrão tem o login `airflow` e a senha `airflow`.

### Enviando Requisições para a API REST

A autenticação básica de nome de usuário e senha é atualmente suportada para a API REST, o que significa que você pode usar ferramentas comuns para enviar requisições à API.

O webserver está disponível em: `http://localhost:8080`. A conta padrão tem o login `airflow` e a senha `airflow`.

Aqui está um exemplo de comando `curl`, que envia uma requisição para recuperar uma lista de pools:

```bash
ENDPOINT_URL="http://localhost:8080"
curl -X GET  \
    --user "airflow:airflow" \
    "${ENDPOINT_URL}/api/v1/pools"
```

## Limpeza

Para parar e excluir containers, excluir volumes com dados do banco de dados e baixar imagens, execute:

```bash
docker compose down --volumes --rmi all
```

## Usando Imagens Personalizadas

Quando você deseja executar o Airflow localmente, pode querer usar uma imagem estendida, contendo algumas dependências adicionais - por exemplo, você pode adicionar novos pacotes Python ou atualizar provedores do Airflow para uma versão posterior. Isso pode ser feito facilmente especificando `build: .` em seu `docker-compose.yaml` e colocando um Dockerfile personalizado ao lado do seu `docker-compose.yaml`. Então você pode usar o comando `docker compose build` para construir sua imagem (você só precisa fazer isso uma vez). Você também pode adicionar a flag `--build` aos seus comandos `docker compose` para reconstruir as imagens dinamicamente quando executar outros comandos `docker compose`.

**Nota**: Criar imagens personalizadas significa que você também precisa manter um nível de automação, pois precisa recriar as imagens quando os pacotes que deseja instalar ou o Airflow forem atualizados. Não se esqueça de manter esses scripts.

## Caso Especial - Adicionando Dependências via Arquivo requirements.txt

O caso usual para imagens personalizadas é quando você deseja adicionar um conjunto de requisitos a ela - geralmente armazenados no arquivo `requirements.txt`. Para desenvolvimento, você pode ser tentado a adicioná-lo dinamicamente quando estiver iniciando a imagem original do Airflow, mas isso tem vários efeitos colaterais (por exemplo, seus containers iniciarão muito mais lentamente - cada dependência adicional atrasará ainda mais o tempo de inicialização de seus containers). Especificamente quando você deseja adicionar seu próprio arquivo de requisitos, deve seguir estas etapas:

1. Comente a linha `image: ...` e remova o comentário da linha `build: .` no arquivo `docker-compose.yaml`. A parte relevante do arquivo docker-compose deve se parecer com (use a tag de imagem correta):

```yaml
#image: ${AIRFLOW_IMAGE_NAME:-apache/airflow:3.1.3}
build: .
```

2. Crie o `Dockerfile` na mesma pasta em que seu arquivo `docker-compose.yaml` está com conteúdo semelhante a:

```dockerfile
FROM apache/airflow:3.1.3
ADD requirements.txt .
RUN pip install apache-airflow==${AIRFLOW_VERSION} -r requirements.txt
```

É a melhor prática instalar o apache-airflow na mesma versão que vem da imagem original. Dessa forma, você pode ter certeza de que `pip` não tentará fazer downgrade ou upgrade do apache airflow ao instalar outros requisitos.

3. Coloque o arquivo `requirements.txt` no mesmo diretório.

Execute `docker compose build` para construir a imagem, ou adicione a flag `--build` aos comandos `docker compose up` ou `docker compose run` para construir a imagem automaticamente conforme necessário.

## Caso Especial - Adicionando um Arquivo de Configuração Personalizado

Se você tiver um arquivo de configuração personalizado e desejar usá-lo em sua instância do Airflow, precisa executar as seguintes etapas:

1. Substitua o arquivo `airflow.cfg` gerado automaticamente na pasta config local pelo seu arquivo de configuração personalizado.
2. Se seu arquivo de configuração tiver um nome diferente de `airflow.cfg`, ajuste o nome do arquivo em `AIRFLOW_CONFIG: '/opt/airflow/config/airflow.cfg'`

## Rede (Networking)

Em geral, se você deseja usar o Airflow localmente, seus DAGs podem tentar se conectar a servidores que estão sendo executados no host. Para conseguir isso, uma configuração extra deve ser adicionada ao `docker-compose.yaml`. Por exemplo, no Linux, a configuração deve estar na seção `services: airflow-worker` adicionando `extra_hosts: - "host.docker.internal:host-gateway"`; e use `host.docker.internal` em vez de `localhost`. Esta configuração varia em diferentes plataformas. Consulte a documentação do Docker para Windows e Mac para mais informações.

## Depurando o Airflow Dentro do Container Docker Usando PyCharm

**Pré-requisitos**: Crie um projeto no PyCharm e baixe o docker-compose.yaml.

**Etapas**:

1. Modifique `docker-compose.yaml`. Adicione a seguinte seção sob a seção `services`:

```yaml
airflow-python:
  <<: *airflow-common
  profiles:
      - debug
  environment:
      <<: *airflow-common-env
  user: "50000:0"
  entrypoint: [ "/bin/bash", "-c" ]
```

**Nota**: Este trecho de código cria um novo serviço chamado "airflow-python" especificamente para o interpretador Python do PyCharm. Em um sistema Linux, se você executou o comando `echo -e "AIRFLOW_UID=$(id -u)" > .env`, você precisa definir `user: "50000:0"` no serviço `airflow-python` para evitar o erro `Unresolved reference 'airflow'` do PyCharm.

2. Configure o Interpretador do PyCharm
    - Abra o PyCharm e navegue até **Settings** > **Project: <Nome do Seu Projeto>** > **Python Interpreter**.
    - Clique no botão **"Add Interpreter"** e escolha **"On Docker Compose"**.
    - No campo **Configuration file**, selecione seu arquivo `docker-compose.yaml`.
    - No campo **Service**, escolha o serviço `airflow-python` recém-adicionado.
    - Clique em **"Next"** e siga as instruções para concluir a configuração.

A construção do índice do interpretador pode levar algum tempo.

Uma vez configurado, você pode depurar seu código Airflow dentro do ambiente de container, imitando sua configuração local.

## FAQ: Perguntas Frequentes

### `ModuleNotFoundError: No module named 'XYZ'`

O arquivo Docker Compose usa a imagem mais recente do Airflow (apache/airflow). Se você precisar instalar uma nova biblioteca Python ou biblioteca do sistema, pode personalizar e estendê-la.

## O Que Vem Depois?

A partir deste ponto, você pode ir para a seção Tutoriais para mais exemplos ou a seção Guias de Como Fazer se estiver pronto para colocar a mão na massa.

## Variáveis de Ambiente Suportadas pelo Docker Compose

Não confunda os nomes das variáveis aqui com os argumentos de build definidos quando a imagem é construída. O argumento de build `AIRFLOW_UID` padrão é `50000` quando a imagem é construída. Por outro lado, as variáveis de ambiente abaixo podem ser definidas quando o container está em execução.

|Variável|Descrição|Padrão|
|---|---|---|
|`AIRFLOW_IMAGE_NAME`|Imagem do Airflow a ser usada.|apache/airflow:3.1.3|
|`AIRFLOW_UID`|UID do usuário para executar os containers Airflow. Sobrescreva se você quiser usar um UID não padrão do Airflow (por exemplo, quando você mapeia pastas do host, deve ser definido como resultado da chamada `id -u`).|`50000`|

**Nota**: Antes do Airflow 2.2, o Docker Compose também tinha o parâmetro `AIRFLOW_GID`, mas ele não forneceu nenhuma funcionalidade adicional - apenas adicionou confusão - então foi removido.

Essas variáveis adicionais são úteis caso você esteja experimentando/testando a instalação do Airflow via Docker Compose. Elas não se destinam a ser usadas em produção:

|Variável|Descrição|Padrão|
|---|---|---|
|`_AIRFLOW_WWW_USER_USERNAME`|Nome de usuário para a conta de interface do administrador. Se este valor for especificado, o usuário da interface administrativa é criado automaticamente.|airflow|
|`_AIRFLOW_WWW_USER_PASSWORD`|Senha para a conta de interface do administrador. Usado apenas quando `_AIRFLOW_WWW_USER_USERNAME` é definido.|airflow|
|`_PIP_ADDITIONAL_REQUIREMENTS`|Se não estiver vazio, os containers do Airflow tentarão instalar os requisitos especificados na variável. Exemplo: `lxml==4.6.3 charset-normalizer==1.4.1`. Disponível na imagem Airflow 2.1.1 e superior.||

---

Esta é uma tradução completa e detalhada da página sobre execução do Airflow no Docker, mantendo toda a estrutura técnica e instruções do original.