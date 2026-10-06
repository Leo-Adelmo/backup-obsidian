# Apache Airflow: Arquitetura e Componentes em Docker Compose

## O Que é o Apache Airflow

Apache Airflow é uma plataforma de código aberto para criar, agendar e monitorar fluxos de trabalho de forma programática. Imagine que você precisa executar várias tarefas relacionadas a dados todos os dias: extrair informações de um banco de dados, processar esses dados, gerar relatórios e enviá-los por email. Fazer isso manualmente ou com scripts simples pode rapidamente se tornar complexo e difícil de gerenciar.

O Airflow resolve esse problema permitindo que você defina esses processos como código Python, organizando-os em fluxos de trabalho chamados DAGs (Grafos Acíclicos Direcionados). Ele não apenas executa essas tarefas na ordem correta, mas também monitora o progresso, tenta novamente em caso de falha, e fornece uma interface visual para acompanhar tudo.

**Fontes:**

- Apache Airflow - Wikipedia (https://en.wikipedia.org/wiki/Apache_Airflow)
- Apache Airflow Explained: A Beginner-Friendly Guide | DigitalOcean (https://www.digitalocean.com/community/tutorials/apache-airflow-explained-beginner-guide)
- An introduction to Apache Airflow | Astronomer Docs (https://www.astronomer.io/docs/learn/intro-to-airflow)

## Um Pouco da História

O Airflow foi criado em outubro de 2014 por Maxime Beauchemin, na época engenheiro de dados do Airbnb. A empresa estava crescendo rapidamente e suas necessidades de processamento de dados se tornavam cada vez mais complexas. As ferramentas disponíveis, como o Cron (agendador de tarefas do Linux), não conseguiam lidar com as dependências complexas entre as diferentes tarefas de dados.

**Fontes:**

- Project — Airflow 3.1.3 Documentation (https://airflow.apache.org/docs/apache-airflow/stable/project.html)
- Apache Airflow - Wikipedia (https://en.wikipedia.org/wiki/Apache_Airflow)

## Problemas Que o Airflow Veio Para Resolver

Antes do Airflow, equipes de dados enfrentavam diversos desafios ao gerenciar seus processos:

**Gerenciamento de dependências complexas:** Quando você tem tarefas que dependem umas das outras (como "só processar dados depois que a extração terminar"), ferramentas simples não conseguem garantir a ordem correta de execução. O Airflow organiza essas dependências de forma visual e automática.

**Agendamento centralizado:** Usar ferramentas como Cron significava ter scripts espalhados por diferentes servidores, sem uma visão unificada do que estava acontecendo. O Airflow oferece um local central para definir e visualizar todos os fluxos de trabalho.

**Monitoramento e tratamento de erros:** Quando algo dá errado em um processo de dados às 3 da manhã, você precisa saber o que falhou e idealmente ter tentativas automáticas de reexecução. O Airflow fornece logs detalhados, alertas e políticas de retry configuráveis.

**Workflows como código:** Escrever processos de dados usando Python permite versionar seu trabalho (usando Git, por exemplo), testar antes de colocar em produção e reutilizar componentes. Isso traz práticas de engenharia de software para a área de dados.

**Integração entre sistemas diversos:** Empresas modernas usam múltiplas ferramentas - bancos de dados, data warehouses na nuvem, APIs externas, sistemas de BI. O Airflow facilita a conexão entre todos esses sistemas através de "operators" pré-construídos e extensíveis.

**Escalabilidade:** Conforme o volume de dados cresce e os processos se tornam mais complexos, o Airflow pode escalar distribuindo a carga entre múltiplos servidores, algo que scripts simples não conseguem fazer.

**Fontes:**

- Airflow: a workflow management platform | Airbnb Tech Blog (http://nerds.airbnb.com/airflow/)
- The Good and the Bad of Apache Airflow Pipeline Orchestration (https://www.altexsoft.com/blog/apache-airflow-pros-cons/)
- Apache Airflow Explained: A Beginner-Friendly Guide | DigitalOcean (https://www.digitalocean.com/community/tutorials/apache-airflow-explained-beginner-guide)
- Apache Airflow Orchestration | Qubole (https://www.qubole.com/blog/apache-airflow-tutorial-etl-elt-workflow-orchestration-made-easy)
- Mastering Apache Airflow: Orchestrating Scalable and Efficient Data Workflows (https://softwarefrontier.substack.com/p/mastering-apache-airflow-orchestrating)

---

# Componentes Principais do Apache Airflow em Docker Compose

O Apache Airflow utiliza uma arquitetura modular e distribuída. Quando implementado via Docker Compose, diferentes serviços trabalham em conjunto para processar, agendar e executar tarefas de forma eficiente. Esta seção apresenta os componentes essenciais dessa arquitetura.

## 1. Scheduler

O Scheduler é o cérebro do Airflow, responsável por monitorar continuamente os DAGs e determinar quando as tarefas devem ser executadas. Ele consulta o banco de metadados para verificar o estado de cada tarefa e submete as tarefas prontas para o Executor.

O Scheduler realiza três operações principais: verifica se há DAGs que precisam de uma nova execução e as cria, examina lotes de DAGRuns para identificar TaskInstances agendáveis ou DAGRuns completos, e seleciona TaskInstances agendáveis respeitando limites de pool e concorrência antes de enfileirá-las para execução.

Uma característica importante é que o Executor não é um processo separado, mas sim uma propriedade de configuração que roda dentro do processo do Scheduler. O Scheduler pode operar em modo de alta disponibilidade, onde múltiplas instâncias trabalham simultaneamente consultando o mesmo banco de metadados.

**Fontes:**

- Architecture Overview — Airflow 3.1.5 Documentation (https://airflow.apache.org/docs/apache-airflow/stable/core-concepts/overview.html)
- Scheduler — Airflow Documentation (https://airflow.apache.org/docs/apache-airflow/2.2.3/concepts/scheduler.html)

## 2. Webserver

O Webserver fornece a interface gráfica do Airflow, permitindo que usuários visualizem, inspecionem, acionem e depurem o comportamento de DAGs e tarefas. Através desta interface, é possível monitorar o progresso das tarefas, acessar logs de execução, pausar ou ativar workflows e editar configurações.

Em uma arquitetura distribuída, o Webserver não possui acesso direto aos arquivos de DAG. O código exibido na aba "Code" da interface é lido do banco de metadados, garantindo isolamento de segurança. O Webserver não pode executar código submetido pelo autor do DAG, apenas código instalado como pacote ou plugin pelo gerente de deployment.

Por padrão, o serviço fica disponível na porta 8080, com credenciais iniciais configuráveis através de variáveis de ambiente.

**Fontes:**

- Architecture Overview — Airflow 3.1.5 Documentation (https://airflow.apache.org/docs/apache-airflow/stable/core-concepts/overview.html)
- Airflow Setup | Wes's Webzone (https://wesmattison.com/posts/airflow-setup/)

## 3. Executor

O Executor é o mecanismo que determina como e onde as tarefas serão executadas. Embora existam vários tipos de executors disponíveis, os mais comuns em Docker Compose são o LocalExecutor e o CeleryExecutor.

O LocalExecutor executa tarefas em paralelo no mesmo processo do Scheduler, utilizando multiprocessing. É adequado para instalações de produção em máquina única, oferece baixa latência e simplicidade de configuração. Requer bancos de dados que suportam múltiplas conexões, como PostgreSQL ou MySQL.

O CeleryExecutor distribui tarefas através de um sistema de filas de mensagens utilizando o framework Celery. Permite escalonamento horizontal distribuindo a carga entre múltiplos workers, cada um podendo residir em máquinas diferentes. Requer componentes adicionais como Redis ou RabbitMQ como message broker, proporcionando tolerância a falhas e capacidade de processamento distribuído.

A escolha entre eles depende dos requisitos: LocalExecutor para configurações simples e de menor escala, CeleryExecutor para ambientes que necessitam distribuição de carga e escalonamento horizontal.

**Fontes:**

- Executor — Airflow 3.1.5 Documentation (https://airflow.apache.org/docs/apache-airflow/stable/core-concepts/executor/index.html)
- Apache Airflow Executors | Astronomer Docs (https://www.astronomer.io/docs/learn/airflow-executors-explained)
- Airflow Executors (Sequential, Local, Celery) (https://www.sparkcodehub.com/airflow/fundamentals/executors)

## 4. Workers

Workers são processos separados responsáveis pela execução efetiva das tarefas. Na configuração básica com LocalExecutor, os workers podem ser parte do próprio Scheduler. Com CeleryExecutor, os workers são processos independentes que pegam tarefas da fila de mensagens.

Cada worker se conecta ao banco de metadados para atualizar o estado das tarefas e ao message broker (quando usando Celery) para receber novas tarefas. Workers podem ser escalados horizontalmente adicionando mais instâncias através do Docker Compose, permitindo distribuir a carga de trabalho.

Os workers têm acesso aos arquivos de DAG sincronizados através de volumes compartilhados, garantindo que possam executar o código definido pelos autores dos workflows.

**Fontes:**

- Deploying Apache Airflow with Docker Compose for Beginners (https://medium.com/@dinhnguyen1812/deploying-apache-airflow-with-docker-compose-for-beginners-5bc2e120ceff)
- Apache Airflow with 3 Celery workers in docker-compose (https://xnuinside.medium.com/quick-tutorial-apache-airflow-with-3-celery-workers-in-docker-composer-9f2f3b445e4)

## 5. Metadata Database (PostgreSQL)

O banco de metadados é um componente crítico que armazena todo o estado do Airflow. O PostgreSQL é o banco mais utilizado em ambientes de produção, armazenando informações sobre DAGs, DAGRuns, TaskInstances, variáveis, conexões, histórico de execuções e configurações de usuários.

Todos os componentes do Airflow interagem com este banco: o Scheduler para gerenciar o estado das tarefas, o Webserver para exibir informações na interface, e os Workers para reportar o progresso da execução. A escolha de PostgreSQL ou MySQL é necessária para habilitar paralelismo e suportar configurações de alta disponibilidade do Scheduler.

A string de conexão segue o formato: `postgresql+psycopg2://usuario:senha@host:porta/database`. Em Docker Compose, o serviço PostgreSQL expõe a porta 5432 internamente, com dados persistidos através de volumes.

**Fontes:**

- Architecture Overview — Airflow 3.1.5 Documentation (https://airflow.apache.org/docs/apache-airflow/stable/core-concepts/overview.html)
- Apache/Airflow and PostgreSQL with Docker and Docker Compose (https://towardsdatascience.com/apache-airflow-and-postgresql-with-docker-and-docker-compose-5651766dfa96/)
- Understanding the Airflow Metadata Database | Astronomer Docs

## 6. Redis (Message Broker)

O Redis funciona como message broker quando se utiliza o CeleryExecutor, gerenciando a fila de tarefas entre o Scheduler e os Workers. É um armazenamento de chave-valor em memória de alta performance que garante distribuição eficiente das tarefas e comunicação confiável entre componentes.

O Redis assegura que tarefas sejam distribuídas de forma balanceada entre workers, mantém a fila de tarefas pendentes, e possibilita comunicação rápida com baixa latência. O serviço geralmente roda na porta 6379 e requer configuração tanto no Scheduler quanto nos Workers através das variáveis `AIRFLOW__CELERY__BROKER_URL` e `AIRFLOW__CELERY__RESULT_BACKEND`.

Enquanto RabbitMQ é uma alternativa ao Redis como message broker, o Redis é frequentemente preferido por sua simplicidade de deployment, facilidade de manutenção e menor overhead operacional.

**Fontes:**

- Airflow Setup | Wes's Webzone (https://wesmattison.com/posts/airflow-setup/)
- How to Scale-out Apache Airflow 2.0 with Redis and Celery (https://medium.com/codex/how-to-scale-out-apache-airflow-2-0-with-redis-and-celery-3e668e003b5c)

## 7. DAG Processor

O DAG Processor é responsável por fazer o parsing dos arquivos de DAG e serializá-los no banco de metadados. Em instalações mais simples, essa função pode ser executada pelo próprio Scheduler, mas em arquiteturas mais complexas e seguras, o DAG Processor pode ser separado como um componente independente.

A separação do DAG Processor permite maior isolamento de segurança, garantindo que o Scheduler não tenha acesso direto aos arquivos de DAG e não possa executar código fornecido pelo autor do DAG. Isso é especialmente importante em ambientes onde múltiplos usuários com diferentes níveis de confiança submetem workflows.

O processo envolve verificação de novos arquivos, exclusão de arquivos processados recentemente que não foram modificados, enfileiramento de caminhos de arquivos descobertos, processamento dos arquivos iniciando um DagFileProcessorProcess para cada um, e coleta de resultados dos processadores finalizados.

**Fontes:**

- Architecture Overview — Airflow 3.1.5 Documentation (https://airflow.apache.org/docs/apache-airflow/stable/core-concepts/overview.html)
- Scheduler — Airflow Documentation (https://airflow.apache.org/docs/apache-airflow/2.2.3/concepts/scheduler.html)
- Modular Deployment of Apache Airflow 3 with Docker Compose (https://medium.com/@lucasmassucci/modular-deployment-of-apache-airflow-3-with-docker-compose-in-homelab-or-desktop-environments-f5662ca744aa)

## 8. Triggerer (Opcional)

O Triggerer é um componente opcional que executa tarefas diferidas (deferred tasks) em um event loop assíncrono. Em instalações básicas onde tarefas diferidas não são utilizadas, um Triggerer não é necessário.

Tarefas diferidas permitem que operadores liberem recursos enquanto aguardam eventos externos, como a conclusão de um job em um sistema externo ou a disponibilidade de um arquivo. O Triggerer monitora esses eventos de forma eficiente usando asyncio, permitindo que o Airflow gerencie milhares de tarefas em espera com overhead mínimo.

Este componente é particularmente útil para sensores que aguardam condições externas por longos períodos, evitando o desperdício de slots de worker.

**Fontes:**

- Architecture Overview — Airflow 3.1.5 Documentation (https://airflow.apache.org/docs/apache-airflow/stable/core-concepts/overview.html)
- Setting Up Apache Airflow with Docker Locally | Dataquest (https://www.dataquest.io/blog/setting-up-apache-airflow-with-docker-locally-part-i/)
- Modular Deployment of Apache Airflow 3 with Docker Compose (https://medium.com/@lucasmassucci/modular-deployment-of-apache-airflow-3-with-docker-compose-in-homelab-or-desktop-environments-f5662ca744aa)

## Considerações Finais

A arquitetura do Apache Airflow em Docker Compose oferece flexibilidade para diferentes cenários de deployment. Para ambientes de desenvolvimento ou pequena escala, uma configuração com LocalExecutor, Webserver, Scheduler e PostgreSQL pode ser suficiente. Para produção com necessidade de escalonamento, adicionar Redis, múltiplos Workers com CeleryExecutor, e componentes como Triggerer proporciona robustez e capacidade de crescimento.

A containerização via Docker facilita o isolamento de serviços, reprodutibilidade do ambiente e gestão de dependências, tornando o deployment e manutenção do Airflow mais acessíveis e confiáveis.