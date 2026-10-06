 
O `data_platform_lite` e uma plataforma local de dados para desenvolvimento e testes usando Apache Spark Standalone e MinIO como storage S3 compativel. O projeto tambem inclui um modelo de execucao seguro para Agents de IA, baseado em DevPod, Firejail e um gateway HTTP que permite apenas comandos Spark previamente aprovados.

## Objetivo Do Projeto

O projeto resolve tres necessidades principais:

- Disponibilizar uma plataforma local para executar e validar jobs Spark.

- Fornecer um storage S3 compativel local usando MinIO, incluindo buckets para dados e logs Spark.

- Permitir que um Agent IA opere a infraestrutura de forma controlada, sem acesso direto ao Docker socket, Docker Compose, SSH, arquivos internos sensiveis ou bancos privados.

## Visao Geral Da Arquitetura


A plataforma e composta por:

  

- Um workspace DevPod com Docker local.

- Um sandbox Firejail para o Agent IA.

- Um sandbox Firejail para operacoes de infraestrutura.

- Um gateway FastAPI chamado `infra-gateway`.

- Um cluster Spark Standalone com master, workers e history server.

- Um MinIO para storage local S3 compativel.

- Scripts operacionais para subir, parar, escalar e consultar a plataforma.

  

Fluxo resumido:

  

```text

DevPod workspace

├── Firejail agent sandbox

│   └── Agent IA

│       └── chama apenas scripts gateway-*

└── Firejail infra sandbox

    └── Docker Compose

        ├── infra-gateway

        ├── spark-master

        ├── spark-worker

        ├── spark-history

        ├── minio

        └── minio-init

```

  

## Componentes Principais

  

### DevContainer

  

Arquivos:

  

- `.devcontainer/devcontainer.json`

- `.devcontainer/Dockerfile`

  

O DevContainer define o ambiente de desenvolvimento usado pelo DevPod. Ele cria um usuario `devpod`, instala ferramentas necessarias e monta o projeto em `/workspace/data_platform_lite`.

  

Ferramentas instaladas:

  

- `curl`

- `firejail`

- `jq`

- `openssh-client`

- `python3`

- `python3-pip`

- `python3-venv`

- `sudo`

- `uidmap`

- `docker-ce-cli`

- `docker-compose-plugin`

  

Montagens relevantes:

  

- Repositorio local em `/workspace/data_platform_lite`.

- Docker socket do host em `/var/run/docker.sock`.

  

Variavel de ambiente relevante:

  

- `AGENT_GATEWAY_URL=http://host.docker.internal:18081`

  

O Docker socket e montado no workspace para permitir operacoes de infraestrutura, mas o Agent nao deve recebe-lo diretamente dentro do sandbox.

  

### Firejail Agent Sandbox

  

Arquivo:

  

- `firejail/agent.profile`

  

Esse profile restringe o ambiente do Agent IA. Ele remove capacidades, impede privilegios novos, usa `/tmp` e `/dev` privados e bloqueia acesso a diretorios e sockets sensiveis.

  

Principais bloqueios:

  

- `${HOME}/.ssh`

- `${HOME}/.gnupg`

- `/run/docker.sock`

- `/var/run/docker.sock`

- `.devcontainer`

- `.git`

- `firejail`

- `infra`

  

Principais permissoes:

  

- `agent/`

- `scripts/gateway-spark-submit.sh`

- `scripts/gateway-spark-status.sh`

- `scripts/gateway-spark-logs.sh`

- `scripts/gateway-spark-kill.sh`

  

O diretorio `scripts` fica somente leitura para o Agent.

  

### Firejail Infra Sandbox

  

Arquivo:

  

- `firejail/infra.profile`

  

Esse profile e usado para iniciar e parar a infraestrutura. Ele permite acesso ao projeto e ao Docker socket, mas bloqueia arquivos sensiveis do usuario e o diretorio do Agent.

  

Principais permissoes:

  

- `/workspace/data_platform_lite`

- `/run/docker.sock`

- `/var/run/docker.sock`

  

Principais bloqueios:

  

- `${HOME}/.ssh`

- `${HOME}/.gnupg`

- `/workspace/data_platform_lite/agent`

  

### Firejail Gateway Profile

  

Arquivo:

  

- `firejail/gateway.profile`

  

Esse profile foi criado para restringir o processo do gateway de comandos. Ele permite leitura do codigo do gateway e acesso ao Docker socket, mas bloqueia `.git`, `agent`, SSH e GPG.

  

No compose atual, o `infra-gateway` roda em container Docker. O profile existe como base de isolamento caso o gateway seja executado como processo local via Firejail.

  

### Infra Gateway

  

Arquivos:

  

- `infra/gateway/app.py`

- `infra/gateway/allowed_commands.py`

- `infra/gateway/Dockerfile`

- `infra/gateway/requirements.txt`

  

O `infra-gateway` e uma API FastAPI que recebe requisicoes HTTP e executa apenas comandos Spark permitidos dentro do container `spark-master` usando `docker exec`.

  

Dependencias Python:

  

- `fastapi==0.115.6`

- `uvicorn[standard]==0.34.0`

- `pydantic==2.10.4`

  

Imagem do gateway:

  

- Base `python:3.12-slim`.

- Instala `docker.io` e `curl`.

- Copia `app.py` e `allowed_commands.py`.

- Expoe porta `8080`.

- Inicia `uvicorn app:app --host 0.0.0.0 --port 8080`.

  

### Spark Master

  

Arquivo:

  

- `conf/docker/docker-compose.yml`

  

Servico:

  

- `spark-master`

  

Responsabilidades:

  

- Coordenar o cluster Spark Standalone.

- Receber comandos `spark-submit`.

- Expor UI do Spark Master.

- Ser o container alvo do gateway para submit, status, logs e kill.

  

Portas:

  

- `7077:7077` para protocolo Spark Standalone.

- `8083:8083` para UI do Spark Master.

  

Comando executado:

  

```bash

/opt/spark/bin/spark-class org.apache.spark.deploy.master.Master

```

  

### Spark Workers

  

Servico:

  

- `spark-worker`

  

Responsabilidades:

  

- Executar tasks das aplicacoes Spark.

- Conectar ao master em `spark://spark-master:7077`.

  

Portas:

  

- `8084-8096:8084` para UIs dos workers.

  

Replicas padrao:

  

- `2` no `docker-compose.yml`.

  

O numero de workers pode ser alterado pelos scripts de escala.

  

### Spark History Server

  

Servico:

  

- `spark-history`

  

Responsabilidades:

  

- Ler event logs Spark gravados no MinIO.

- Expor interface historica das aplicacoes Spark.

  

Porta:

  

- `18080:18080`

  

Comando executado:

  

```bash

/opt/spark/bin/spark-class org.apache.spark.deploy.history.HistoryServer

```

  

### MinIO

  

Servico:

  

- `minio`

  

Responsabilidades:

  

- Fornecer storage S3 compativel local.

- Armazenar dados do warehouse Spark.

- Armazenar event logs do Spark.

  

Portas:

  

- `9000:9000` para API S3.

- `9001:9001` para console web.

  

Credenciais locais padrao:

  

- Usuario: `minioadmin`

- Senha: `minioadmin`

  

Volume:

  

- `conf/minio/data:/data`

  

### MinIO Init

  

Servico:

  

- `minio-init`

  

Responsabilidades:

  

- Aguardar o MinIO ficar disponivel.

- Criar buckets iniciais.

  

Buckets criados:

  

- `spark-logs`

- `data-bureau`

  

## Redes Docker

  

### lite-platform-network

  

Definida em:

  

- `conf/docker/docker-compose.yml`

  

E a rede principal da plataforma. Conecta:

  

- `spark-master`

- `spark-worker`

- `spark-history`

- `minio`

- `minio-init`

  

### agent_control

  

Definida no exemplo de override:

  

- `conf/docker/docker-compose.override.yml.example`

  

Essa rede e usada para controle pelo Agent e pelo gateway. O override conecta:

  

- `infra-gateway`

- `spark-master`

  

Regra de seguranca importante:

  

- Servicos sensiveis, como bancos privados, nao devem ser conectados a `agent_control`.

  

## Docker Compose

  

Arquivo principal:

  

- `conf/docker/docker-compose.yml`

  

Arquivo de override recomendado:

  

- `conf/docker/docker-compose.override.yml.example`

  

O compose principal define Spark, MinIO e rede principal. O override adiciona o `infra-gateway`, publica sua porta no loopback e cria a rede `agent_control`.

  

Para usar o override, copie:

  

```bash

cp conf/docker/docker-compose.override.yml.example conf/docker/docker-compose.override.yml

```

  

O arquivo `conf/docker/docker-compose.override.yml` e ignorado pelo Git para permitir ajustes locais.

  

## Imagem Spark Customizada

  

Arquivo:

  

- `conf/docker/Dockerfile`

  

A imagem Spark customizada usa:

  

- Base `spark:4.0.1`.

- JARs resolvidos pelo Maven em `conf/maven/target/jars`.

- Copia JARs para `/opt/spark/external-jars/` e `/opt/spark/jars/`.

- Define `SPARK_EXTRA_CLASSPATH=/opt/spark/external-jars/*`.

  

Essa imagem adiciona suporte para integrações usadas pela plataforma, especialmente Hudi e S3A.

  

## Dependencias Maven Do Runtime Spark

  

Arquivos:

  

- `conf/maven/pom.xml`

- `conf/maven/maven_setup.sh`

  

O `pom.xml` resolve dependencias necessarias para o runtime Spark:

  

- `org.apache.hudi:hudi-spark4.0-bundle_2.13:1.1.1`

- `org.apache.hadoop:hadoop-aws:3.4.0`

- `com.amazonaws:aws-java-sdk-bundle:1.12.700`

  

O script `maven_setup.sh` executa:

  

```bash

mvn -f conf/maven/pom.xml clean package

```

  

Ele pula a execucao se o diretorio `conf/maven/target` ja existir.

  

## Configuracao Spark

  

Arquivo:

  

- `conf/spark/spark-defaults.conf`

  

Configuracoes principais:

  

- Endpoint S3A: `http://minio:9000`

- Access key: `minioadmin`

- Secret key: `minioadmin`

- Path style access: `true`

- SSL desabilitado para conexao local.

- Warehouse Spark em `s3a://data-bureau/`.

- Event logs habilitados.

- Event logs em `s3a://spark-logs/`.

- History Server lendo logs em `s3a://spark-logs/`.

- Serializer Kryo.

- Extensoes e catalogo Hudi habilitados.

  

## Scripts Operacionais

  

### start_platform.sh

  

Arquivo:

  

- `start_platform.sh`

  

Responsabilidades:

  

- Preparar diretorios do MinIO.

- Resolver dependencias Maven.

- Delegar subida e escala para `conf/docker/scale_worker.sh`.

- Exibir URLs das UIs ao final.

  

Fluxo interno:

  

```text

start_platform.sh

├── conf/minio/minio_setup.sh

├── conf/maven/maven_setup.sh

└── conf/docker/scale_worker.sh

```

  

### start-infra.sh

  

Arquivo:

  

- `scripts/start-infra.sh`

  

Responsabilidades:

  

- Verificar se `conf/docker/docker-compose.yml` existe.

- Verificar se `conf/docker/docker-compose.override.yml` existe.

- Executar `start_platform.sh` dentro do profile Firejail de infraestrutura.

  

Uso:

  

```bash

./scripts/start-infra.sh

```

  

Com escala de workers:

  

```bash

./scripts/start-infra.sh --workers 4

```

  

ou:

  

```bash

./scripts/start-infra.sh -w 4

```

  

### stop-infra.sh

  

Arquivo:

  

- `scripts/stop-infra.sh`

  

Responsabilidades:

  

- Verificar se compose principal e override existem.

- Executar `conf/docker/scale_worker.sh --down` dentro do profile Firejail de infraestrutura.

  

Uso:

  

```bash

./scripts/stop-infra.sh

```

  

### scale_worker.sh

  

Arquivo:

  

- `conf/docker/scale_worker.sh`

  

Responsabilidades:

  

- Subir a plataforma com `docker compose up -d`.

- Escalar o numero de `spark-worker`.

- Parar a plataforma com `--down`.

  

Valores:

  

- Minimo: `1`

- Maximo: `12`

- Padrao: `2`

  

Comando principal:

  

```bash

docker compose up -d --scale spark-worker="${SPARK_WORKERS}"

```

  

### start-agent.sh

  

Arquivo:

  

- `scripts/start-agent.sh`

  

Responsabilidades:

  

- Criar `agent/workspace` se necessario.

- Entrar nesse diretorio.

- Iniciar shell dentro do `firejail/agent.profile`.

  

Uso:

  

```bash

./scripts/start-agent.sh

```

  

## Scripts Gateway Para O Agent

  

Os scripts `gateway-*` sao a interface permitida para o Agent IA operar Spark.

  

### gateway-spark-submit.sh

  

Arquivo:

  

- `scripts/gateway-spark-submit.sh`

  

Uso:

  

```bash

./scripts/gateway-spark-submit.sh /opt/spark/jobs/job.py --master spark://spark-master:7077

```

  

Responsabilidades:

  

- Receber o path da aplicacao.

- Receber argumentos opcionais de `spark-submit`.

- Montar JSON com `jq`.

- Fazer `POST /commands/spark-submit` no gateway.

  

Payload enviado:

  

```json

{

  "application": "/opt/spark/jobs/job.py",

  "args": ["--master", "spark://spark-master:7077"]

}

```

  

### gateway-spark-status.sh

  

Arquivo:

  

- `scripts/gateway-spark-status.sh`

  

Uso:

  

```bash

./scripts/gateway-spark-status.sh

```

  

Responsabilidade:

  

- Fazer `GET /commands/spark-status`.

  

### gateway-spark-logs.sh

  

Arquivo:

  

- `scripts/gateway-spark-logs.sh`

  

Uso:

  

```bash

./scripts/gateway-spark-logs.sh 100

```

  

Responsabilidade:

  

- Fazer `GET /commands/spark-logs?lines=100`.

  

### gateway-spark-kill.sh

  

Arquivo:

  

- `scripts/gateway-spark-kill.sh`

  

Uso:

  

```bash

./scripts/gateway-spark-kill.sh app-20260101000000-0001

```

  

Responsabilidades:

  

- Receber um Spark application id.

- Montar JSON com `jq`.

- Fazer `POST /commands/spark-kill`.

  

Payload enviado:

  

```json

{

  "app_id": "app-20260101000000-0001"

}

```

  

## API Do Infra Gateway

  

### GET /health

  

Retorna status basico do gateway.

  

Resposta:

  

```json

{

  "status": "ok"

}

```

  

### POST /commands/spark-submit

  

Executa `spark-submit` no container `spark-master`.

  

Entrada:

  

```json

{

  "application": "/opt/spark/jobs/job.py",

  "args": ["--master", "spark://spark-master:7077"]

}

```

  

Validacoes:

  

- `application` deve ter entre 1 e 512 caracteres.

- `application` deve iniciar com um prefixo permitido.

- `args` pode ter no maximo 64 itens.

- Cada argumento pode ter no maximo 512 caracteres.

- Argumentos nao podem conter byte nulo, `\n` ou `\r`.

- Flags Spark precisam estar na allowlist.

  

### GET /commands/spark-status

  

Executa dentro do `spark-master`:

  

```bash

curl -fsS http://localhost:8083/json/

```

  

Serve para consultar o estado do Spark Master.

  

### GET /commands/spark-logs

  

Parametro:

  

- `lines`: inteiro entre `1` e `500`, padrao `100`.

  

Executa dentro do `spark-master` um comando para buscar o log Spark mais recente e retornar as ultimas linhas.

  

### POST /commands/spark-kill

  

Entrada:

  

```json

{

  "app_id": "app-20260101000000-0001"

}

```

  

Validacao:

  

- `app_id` precisa bater com o padrao `^app-[0-9-]+-[0-9]+$`.

  

Executa dentro do `spark-master`:

  

```bash

spark-class org.apache.spark.deploy.Client kill <app_id>

```

  

## Allowlist Do Gateway

  

Arquivo:

  

- `infra/gateway/allowed_commands.py`

  

### Comandos Permitidos

  

- `spark-submit`

- `spark-status`

- `spark-logs`

- `spark-kill`

  

Todos apontam para o servico alvo `spark-master`.

  

Timeouts:

  

- `spark-submit`: `900` segundos.

- `spark-status`: `30` segundos.

- `spark-logs`: `60` segundos.

- `spark-kill`: `60` segundos.

  

### Argumentos Permitidos Para spark-submit

  

- `--class`

- `--conf`

- `--deploy-mode`

- `--driver-memory`

- `--executor-cores`

- `--executor-memory`

- `--jars`

- `--master`

- `--name`

- `--packages`

- `--py-files`

  

### Prefixos Permitidos Para Aplicacoes

  

- `/workspace/jobs/`

- `/opt/spark/jobs/`

- `local:///opt/spark/jobs/`

  

## Portas Expostas

  

| Servico | Porta | Descricao |

| --- | ---: | --- |

| Spark Master | `7077` | Endpoint Spark Standalone |

| Spark Master UI | `8083` | UI do master |

| Spark Worker UI | `8084-8096` | UIs dos workers |

| Spark History Server | `18080` | Historico de aplicacoes Spark |

| MinIO API | `9000` | API S3 compativel |

| MinIO Console | `9001` | Console web do MinIO |

| Infra Gateway | `18081` | Gateway HTTP publicado no loopback pelo override recomendado |

  

## Fluxo De Uso Recomendado

  

### 1. Subir Workspace DevPod

  

```bash

devpod up .

```

  

### 2. Criar Override Local Do Docker Compose

  

```bash

cp conf/docker/docker-compose.override.yml.example conf/docker/docker-compose.override.yml

```

  

Revise o arquivo antes de subir a infraestrutura.

  

### 3. Subir Infraestrutura

  

```bash

./scripts/start-infra.sh

```

  

Com quatro workers:

  

```bash

./scripts/start-infra.sh --workers 4

```

  

### 4. Iniciar Sandbox Do Agent

  

```bash

./scripts/start-agent.sh

```

  

### 5. Operar Spark Via Gateway

  

Dentro do sandbox do Agent:

  

```bash

./scripts/gateway-spark-status.sh

./scripts/gateway-spark-logs.sh 100

./scripts/gateway-spark-submit.sh /opt/spark/jobs/job.py --master spark://spark-master:7077

./scripts/gateway-spark-kill.sh app-20260101000000-0001

```

  

### 6. Parar Infraestrutura

  

```bash

./scripts/stop-infra.sh

```

  

## Modelo De Seguranca

  

O modelo de seguranca separa operacao do Agent e controle da infraestrutura.

  

O Agent pode:

  

- Usar seu workspace em `agent/workspace`.

- Executar apenas scripts `gateway-*` permitidos.

- Chamar o `infra-gateway` por HTTP.

  

O Agent nao deve poder:

  

- Acessar Docker socket.

- Executar Docker ou Docker Compose diretamente.

- Acessar arquivos de infraestrutura.

- Acessar `.git`.

- Acessar SSH/GPG do usuario.

- Acessar bancos ou servicos internos sensiveis.

  

O gateway pode:

  

- Receber chamadas HTTP.

- Validar payloads.

- Executar somente comandos definidos em `ALLOWED_COMMANDS`.

- Executar `docker exec` no container alvo.

  

O gateway nao deve:

  

- Expor comandos arbitrarios.

- Aceitar paths fora de `ALLOWED_PATH_PREFIXES`.

- Aceitar flags Spark fora de `ALLOWED_SPARK_SUBMIT_ARGS`.

- Ser exposto publicamente sem protecao adicional.

  

Cuidados ao adicionar servicos:

  

- Nao conectar bancos sigilosos a `agent_control`.

- Nao publicar portas de bancos sigilosos no host.

- Nao usar `network_mode: host` sem revisao.

- Nao montar `/var/run/docker.sock` em containers acessiveis pelo Agent.

- Manter o gateway publicado em loopback quando possivel.

  

## Mapa De Arquivos

  

```text

data_platform_lite/

├── README.md

├── LICENSE

├── start_platform.sh

├── .gitignore

├── .devcontainer/

│   ├── Dockerfile

│   └── devcontainer.json

├── .opencode/

│   └── skills/

│       └── c4-model/

│           └── SKILL.md

├── agent/

│   └── workspace/

│       └── .gitkeep

├── conf/

│   ├── docker/

│   │   ├── Dockerfile

│   │   ├── docker-compose.yml

│   │   ├── docker-compose.override.yml.example

│   │   └── scale_worker.sh

│   ├── maven/

│   │   ├── maven_setup.sh

│   │   └── pom.xml

│   ├── minio/

│   │   └── minio_setup.sh

│   └── spark/

│       └── spark-defaults.conf

├── docs/

│   ├── c4-model.md

│   └── project-documentation.md

├── firejail/

│   ├── agent.profile

│   ├── gateway.profile

│   └── infra.profile

├── infra/

│   └── gateway/

│       ├── Dockerfile

│       ├── allowed_commands.py

│       ├── app.py

│       └── requirements.txt

└── scripts/

    ├── gateway-spark-kill.sh

    ├── gateway-spark-logs.sh

    ├── gateway-spark-status.sh

    ├── gateway-spark-submit.sh

    ├── start-agent.sh

    ├── start-infra.sh

    └── stop-infra.sh

```

  

## Responsabilidade Por Arquivo

  

| Caminho | Responsabilidade |

| --- | --- |

| `README.md` | Introducao resumida e fluxo inicial do projeto |

| `start_platform.sh` | Entrada principal para preparar MinIO, Maven e subir a plataforma |

| `.devcontainer/Dockerfile` | Imagem do workspace DevPod |

| `.devcontainer/devcontainer.json` | Configuracao do workspace, mounts e variaveis |

| `.opencode/skills/c4-model/SKILL.md` | Skill local para criacao/revisao de diagramas C4 em Mermaid |

| `agent/workspace/.gitkeep` | Mantem o diretorio de trabalho do Agent versionado |

| `conf/docker/Dockerfile` | Imagem Spark customizada com JARs externos |

| `conf/docker/docker-compose.yml` | Compose principal da plataforma Spark + MinIO |

| `conf/docker/docker-compose.override.yml.example` | Exemplo de override com gateway e rede de controle |

| `conf/docker/scale_worker.sh` | Sobe, escala e derruba containers via Docker Compose |

| `conf/maven/pom.xml` | Dependencias Java/Scala usadas no runtime Spark |

| `conf/maven/maven_setup.sh` | Resolve e empacota dependencias Maven |

| `conf/minio/minio_setup.sh` | Cria diretorio local de dados do MinIO |

| `conf/spark/spark-defaults.conf` | Configuracao Spark, S3A, Hudi e event logs |

| `docs/c4-model.md` | Documentacao C4 da arquitetura |

| `docs/project-documentation.md` | Documentacao detalhada consolidada do projeto |

| `firejail/agent.profile` | Isolamento do Agent IA |

| `firejail/infra.profile` | Isolamento de operacoes de infraestrutura |

| `firejail/gateway.profile` | Isolamento proposto para o gateway quando executado localmente |

| `infra/gateway/app.py` | API FastAPI do gateway de comandos |

| `infra/gateway/allowed_commands.py` | Allowlist de comandos, argumentos e paths permitidos |

| `infra/gateway/Dockerfile` | Imagem Docker do gateway |

| `infra/gateway/requirements.txt` | Dependencias Python do gateway |

| `scripts/start-infra.sh` | Inicia a infraestrutura via Firejail |

| `scripts/stop-infra.sh` | Para a infraestrutura via Firejail |

| `scripts/start-agent.sh` | Inicia o sandbox do Agent |

| `scripts/gateway-spark-submit.sh` | Wrapper permitido para submit Spark |

| `scripts/gateway-spark-status.sh` | Wrapper permitido para status Spark |

| `scripts/gateway-spark-logs.sh` | Wrapper permitido para logs Spark |

| `scripts/gateway-spark-kill.sh` | Wrapper permitido para encerrar aplicacao Spark |

  

## Pontos De Atencao Encontrados

  

### Divergencia De Porta Do Gateway

  

O README, o DevContainer e o override recomendado indicam gateway em `18081`:

  

- README: `127.0.0.1:18081`

- `.devcontainer/devcontainer.json`: `http://host.docker.internal:18081`

- `docker-compose.override.yml.example`: `127.0.0.1:18081:8080`

  

Porem, os scripts `gateway-*` usam fallback para `http://127.0.0.1:18080` quando `AGENT_GATEWAY_URL` nao esta definida.

  

Isso e potencialmente problematico porque `18080` e a porta do Spark History Server. O recomendado e alinhar o fallback dos scripts para `18081` ou garantir que `AGENT_GATEWAY_URL` esteja sempre definida no ambiente do Agent.

  

Arquivos afetados:

  

- `scripts/gateway-spark-submit.sh`

- `scripts/gateway-spark-status.sh`

- `scripts/gateway-spark-logs.sh`

- `scripts/gateway-spark-kill.sh`

  

### Validacao De Workers Em scale_worker.sh

  

No parser de argumentos de `conf/docker/scale_worker.sh`, a validacao dentro de `-w|--workers` verifica `SPARK_WORKERS` antes de atribuir `$2`. Como `SPARK_WORKERS` ja tem valor padrao numerico, isso nao valida corretamente o argumento recebido.

  

Trecho relevante:

  

```bash

if [[ "$SPARK_WORKERS" =~ ^[0-9]+$ ]]; then

  SPARK_WORKERS="$2"

```

  

O esperado seria validar `$2` antes da atribuicao.

  

### Comentario Incorreto Em minio_setup.sh

  

O arquivo `conf/minio/minio_setup.sh` possui comentario dizendo `Skip Maven build`, mas o script trata setup do MinIO. O comportamento do script esta correto, mas o comentario pode confundir manutencao futura.

  

### Gateway Profile Nao Referenciado Pelo Compose

  

Existe `firejail/gateway.profile`, mas o gateway atual roda como container definido no override do Docker Compose. O profile parece ser uma alternativa para execucao local do gateway, mas isso nao esta operacionalizado por script no projeto atual.

  

## Licenca

  

O projeto inclui `LICENSE` com Apache License 2.0.

  

## Documentacao C4

  

A documentacao C4 detalhada esta em:

  

- `docs/c4-model.md`

  

Ela contem:

  

- Diagrama de contexto.

- Diagrama de containers.

- Diagrama de componentes do gateway.

- Fluxo principal de submit Spark via Agent.