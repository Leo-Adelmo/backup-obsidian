link: https://www.digitalocean.com/community/tutorials/apache-airflow-explained-beginner-guide
# Apache Airflow Explicado: Um Guia para Iniciantes

## Introdução

Empresas modernas orientadas por dados dependem de pipelines de dados que buscam, transformam, enriquecem e movem informações de um local para outro. Esses pipelines podem envolver muitas etapas (extrair dados brutos, limpá-los, treinar modelos de aprendizado de máquina, construir dashboards, etc.). Além disso, eles precisam ser executados em uma ordem muito específica. Ferramentas de orquestração de fluxo de trabalho como o Apache Airflow ajudam a garantir que todas essas etapas sejam executadas no momento certo e na ordem correta. Elas também facilitam o monitoramento e gerenciamento dos pipelines.

O Airflow foi desenvolvido na Airbnb em 2014 e se tornou uma das plataformas de orquestração de fluxo de trabalho de código aberto mais populares. É usado por milhares de organizações para automatizar e monitorar pipelines em lote. Neste artigo, explicaremos o Apache Airflow desde o início. 

---

## O que é o Apache Airflow?

O Apache Airflow é um sistema de código aberto para criar, agendar e monitorar fluxos de trabalho. Um fluxo de trabalho do Airflow é um DAG (grafo acíclico direcionado), que é uma série de tarefas com relacionamentos claramente definidos (dependências) e sem ciclos (uma tarefa não pode se repetir upstream). A filosofia central do Airflow é "configuração como código", então ao invés de usar uma UI com uma interface de arrastar e soltar, você cria scripts Python para representar fluxos de trabalho. Essa abordagem fornece flexibilidade significativa para engenheiros: você pode se conectar a quase qualquer tecnologia via operadores do Airflow e bibliotecas Python. Você também pode aplicar práticas recomendadas de engenharia de software (controle de versão, testes, etc.) às definições de seus pipelines.

**Por que a orquestração de fluxo de trabalho é importante?**

O ponto chave é que frequentemente você tem múltiplas etapas em qualquer fluxo de trabalho de dados que devem ser executadas em sequência. Um exemplo simples é o pipeline ETL (Extract/Transform/Load - Extrair/Transformar/Carregar). Os pipelines podem rapidamente se tornar mais complexos à medida que você adiciona mais tarefas, dependências, ramificações e assim por diante, e também precisam ser executados em uma cadência regular. Intervenção manual ou ferramentas simples de agendamento frequentemente se tornam inadequadas nesses casos. O Airflow resolve isso atuando como um orquestrador central. Ele tem um grafo acíclico direcionado (DAG) de tarefas (nós) com dependências (arestas). Ele sabe quando executar qual tarefa (esperando as dependências upstream serem concluídas antes de executar), incluindo agendamento (execução em intervalos periódicos), tratamento de erros (retries automáticos, alertas) e logging.

Se considerarmos o pipeline de dados inteiro como uma receita, então cada tarefa individual é uma sub-etapa (cortar, ferver, pôr no prato, etc.). O Airflow é como o gerente da cozinha que conhece as receitas e mantém um olho em todas as diferentes sub-etapas acontecendo em cada cozinha, certificando-se de que tudo aconteça na ordem correta e no momento certo.

Sem orquestração de fluxo de trabalho, você pode ter todos os chefs e ingredientes, mas eles não estão coordenados. O Airflow fornece essa coordenação, permitindo a produção do "prato" desejado (execução do fluxo de trabalho).

---

## Por que as Equipes de Dados Usam o Airflow?

O Airflow se tornou popular entre engenheiros de dados, engenheiros de ML e equipes de DevOps porque aborda muitas necessidades de pipelines de dados complexos. Aqui estão algumas grandes razões pelas quais as equipes escolhem o Airflow:

- **Orquestração de Fluxos de Trabalho e Dependências Complexas:** O Airflow foi projetado para fluxos de trabalho com um grande número de etapas interdependentes. O Airflow torna fácil declarar requisitos upstream e downstream, garantindo que as tarefas sejam executadas na ordem apropriada. Se a tarefa C depende das tarefas A e B, o agendador do Airflow está ciente desses relacionamentos e garantirá que A e B sejam executados (potencialmente em paralelo) antes de executar C. O Airflow pode lidar com dependências complexas, lógica de ramificação, etc.
    
- **Agendamento Centralizado (Cron com Esteroides):** Ele fornece um agendador que permite executar tarefas em um determinado cronograma. Você poderia definir dezenas ou centenas de pipelines que acionam tarefas em diferentes horários (por hora, diariamente, mensalmente) usando o agendador do Airflow. O Airflow entende os relacionamentos das tarefas. Ele pode agendar e acionar tarefas inteligentemente para serem executadas. O agendador central libera você de ter que escrever scripts diferentes em servidores diferentes.
    
- **Escalabilidade e Execução Distribuída:** O Airflow também pode escalar com seus fluxos de trabalho. Ele pode ser configurado para executar tarefas em paralelo através de múltiplos processos worker ou máquinas. Ele também fornece múltiplos executores (discutidos mais tarde neste artigo) que podem executar tarefas em sistemas distribuídos como Celery ou Kubernetes através de um cluster de nós. Isso significa que você pode paralelizar tarefas pesadas (processamento de big data) e ter múltiplos pipelines executando simultaneamente.
    
- **Monitoramento, Logging e Alertas:** Ele vem com uma UI web rica que fornece visibilidade em tempo real de seus fluxos de trabalho. Você pode ver quais tarefas tiveram sucesso e quais falharam, e visualizar a saída de log para cada tarefa. Você pode até acionar manualmente e/ou retentar tarefas via interface web.
    

---

## Componentes Principais do Airflow

Abaixo está uma breve visão geral dos blocos de construção principais do Apache Airflow. Use esta tabela como referência rápida para entender o que cada componente é e o que ele faz.

|Componente|O que é|Responsabilidades principais|
|---|---|---|
|**DAGs (Grafos Acíclicos Direcionados)**|O blueprint do fluxo de trabalho escrito em Python; um grafo de tarefas com dependências e sem ciclos.|Define quais tarefas executam e em que ordem; codifica cronograma e metadados; permite visualização na UI.|
|**Tasks (Tarefas)**|A menor unidade de trabalho (nó no grafo DAG) instanciada de operadores ou funções TaskFlow.|Executa uma ação específica (ex: executar SQL, chamar Python, acionar API, mover arquivos); reporta sucesso/falha.|
|**Scheduler (Agendador)**|Um serviço persistente que avalia DAGs e estados de tarefas continuamente.|Determina _quando_ cada tarefa deve executar baseado em cronogramas e dependências; cria instâncias de tarefas; gerencia retries e backfills.|
|**Executor (e Workers)**|O backend de execução e processos que realmente executam tarefas.|Lança instâncias de tarefas no backend escolhido (processo/thread local, workers Celery, pods Kubernetes); retorna resultados ao Agendador.|
|**Web Server (Web UI)**|A interface de usuário operacional servida pelo webserver do Airflow.|Observabilidade e controle: visualizar DAGs, acionar/limpar tarefas, inspecionar logs, pausar DAGs, gerenciar conexões/variáveis.|
|**Metadata Database (Banco de Dados de Metadados)**|O sistema persistente de registro (tipicamente PostgreSQL/MySQL).|Armazena execuções de DAG, instâncias de tarefas e estados, configurações, metadados de logs e histórico operacional.|

---

## Como o Airflow Funciona (Arquitetura)

Aqui está uma visão geral passo a passo de **como o Airflow processa um fluxo de trabalho (DAG)**:
![[Pasted image 20251215202311.png]]
1. **Criando o DAG:** O autor do DAG escreve um script Python que define um DAG. Este script inclui coisas como o cronograma (ex: "executar diariamente às 9h") ou configurações de acionamento manual. Ele também contém as definições das tarefas (via operadores ou funções @task), bem como as dependências das tarefas (ex: task1 >> task2 (task2 espera task1 terminar)). O arquivo Python é salvo na pasta DAGs do Airflow para ser encontrado pelo Airflow. O Airflow lerá (analisará) este arquivo periodicamente para carregar a definição do DAG.
    
2. **Agendamento e Análise do DAG:** O Agendador do Airflow é um processo que executa continuamente em segundo plano. Ele verifica se cada DAG deve ser executado com base em seu cronograma ou acionadores. Por exemplo, um DAG com um cronograma de "diário" pode ser capturado pelo agendador, e observar que uma nova execução deve ser agendada à meia-noite. O Agendador do Airflow também verifica se tarefas dentro dos DAGs estão prontas para executar – significando que todas as suas dependências upstream estão completas e seu horário agendado (data lógica) chegou.
    
3. **Executando Tarefas:** Cada tarefa a ser executada é entregue ao Executor pelo Agendador. O executor decide onde e como executar a tarefa. Pode ser executado localmente ou enviado para uma fila de workers. Para uma arquitetura distribuída, a tarefa será adicionada a uma fila e então selecionada por processos Worker que executam tarefas.
    
4. **Atualizando Estado e Tratamento de Erros:** Cada instância de tarefa reporta seu status (sucesso, falha, etc.) de volta ao agendador conforme ela completa, e o agendador atualiza seu banco de dados de metadados com o estado da tarefa. O rastreamento de estado é o que permite ao agendador saber o que fazer em seguida – por exemplo, se a Tarefa A foi concluída com sucesso, ele sabe que pode agendar a Tarefa B agora, ou se a Tarefa A falhou com retries restantes, ele agenda uma nova tentativa, ou se falhou sem mais retries, ele marca a execução do DAG como falhada. Se uma tarefa falha, o Airflow tem lógica para retentar automaticamente a tarefa se configurado para retries. Se continuar a falhar após as retries, o Airflow pode alertar sobre falhas (enviar email ou mensagem Slack, etc.). Todos esses são configuráveis como parte dos parâmetros do DAG ou tarefa (por exemplo, número de retries, atraso de retry, email para enviar no alerta).
    
5. **Monitoramento e Intervenção:** A UI Web do Airflow fornece uma interface ao vivo do processo acima. Você pode ver uma execução do DAG aparecer (por exemplo, uma nova linha na visualização Tree para uma execução de hoje), e assistir os status das tarefas mudarem conforme elas progridem. A visualização Graph visualiza as tarefas do DAG e mostra quais foram concluídas ou falharam.
    
6. **Limpeza e Próxima Execução:** Quando uma execução do DAG é concluída (todas as tarefas foram bem-sucedidas ou foram puladas/falharam), o Airflow move para a próxima execução agendada. Todos os metadados sobre execuções passadas ainda são mantidos no banco de dados, e você pode sempre verificar o histórico. O agendador continuará a acionar novas execuções de acordo com o cronograma ou acionamento manual. Este processo é repetido até que o processo do Airflow seja encerrado.
    

---

## Entendendo DAGs no Airflow

Um DAG no Airflow define o cronograma e as tarefas e dependências necessárias para executar um fluxo de trabalho. Um DAG não precisa saber o que está dentro de cada tarefa; ele apenas precisa definir quando e em que ordem elas devem executar.

### Tarefas, Operadores e Sensores

Tarefas são as unidades básicas de trabalho no Airflow. No Airflow, tarefas são instâncias de operadores, que são templates para ações predefinidas. O Airflow vem com uma variedade de operadores principais. O _BashOperator_ permite executar comandos shell, e o PythonOperator permite chamar funções Python. O Airflow também fornece um decorador _@task_, que permite transformar uma função Python regular em uma tarefa.

Além dos operadores, o Airflow fornece sensores. Sensores são tarefas que esperam uma condição ou evento ser atendido antes de terem sucesso. Podem ser usados para coisas como esperar um arquivo aparecer ou uma tabela ser populada.

### Dependências de Tarefas

Para declarar dependências entre tarefas, você usa os operadores >> e << (recomendado) ou métodos como _set_upstream_ e _set_downstream_. Também é possível encadear tarefas, criar dependências cross-downstream e até construir listas de tarefas dinamicamente. Um DAG sem dependências seria apenas um conjunto independente de tarefas.

### Melhores Práticas para Design de DAG

|Melhor Prática|Descrição|Dicas Práticas / Exemplos|
|---|---|---|
|**Garantir que DAGs são verdadeiramente acíclicos**|DAGs não devem conter ciclos. Em fluxos de trabalho complexos, é fácil introduzir loops indiretos (ex: A → B → C → A). O Airflow detecta ciclos automaticamente e se recusa a executar tais DAGs.|Revisar dependências regularmente. Usar visualização Graph para confirmar visualmente que o DAG não tem loops. Refatorar DAGs complexos para evitar ciclos acidentais.|
|**Manter tarefas idempotentes e pequenas**|Cada tarefa deve realizar uma única etapa lógica e ser segura para reexecutar sem corromper dados. Tarefas idempotentes garantem que retries não produzam resultados inconsistentes.|Usar padrões "insert or replace". Escrever em arquivos temporários antes de confirmar resultados. Dividir scripts grandes em múltiplas tarefas.|
|**Usar IDs descritivos e documentação**|Nomenclatura clara e documentação melhoram legibilidade e manutenibilidade tanto no código quanto na UI do Airflow.|Usar IDs significativos para DAG e tarefas. Adicionar documentação usando `dag.doc_md` ou `task.doc_md`. Alinhar nomes com lógica de negócio.|
|**Aproveitar recursos do Airflow**|Usar capacidades integradas do Airflow para comunicação, tratamento de credenciais e coordenação de pipeline ao invés de lógica personalizada.|Usar XComs para compartilhamento de pequenos dados. Usar Variables e Connections para config e secrets. Usar operadores e hooks integrados sempre que possível.|
|**Testar e versionar o código do DAG**|DAGs são código e devem ser mantidos usando práticas adequadas de engenharia de software: controle de versão, testes e CI/CD.|Armazenar DAGs no Git. Escrever testes para operadores personalizados e lógica. Usar ambientes local/teste antes de implantar em produção.|
|**Evitar sobrecarregar o agendador**|Números enormes de DAGs ou computações pesadas dentro de arquivos DAG podem desacelerar o agendador. DAGs devem ser leves para analisar.|Monitorar performance do agendador. Dividir DAGs massivos em menores. Evitar chamadas API ou computação pesada durante importação do DAG.|
|**Definir estratégia clara de tratamento de falhas**|Planejar como falhas devem ser tratadas. Nem todos os erros devem ter retry, e tarefas de longa execução podem precisar de SLAs ou alertas.|Usar retries apenas para falhas transitórias. Usar Sensores e triggers para dependências baseadas em eventos. Definir SLAs para tarefas longas ou críticas.|

---

## Exemplo: Um DAG Simples do Airflow

Nada solidifica conceitos melhor do que um exemplo. Vamos examinar um código de DAG simples do Airflow e explicar suas partes. Este exemplo criará um pequeno fluxo de trabalho com duas tarefas em sequência:

```python
from datetime import datetime
from airflow import DAG
from airflow.operators.bash import BashOperator

# Define o DAG e seu cronograma
with DAG(
    dag_id="example_dag",
    description="Um DAG de exemplo simples",
    start_date=datetime(2025, 1, 1),
    schedule_interval="@daily",  # executa diariamente
    catchup=False
) as dag:
    # Tarefa 1: Imprimir data atual
    task1 = BashOperator(
        task_id="print_date",
        bash_command="date"
    )
    # Tarefa 2: Ecoar uma mensagem
    task2 = BashOperator(
        task_id="echo_hello",
        bash_command="echo 'Olá, Airflow!'"
    )
    # Definir dependência: task1 deve executar antes de task2
    task1 >> task2
```

Vamos decompor o que está acontecendo aqui:

- Importamos as classes necessárias _DAG_ (para definir o DAG) e _BashOperator_ (para executar comandos shell/bash em tarefas). Também importamos _datetime_ para especificar a data de início do DAG.
    
- Criamos um objeto DAG com um gerenciador de contexto (_with DAG(…) as dag_:). Fornecemos ao DAG um ID único "example_dag" (deve ser único em um ambiente Airflow), uma descrição (opcional, para ajudar humanos a lerem), uma **start_date** e um **schedule_interval**. Aqui, definimos um preset integrado de _schedule_interval (@daily)_ e _start_date=datetime(2025,1,1)_. Também definimos _catchup=False_ para prevenir backfilling de quaisquer execuções perdidas entre start_date e hoje (queremos apenas que execute a partir de agora diariamente). Este bloco inteiro cria um DAG do qual o Airflow estará ciente para executar diariamente no início do dia.
    
- Então definimos duas tarefas no contexto do DAG:
    
    - _task1_ é definida usando BashOperator para executar um comando bash (_date_). _task_id é "print_date"_ e será usado para identificar esta tarefa na UI e nos logs. Quando executar, esta tarefa simplesmente imprimirá a data/hora atual (no log da tarefa).
    - _task2_ também usa um BashOperator com _task_id="echo_hello"_ e command=_echo 'Olá, Airflow!'_. Isso imprimirá uma mensagem agradável.
    - Essas tarefas agora estão apenas definidas, ainda não vinculadas.
- A última linha (_task1 >> task2_) especifica uma dependência (fluxo): task1 flui para task2. Isso significa que task1 executará e completará com sucesso antes que task2 possa ser considerada para agendamento. Graficamente, no grafo do DAG, haverá uma seta apontando de task1 para task2.
    

Algumas coisas a notar neste exemplo:

- Usamos apenas BashOperator para simplicidade, mas poderíamos ter usado um DAG com tipos mistos de operadores. Poderíamos ter um PythonOperator para chamar uma função Python, ou um SimpleHttpOperator para pingar uma API, e assim por diante.
    
- O padrão with DAG(…) as dag: é um idioma comum para criar uma instância DAG. Todas as tarefas que são criadas dentro dele são automaticamente associadas àquele DAG.
    
- catchup=False é um parâmetro usado em muitos DAGs com start_date no passado, quando você não quer que execute para todas as datas passadas. Se deixássemos catchup=True (o padrão) e hoje fosse 10 de janeiro de 2025, o Airflow tentaria executar o DAG para 1, 2, 3 de janeiro... até 10 de janeiro, recuperando esses dias perdidos em um backlog. Desligamos aqui para simplicidade.
    
- Este exemplo usa o caminho de importação relativo para BashOperator. No Airflow 2, muitos dos operadores foram movidos para _airflow.providers_. Este exemplo é escrito sob a suposição de que esses providers estão instalados (BashOperator está no próprio Airflow principal).
    

Este exemplo simples pode ser facilmente expandido – você pode adicionar tarefas adicionais (talvez fazer uma chamada API, então carregar resultados em um banco de dados, então enviar uma notificação). Você também pode adicionar lógica mais complexa (como ramificação – Airflow suporta estilo if/else usando o BranchOperator, por exemplo).

À medida que seus DAGs crescem, a UI e logging do Airflow se tornam inestimáveis para descobrir o que está acontecendo. Neste exemplo, o log para _print_date_ conteria a data do sistema que foi impressa, e o log para _echo_hello_ conteria "Olá, Airflow!". Se _print_date_ falhasse (talvez não conseguisse encontrar o comando date, em um cenário hipotético), então _echo_hello_ nunca seria executado (devido à dependência), e a execução do DAG seria marcada como falhada. Você poderia então inspecionar o log, corrigir o problema e reexecutar aquela tarefa ou todo o DAG.

---

## Casos de Uso Comuns para o Airflow

|Caso de Uso|Descrição|Como o Airflow Ajuda / Benefícios|
|---|---|---|
|**Pipelines ETL/ELT de Dados**|Mover dados de múltiplas fontes para um data warehouse ou data lake usando padrões ETL (Extract–Transform–Load) ou ELT.|DAGs orquestram extração → transformação → carga na ordem correta. Airflow integra com muitos bancos de dados e sistemas de armazenamento. Se uma etapa falha (ex: API fora do ar), tarefas downstream são paradas e alertas podem ser acionados, melhorando confiabilidade para sincronizações em lote e incrementais.|
|**Data Warehousing e Relatórios de BI**|Preparar e atualizar dados de análise usados em dashboards e relatórios.|Agenda jobs diários ou periódicos para que relatórios sempre executem contra dados atualizados. Coordena cargas de trabalho SQL, verificações de qualidade e etapas de relatórios. Fornece monitoramento e notificações para que falhas (ex: agregações quebradas) sejam visíveis ao invés de silenciosamente corromper saídas de BI.|
|**Pipelines de Aprendizado de Máquina**|Automatizar fluxos de trabalho ML de ponta a ponta desde preparação de dados até implantação.|Representa cada estágio ML como uma tarefa em um DAG e reforça a ordenação correta. Pode persistir artefatos entre etapas (dados pré-processados, binários de modelo). Integra com frameworks ML e operadores Kubernetes e permite retreinamento agendado e orquestração de experimentos durante horários de baixo uso.|
|**Verificações de Qualidade de Dados e Validação**|Executar verificações automatizadas para garantir que dados sejam completos, consistentes e confiáveis.|Agenda DAGs de qualidade de dados recorrentes (diário/semanal). Coordena verificações através de bancos de dados e scripts de QA, alerta equipes sobre anomalias e pode acionar tarefas de correção downstream como parte de engenharia de confiabilidade de dados.|
|**Manutenção de DB Transacional e Backups**|Automatizar tarefas operacionais de rotina para bancos de dados de produção e infraestrutura.|Centraliza agendamento e monitoramento de tarefas de manutenção. Garante que backups e manutenção executem consistentemente em horários definidos, reduz esforço manual e erro humano, e fornece logs/histórico para que equipes possam confirmar que jobs críticos de manutenção foram concluídos com sucesso.|
|**Integração / Automação de Fluxo de Trabalho**|Orquestrar fluxos de trabalho de negócio ou sistema que abrangem múltiplos serviços e APIs.|Atua como uma camada flexível de "cola" que conecta serviços. DAGs codificam ramificação complexa e lógica condicional. Operadores e tarefas Python permitem integrações personalizadas. Fornece um lugar central para gerenciar, monitorar e retentar etapas de processo de negócio ao invés de espalhar lógica de automação através de scripts e ferramentas ad-hoc.|

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

|Ferramenta / Serviço|Resumo|Referência|
|---|---|---|
|**Luigi**|Agendador de fluxo de trabalho Python de código aberto criado pelo Spotify. Bom para construir pipelines em lote com tarefas e dependências em código. Arquitetura mais simples e mais leve que o Airflow, mas com um ecossistema menor e menos integrações integradas.|[Documentação Luigi](https://luigi.readthedocs.io/)|
|**Prefect**|Framework de orquestração nativo Python posicionado como uma alternativa mais moderna e amigável ao desenvolvedor que o Airflow. Usa Flows e Tasks, oferece agendamento, retries e observabilidade. Pode executar totalmente código aberto ou com Prefect Cloud para UI hospedada e plano de controle.|[Site Prefect](https://www.prefect.io/)|
|**|||