## O que é o AirFlow

O Apache Airflow é um sistema de código aberto para criar, agendar e monitorar fluxos de trabalho. Um fluxo de trabalho do Airflow é um DAG (grafo acíclico direcionado), que é uma série de tarefas com relacionamentos claramente definidos (dependências) e sem ciclos (uma tarefa não pode se repetir upstream). A execução de várias tarefas relacionadas a dados periodicamente, como: extrair informações de um banco de dados, processar esses dados, gerar relatórios e enviá-los por email. Fazer isso manualmente ou com scripts simples pode rapidamente se tornar complexo e difícil de gerenciar.

O Airflow resolve esse problema permitindo que você defina esses processos como código Python, organizando-os em fluxos de trabalho chamados DAGs (Grafos Acíclicos Direcionados). Ele não apenas executa essas tarefas na ordem correta, mas também monitora o progresso, tenta novamente em caso de falha, e fornece uma interface visual para acompanhar tudo.
Além disso, eles precisam ser executados em uma ordem muito específica. Ferramentas de orquestração de fluxo de trabalho como o Apache Airflow ajudam a garantir que todas essas etapas sejam executadas no momento certo e na ordem correta. Elas também facilitam o monitoramento e gerenciamento dos pipelines.

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

- **Gerenciamento de dependências complexas:** Quando você tem tarefas que dependem umas das outras (como "só processar dados depois que a extração terminar"), ferramentas simples não conseguem garantir a ordem correta de execução. O Airflow organiza essas dependências de forma visual e automática.

- **Agendamento centralizado:** Usar ferramentas como Cron significava ter scripts espalhados por diferentes servidores, sem uma visão unificada do que estava acontecendo. O Airflow oferece um local central para definir e visualizar todos os fluxos de trabalho.

- **Monitoramento e tratamento de erros:** Quando algo dá errado em um processo de dados às 3 da manhã, você precisa saber o que falhou e idealmente ter tentativas automáticas de reexecução. O Airflow fornece logs detalhados, alertas e políticas de retry configuráveis.

- **Workflows como código:** Escrever processos de dados usando Python permite versionar seu trabalho (usando Git, por exemplo), testar antes de colocar em produção e reutilizar componentes. Isso traz práticas de engenharia de software para a área de dados.

- **Integração entre sistemas diversos:** Empresas modernas usam múltiplas ferramentas - bancos de dados, data warehouses na nuvem, APIs externas, sistemas de BI. O Airflow facilita a conexão entre todos esses sistemas através de "operators" pré-construídos e extensíveis.

- **Escalabilidade:** Conforme o volume de dados cresce e os processos se tornam mais complexos, o Airflow pode escalar distribuindo a carga entre múltiplos servidores, algo que scripts simples não conseguem fazer.

**Fontes:**
- Airflow: a workflow management platform | Airbnb Tech Blog (http://nerds.airbnb.com/airflow/)
- The Good and the Bad of Apache Airflow Pipeline Orchestration (https://www.altexsoft.com/blog/apache-airflow-pros-cons/)
- Apache Airflow Explained: A Beginner-Friendly Guide | DigitalOcean (https://www.digitalocean.com/community/tutorials/apache-airflow-explained-beginner-guide)
- Apache Airflow Orchestration | Qubole (https://www.qubole.com/blog/apache-airflow-tutorial-etl-elt-workflow-orchestration-made-easy)
- Mastering Apache Airflow: Orchestrating Scalable and Efficient Data Workflows (https://softwarefrontier.substack.com/p/mastering-apache-airflow-orchestrating)
---
# Tradução livre do artigo: Apache Airflow Explained: A Beginner-Friendly Guide | DigitalOcean 
## Componentes Principais do Airflow 

Abaixo está uma breve visão geral dos blocos de construção principais do Apache Airflow. Use esta tabela como referência rápida para entender o que cada componente é e o que ele faz.

| Componente                                          | O que é                                                                                                | Responsabilidades principais                                                                                                                  |
| --------------------------------------------------- | ------------------------------------------------------------------------------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------- |
| **DAGs (Grafos Acíclicos Direcionados)**            | O blueprint do fluxo de trabalho escrito em Python; um grafo de tarefas com dependências e sem ciclos. | Define quais tarefas executam e em que ordem; codifica cronograma e metadados; permite visualização na UI.                                    |
| **Tasks (Tarefas)**                                 | A menor unidade de trabalho (nó no grafo DAG) instanciada de operadores ou funções TaskFlow.           | Executa uma ação específica (ex: executar SQL, chamar Python, acionar API, mover arquivos); reporta sucesso/falha.                            |
| **Scheduler (Agendador)**                           | Um serviço persistente que avalia DAGs e estados de tarefas continuamente.                             | Determina _quando_ cada tarefa deve executar baseado em cronogramas e dependências; cria instâncias de tarefas; gerencia retries e backfills. |
| **Executor (e Workers)**                            | O backend de execução e processos que realmente executam tarefas.                                      | Lança instâncias de tarefas no backend escolhido (processo/thread local, workers Celery, pods Kubernetes); retorna resultados ao Agendador.   |
| **Web Server (Web UI)**                             | A interface de usuário operacional servida pelo webserver do Airflow.                                  | Observabilidade e controle: visualizar DAGs, acionar/limpar tarefas, inspecionar logs, pausar DAGs, gerenciar conexões/variáveis.             |
| **Metadata Database (Banco de Dados de Metadados)** | O sistema persistente de registro (tipicamente PostgreSQL/MySQL).                                      | Armazena execuções de DAG, instâncias de tarefas e estados, configurações, metadados de logs e histórico operacional.                         |

---

## Como o Airflow Funciona (Arquitetura)

Aqui está uma visão geral passo a passo de **como o Airflow processa um fluxo de trabalho (DAG)**:
![[Pasted image 20251215202311.png]]
1. **Criando o DAG:** O autor do DAG escreve um script Python que define um DAG. Este script inclui coisas como o cronograma (ex: "executar diariamente às 9h") ou configurações de acionamento manual. Ele também contém as definições das tarefas (via operadores ou funções @task), bem como as dependências das tarefas (ex: task1 >> task2 (task2 espera task1 terminar)). O arquivo Python é salvo na pasta DAGs do Airflow para ser encontrado pelo Airflow. O Airflow lerá (analisará) este arquivo periodicamente para carregar a definição do DAG.
    
2. **Agendamento e Análise do DAG (Scheduler):** O Agendador do Airflow é um processo que executa continuamente em segundo plano. Ele verifica se cada DAG deve ser executado com base em seu cronograma ou acionadores. Por exemplo, um DAG com um cronograma de "diário" pode ser capturado pelo agendador, e observar que uma nova execução deve ser agendada à meia-noite. O Agendador do Airflow também verifica se tarefas dentro dos DAGs estão prontas para executar – significando que todas as suas dependências upstream estão completas e seu horário agendado (data lógica) chegou.
    
3. **Executando Tarefas (Executor):** Cada tarefa a ser executada é entregue ao Executor pelo Agendador. O executor decide onde e como executar a tarefa. Pode ser executado localmente ou enviado para uma fila de workers. Para uma arquitetura distribuída, a tarefa será adicionada a uma fila e então selecionada por processos Worker que executam tarefas.
    
4. **Atualizando Estado e Tratamento de Erros:** Cada instância de tarefa reporta seu status (sucesso, falha, etc.) de volta ao agendador conforme ela completa, e o agendador atualiza seu banco de dados de metadados com o estado da tarefa. O rastreamento de estado é o que permite ao agendador saber o que fazer em seguida – por exemplo, se a Tarefa A foi concluída com sucesso, ele sabe que pode agendar a Tarefa B agora, ou se a Tarefa A falhou com retries restantes, ele agenda uma nova tentativa, ou se falhou sem mais retries, ele marca a execução do DAG como falhada. Se uma tarefa falha, o Airflow tem lógica para retentar automaticamente a tarefa se configurado para retries. Se continuar a falhar após as retries, o Airflow pode alertar sobre falhas (enviar email ou mensagem Slack, etc.). Todos esses são configuráveis como parte dos parâmetros do DAG ou tarefa (por exemplo, número de retries, atraso de retry, email para enviar no alerta).
    
5. **Monitoramento e Intervenção:** A UI Web do Airflow fornece uma interface ao vivo do processo acima. Você pode ver uma execução do DAG aparecer (por exemplo, uma nova linha na visualização Tree para uma execução de hoje), e assistir os status das tarefas mudarem conforme elas progridem. A visualização Graph visualiza as tarefas do DAG e mostra quais foram concluídas ou falharam.
    
6. **Limpeza e Próxima Execução:** Quando uma execução do DAG é concluída (todas as tarefas foram bem-sucedidas ou foram puladas/falharam), o Airflow move para a próxima execução agendada. Todos os metadados sobre execuções passadas ainda são mantidos no banco de dados, e você pode sempre verificar o histórico. O agendador continuará a acionar novas execuções de acordo com o cronograma ou acionamento manual. Este processo é repetido até que o processo do Airflow seja encerrado.

## Entendendo DAGs no Airflow

Um DAG no Airflow define o cronograma e as tarefas e dependências necessárias para executar um fluxo de trabalho. Um DAG não precisa saber o que está dentro de cada tarefa; ele apenas precisa definir quando e em que ordem elas devem executar.

### Tarefas, Operadores e Sensores

Tarefas são as unidades básicas de trabalho no Airflow. No Airflow, tarefas são instâncias de operadores, que são templates para ações predefinidas. O Airflow vem com uma variedade de operadores principais. O _BashOperator_ permite executar comandos shell, e o PythonOperator permite chamar funções Python. O Airflow também fornece um decorador _@task_, que permite transformar uma função Python regular em uma tarefa.

Além dos operadores, o Airflow fornece sensores. Sensores são tarefas que esperam uma condição ou evento ser atendido antes de terem sucesso. Podem ser usados para coisas como esperar um arquivo aparecer ou uma tabela ser populada.

### Dependências de Tarefas

Para declarar dependências entre tarefas, você usa os operadores >> e << (recomendado) ou métodos como _set_upstream_ e _set_downstream_. Também é possível encadear tarefas, criar dependências cross-downstream e até construir listas de tarefas dinamicamente. Um DAG sem dependências seria apenas um conjunto independente de tarefas.

### Melhores Práticas para Design de DAG

| Melhor Prática                                       | Descrição                                                                                                                                                                                         | Dicas Práticas / Exemplos                                                                                                                                              |
| ---------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Garantir que DAGs são verdadeiramente acíclicos**  | DAGs não devem conter ciclos. Em fluxos de trabalho complexos, é fácil introduzir loops indiretos (ex: A → B → C → A). O Airflow detecta ciclos automaticamente e se recusa a executar tais DAGs. | Revisar dependências regularmente. Usar visualização Graph para confirmar visualmente que o DAG não tem loops. Refatorar DAGs complexos para evitar ciclos acidentais. |
| **Manter tarefas idempotentes e pequenas**           | Cada tarefa deve realizar uma única etapa lógica e ser segura para reexecutar sem corromper dados. Tarefas idempotentes garantem que retries não produzam resultados inconsistentes.              | Usar padrões "insert or replace". Escrever em arquivos temporários antes de confirmar resultados. Dividir scripts grandes em múltiplas tarefas.                        |
| **Usar IDs descritivos e documentação**              | Nomenclatura clara e documentação melhoram legibilidade e manutenibilidade tanto no código quanto na UI do Airflow.                                                                               | Usar IDs significativos para DAG e tarefas. Adicionar documentação usando `dag.doc_md` ou `task.doc_md`. Alinhar nomes com lógica de negócio.                          |
| **Aproveitar recursos do Airflow**                   | Usar capacidades integradas do Airflow para comunicação, tratamento de credenciais e coordenação de pipeline ao invés de lógica personalizada.                                                    | Usar XComs para compartilhamento de pequenos dados. Usar Variables e Connections para config e secrets. Usar operadores e hooks integrados sempre que possível.        |
| **Evitar sobrecarregar o agendador**                 | Números enormes de DAGs ou computações pesadas dentro de arquivos DAG podem desacelerar o agendador. DAGs devem ser leves para analisar.                                                          | Monitorar performance do agendador. Dividir DAGs massivos em menores. Evitar chamadas API ou computação pesada durante importação do DAG.                              |
| **Definir estratégia clara de tratamento de falhas** | Planejar como falhas devem ser tratadas. Nem todos os erros devem ter retry, e tarefas de longa execução podem precisar de SLAs ou alertas.                                                       | Usar retries apenas para falhas transitórias. Usar Sensores e triggers para dependências baseadas em eventos. Definir SLAs para tarefas longas ou críticas.            |
## Casos de Uso Comuns para o Airflow

| Caso de Uso                                        | Descrição                                                                                                               | Como o Airflow Ajuda / Benefícios                                                                                                                                                                                                                                                                                                                                      |
| -------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Pipelines ETL/ELT de Dados**                     | Mover dados de múltiplas fontes para um data warehouse ou data lake usando padrões ETL (Extract–Transform–Load) ou ELT. | DAGs orquestram extração → transformação → carga na ordem correta. Airflow integra com muitos bancos de dados e sistemas de armazenamento. Se uma etapa falha (ex: API fora do ar), tarefas downstream são paradas e alertas podem ser acionados, melhorando confiabilidade para sincronizações em lote e incrementais.                                                |
| **Data Warehousing e Relatórios de BI**            | Preparar e atualizar dados de análise usados em dashboards e relatórios.                                                | Agenda jobs diários ou periódicos para que relatórios sempre executem contra dados atualizados. Coordena cargas de trabalho SQL, verificações de qualidade e etapas de relatórios. Fornece monitoramento e notificações para que falhas (ex: agregações quebradas) sejam visíveis ao invés de silenciosamente corromper saídas de BI.                                  |
| **Pipelines de Aprendizado de Máquina**            | Automatizar fluxos de trabalho ML de ponta a ponta desde preparação de dados até implantação.                           | Representa cada estágio ML como uma tarefa em um DAG e reforça a ordenação correta. Pode persistir artefatos entre etapas (dados pré-processados, binários de modelo). Integra com frameworks ML e operadores Kubernetes e permite retreinamento agendado e orquestração de experimentos durante horários de baixo uso.                                                |
| **Verificações de Qualidade de Dados e Validação** | Executar verificações automatizadas para garantir que dados sejam completos, consistentes e confiáveis.                 | Agenda DAGs de qualidade de dados recorrentes (diário/semanal). Coordena verificações através de bancos de dados e scripts de QA, alerta equipes sobre anomalias e pode acionar tarefas de correção downstream como parte de engenharia de confiabilidade de dados.                                                                                                    |
| **Manutenção de DB Transacional e Backups**        | Automatizar tarefas operacionais de rotina para bancos de dados de produção e infraestrutura.                           | Centraliza agendamento e monitoramento de tarefas de manutenção. Garante que backups e manutenção executem consistentemente em horários definidos, reduz esforço manual e erro humano, e fornece logs/histórico para que equipes possam confirmar que jobs críticos de manutenção foram concluídos com sucesso.                                                        |
| **Integração / Automação de Fluxo de Trabalho**    | Orquestrar fluxos de trabalho de negócio ou sistema que abrangem múltiplos serviços e APIs.                             | Atua como uma camada flexível de "cola" que conecta serviços. DAGs codificam ramificação complexa e lógica condicional. Operadores e tarefas Python permitem integrações personalizadas. Fornece um lugar central para gerenciar, monitorar e retentar etapas de processo de negócio ao invés de espalhar lógica de automação através de scripts e ferramentas ad-hoc. |

---

## Quando NÃO Usar o Airflow

Embora o Airflow seja poderoso, ele não é a ferramenta certa para todas as situações. Entender suas limitações ajuda você a escolher a abordagem de orquestração correta:

- **Cargas de Trabalho em Tempo Real ou Streaming:** O Airflow é orientado a lote com início e fim bem definidos. Não é para cargas de trabalho de longa execução, orientadas por eventos ou streaming. Se você está ingerindo fluxos contínuos de eventos (cliques de usuário, sensores IoT, etc.) e precisa processá-los com latência muito baixa em tempo real, o Airflow não é a melhor escolha.
    
- **Tarefas Curtas de Alta Frequência ou Muitas:** Este é outro caso onde o Airflow pode não funcionar, ou pelo menos não eficientemente. Tarefas do Airflow têm algum overhead (no rastreamento do banco de dados, etc.), então se você precisa acionar algo a cada poucos segundos ou tem milhares de tarefas minúsculas, o Airflow pode não escalar bem.
    
- **Fluxos de Trabalho Puramente Orientados por Eventos:** Isso é similar ao caso de uso de streaming, mas para algo que deve ser acionado por um evento. "Sempre que um evento X acontece, faça Y" é um caso de uso perfeitamente válido para o Airflow, mas se a única coisa que deve acionar isso é um evento (ex: "sempre que um arquivo chega em um bucket S3, faça X"), então o Airflow não é a solução mais leve aqui. O Airflow geralmente usa sensores (S3KeySensor, por exemplo) para "sentir" a presença de arquivos/eventos. Sensores têm desvantagens pois dependem de polling. Problemas com sensores incluem acionar em arquivos existentes e não necessariamente serem responsivos a eventos (ex: eles fazem polling em intervalos).
    

Em resumo, o Airflow é adequado para fluxos de trabalho que são periódicos, orientados a lote ou complexos de definir e gerenciar. Pode não ser uma boa escolha se seu caso de uso requer processamento em tempo real ou contínuo, jobs ultra-frequentes ou fluxos de trabalho muito simples que não precisam do overhead do Airflow. Se for o caso, você pode querer explorar outras opções ou simplificar sua solução.

---

## Alternativas ao Airflow

O espaço de orquestração de fluxo de trabalho explodiu nos últimos anos, e embora o Airflow seja líder, existem várias **ferramentas alternativas** que vale a pena conhecer:

| Ferramenta / Serviço | Resumo                                                                                                                                                                                                                                                                                                 | Referência                                          |
| -------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | --------------------------------------------------- |
| **Luigi**            | Agendador de fluxo de trabalho Python de código aberto criado pelo Spotify. Bom para construir pipelines em lote com tarefas e dependências em código. Arquitetura mais simples e mais leve que o Airflow, mas com um ecossistema menor e menos integrações integradas.                                | [Documentação Luigi](https://luigi.readthedocs.io/) |
| **Prefect**          | Framework de orquestração nativo Python posicionado como uma alternativa mais moderna e amigável ao desenvolvedor que o Airflow. Usa Flows e Tasks, oferece agendamento, retries e observabilidade. Pode executar totalmente código aberto ou com Prefect Cloud para UI hospedada e plano de controle. | [Site Prefect](https://www.prefect.io/)             |
| **                   |                                                                                                                                                                                                                                                                                                        |                                                     |

**Fonte:**
- Apache Airflow Explained: A Beginner-Friendly Guide | DigitalOcean (https://www.digitalocean.com/community/tutorials/apache-airflow-explained-beginner-guide)