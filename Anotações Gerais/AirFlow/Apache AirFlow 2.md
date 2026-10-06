# Visão Geral da Arquitetura do Airflow

O Airflow é uma plataforma que permite construir e executar **workflows** (fluxos de trabalho). Um workflow é representado como um **Dag** (Grafo Acíclico Dirigido) e contém peças individuais de trabalho chamadas **Tasks** (tarefas), organizadas levando em conta dependências e fluxos de dados.
![[Pasted image 20251222181746.png]]
Um Dag especifica as dependências entre tarefas, o que define a ordem na qual executar as tarefas. As tarefas descrevem o que fazer, seja buscar dados, executar análises, acionar outros sistemas ou mais.

O Airflow em si é agnóstico ao que você está executando - ele irá orquestrar e executar qualquer coisa, seja com suporte de alto nível de um dos nossos provedores, ou diretamente como um comando usando shell ou operadores Python.

## Componentes do Airflow

A arquitetura do Airflow consiste em múltiplos componentes:

### Componentes obrigatórios

Uma instalação mínima do Airflow consiste nos seguintes componentes:

| Componente                                          | O que é                                                                                                                                                                                                                                                                              | Responsabilidades principais                                                                                                                                                                                |
| --------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Scheduler (Agendador)**                           | Um **scheduler** (agendador), que lida tanto com o acionamento de workflows agendados quanto com o envio de tarefas ao executor para execução. O executor é uma propriedade de configuração do scheduler, não um componente separado, e é executado dentro do processo do scheduler. | Define quais tarefas executam e em que ordem; codifica cronograma e metadados; permite visualização na UI.                                                                                                  |
| **Dag processor**                                   | Um **Dag processor** (processador de Dag), que analisa arquivos Dag e os serializa no banco de dados de metadados.                                                                                                                                                                   | O DAG Processor  é responsável por varrer o diretório de DAGs, interpretar e validar os arquivos Python, detectar alterações e transformar o código em metadados persistidos no metadatabase.               |
| **Pasta de Arquivos DAG's**                         | Uma pasta de **arquivos Dag**, que é lida pelo scheduler para descobrir quais tarefas executar e quando executá-las.                                                                                                                                                                 | A pasta de DAGs é responsável por armazenar os arquivos Python que definem os workflows como código. Esses arquivos descrevem a estrutura do DAG, suas tarefas, dependências, agendamentos e configurações. |
| **Web Server (Web UI)**                             | A interface de usuário operacional servida pelo webserver do Airflow.                                                                                                                                                                                                                | Observabilidade e controle: visualizar DAGs, acionar/limpar tarefas, inspecionar logs, pausar DAGs, gerenciar conexões/variáveis.                                                                           |
| **Metadata Database (Banco de Dados de Metadados)** | O sistema persistente de registro (tipicamente PostgreSQL/MySQL).                                                                                                                                                                                                                    | Armazena execuções de DAG, instâncias de tarefas e estados, configurações, metadados de logs e histórico operacional.                                                                                       |

### Componentes opcionais

Alguns componentes do Airflow são opcionais e podem habilitar melhor extensibilidade, escalabilidade e desempenho:

| Componente           | O que é                                                                                                                                                                                                                   | Responsabilidades principais                                                                                                                      |
| -------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Worker**           | Um **worker** opcional, responsável por executar as tarefas enviadas pelo scheduler. Pode estar embutido no scheduler em instalações simples, rodar como processo no CeleryExecutor ou como um POD no KubernetesExecutor. | Executar tasks; consumir tarefas do executor; reportar status de execução ao metadatabase.                                                        |
| **Trigger**          | Um **trigger** opcional, responsável por executar tarefas diferidas usando um loop de eventos asyncio. Necessário apenas quando sensores ou operadores diferidos são utilizados.                                          | Gerenciar tarefas diferidas; aguardar eventos externos de forma assíncrona; liberar tasks quando as condições são atendidas.                      |
| **Pasta de Plugins** | Uma pasta opcional de **plugins** que permite estender o Airflow com operadores, hooks, sensores, macros e extensões customizadas.                                                                                        | Carregar extensões customizadas; ampliar funcionalidades do Airflow; disponibilizar plugins para scheduler, DAG processor, triggerer e webserver. |
### Diferenças entre Serviços e componentes no AirFlow

#### Serviços

São os **processos executáveis** que precisam estar rodando para o Airflow funcionar. Eles são iniciados como daemons/processos separados:

- **Webserver** - processo que roda o servidor web (geralmente com Gunicorn)
- **Scheduler** - processo daemon que agenda e dispara as tasks
- **Worker** - processo que executa as tasks (em modo distribuído com Celery)
- **Flower** - processo opcional para monitorar workers do Celery
- **Triggerer** - processo para lidar com tasks assíncronas (deferable operators)

Você inicia esses serviços com comandos como `airflow webserver`, `airflow scheduler`, etc.

##### Componentes

São os **elementos arquiteturais** que compõem o ecossistema do Airflow, mas não necessariamente rodam como processos independentes:

- **Metadata Database** - banco de dados (PostgreSQL/MySQL) que armazena metadados
- **Executor** - lógica/estratégia de execução (LocalExecutor, CeleryExecutor, KubernetesExecutor)
- **DAGs** - definições de workflows escritas em Python
- **Operators** - blocos de construção das tasks (PythonOperator, BashOperator, etc.)
- **Hooks** - interfaces para conexão com sistemas externos
- **Sensors** - tipo especial de operator que aguarda condições
- **Message Broker** - Redis/RabbitMQ (quando usando CeleryExecutor)

Em resumo: **serviços** são processos que você executa, enquanto **componentes** são partes da arquitetura que podem ou não ser processos separados. Por exemplo, o Executor é um componente configurado dentro do Scheduler, não um serviço independente.

### Excutors

Um **Executor** é o componente do Airflow que define **como** e **onde** as tasks serão executadas. Ele é responsável por determinar o mecanismo de execução das tarefas: se serão executadas localmente, distribuídas em múltiplos workers, ou isoladas em containers. O Executor atua como uma ponte entre o Scheduler (que decide **quando** executar) e a execução real das tasks. A escolha do executor adequado impacta diretamente na escalabilidade, isolamento e performance dos seus workflows.
#### SequentialExecutor

**IMPORTANTE: Descontinuado no Airflow 3.0.0**

O SequentialExecutor foi descontinuado a partir do Airflow 3.0.0, pois se tornou redundante já que o LocalExecutor suporta SQLite com modo WAL e oferece melhor desempenho com execução paralela [Apache Airflow](https://airflow.apache.org/docs/apache-airflow/3.0.0/release_notes.html)[Apache Airflow](https://airflow.apache.org/docs/apache-airflow/3.0.3/release_notes.html).

O mais simples, executa uma task por vez de forma sequencial. Usa SQLite como banco de dados. É adequado apenas para testes e desenvolvimento local, não para produção, pois não permite paralelismo.

**Documentação oficial:** [https://airflow.apache.org/docs/apache-airflow/stable/core-concepts/executor/index.html](https://airflow.apache.org/docs/apache-airflow/stable/core-concepts/executor/index.html)

#### LocalExecutor

Executa múltiplas tasks em paralelo usando processos separados na mesma máquina. Requer um banco de dados mais robusto (PostgreSQL/MySQL). É ideal para ambientes de produção pequenos ou médios onde uma única máquina tem recursos suficientes.

**Documentação oficial:** [https://airflow.apache.org/docs/apache-airflow/stable/core-concepts/executor/local.html](https://airflow.apache.org/docs/apache-airflow/stable/core-concepts/executor/local.html)

#### CeleryExecutor

Permite execução distribuída usando o framework Celery. As tasks são enviadas para uma fila de mensagens (Redis ou RabbitMQ) e executadas por workers distribuídos em múltiplas máquinas. É excelente para ambientes de produção que precisam escalar horizontalmente e processar grandes volumes de tasks.

**Documentação oficial:** [https://airflow.apache.org/docs/apache-airflow-providers-celery/stable/celery_executor.html](https://airflow.apache.org/docs/apache-airflow-providers-celery/stable/celery_executor.html)

#### KubernetesExecutor

Cria um pod do Kubernetes para cada task executada. Oferece isolamento total entre tasks e escalabilidade automática usando a infraestrutura do Kubernetes. Ideal para ambientes cloud-native e quando você precisa de recursos específicos ou isolamento por task.

**Documentação oficial:** [https://airflow.apache.org/docs/apache-airflow-providers-cncf-kubernetes/stable/kubernetes_executor.html](https://airflow.apache.org/docs/apache-airflow-providers-cncf-kubernetes/stable/kubernetes_executor.html)

#### CeleryKubernetesExecutor

**IMPORTANTE: Não suportado no Airflow 3.0.0+**

O CeleryKubernetesExecutor não é mais suportado a partir do Airflow 3.0.0, sendo recomendado usar o recurso de Múltiplos Executores Concorrentemente, que fornece funcionalidade equivalente de maneira mais flexível [Apache Airflow](https://airflow.apache.org/docs/apache-airflow-providers-celery/stable/celery_kubernetes_executor.html).

Combina CeleryExecutor e KubernetesExecutor, permitindo escolher qual usar por DAG ou task. Oferece flexibilidade para otimizar custos e performance usando Celery para tasks rápidas e Kubernetes para tasks que precisam de mais recursos ou isolamento.

**Documentação oficial:** [https://airflow.apache.org/docs/apache-airflow-providers-celery/stable/celery_kubernetes_executor.html](https://airflow.apache.org/docs/apache-airflow-providers-celery/stable/celery_kubernetes_executor.html)

#### LocalKubernetesExecutor

**IMPORTANTE: Não suportado no Airflow 3.0.0+**

Tanto o LocalKubernetesExecutor quanto o CeleryKubernetesExecutor não são mais suportados a partir do Airflow 3.0.0, pois sua implementação não era nativa do core do Airflow e faziam uso inadequado do campo queue nas Task Instances [Apache Airflow](https://airflow.apache.org/docs/apache-airflow/stable/core-concepts/executor/index.html).

Semelhante ao anterior, mas combina LocalExecutor com KubernetesExecutor, permitindo executar tasks simples localmente e tasks mais pesadas em pods do Kubernetes.

**Documentação oficial:** [https://airflow.apache.org/docs/apache-airflow/stable/core-concepts/executor/index.html](https://airflow.apache.org/docs/apache-airflow/stable/core-concepts/executor/index.html)

**Fonte:**

- Apache AirFlow Architecture Overview — Airflow 3.1.5 Documentation ([https://airflow.apache.org/docs/apache-airflow/stable/core-concepts/overview.html](https://airflow.apache.org/docs/apache-airflow/stable/core-concepts/overview.html "https://airflow.apache.org/docs/apache-airflow/stable/core-concepts/overview.html"))

- Apache Airflow Explained: A Beginner-Friendly Guide | DigitalOcean ([https://www.digitalocean.com/community/tutorials/apache-airflow-explained-beginner-guide](https://www.digitalocean.com/community/tutorials/apache-airflow-explained-beginner-guide "https://www.digitalocean.com/community/tutorials/apache-airflow-explained-beginner-guide"))****