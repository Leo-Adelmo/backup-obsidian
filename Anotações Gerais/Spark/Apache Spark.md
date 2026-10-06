# 1 - Apache Spark

O **Apache Spark** é um mecanismo de análise unificado para processamento de dados em larga escala, oferecendo APIs em **Java, Scala, Python e R**. Ele suporta diversas ferramentas avançadas, como **Spark SQL**, **Pandas API on Spark**, **MLlib**, **GraphX** e **Structured Streaming**.

O **Apache Spark** foi criado no **AMPlab (Algorithms, Machines, and People Lab)** da Universidade da Califórnia, Berkeley, por volta de **2009**. Na época, o grande volume de dados gerado por aplicações na internet e outras áreas tornou-se um desafio para os sistemas de processamento. O **Hadoop MapReduce**, que era o padrão da época, tinha limitações significativas em termos de eficiência e velocidade, especialmente para workloads iterativos e interativos.

O Spark surgiu como uma alternativa para melhorar o processamento distribuído de grandes volumes de dados, permitindo que as aplicações fossem mais rápidas e eficientes. Em **2013**, o projeto tornou-se open-source e foi doado à **Apache Software Foundation**, tornando-se um dos frameworks mais populares para Big Data.

## 1.1 Problemas que o Spark Busca Resolver

1. **Baixo desempenho do Hadoop MapReduce**
    
    - O MapReduce escreve e lê dados do disco constantemente, tornando o processamento lento. O Spark usa a **memória RAM** para evitar esse gargalo.
        
2. **Dificuldade em processar dados em tempo real**
    
    - O Hadoop tradicionalmente foca no processamento em lote (batch), enquanto o Spark suporta **processamento em tempo real** (streaming).
        
3. **Falta de suporte para Machine Learning e Analytics Avançado**
    
    - O Spark inclui bibliotecas como **MLlib** para aprendizado de máquina e **GraphX** para análise de grafos.
        
4. **Dificuldade em lidar com múltiplos tipos de dados**
    
    - O Spark suporta **estruturas de dados complexas** e é compatível com diversas fontes como bancos de dados SQL, NoSQL, HDFS e sistemas de mensagens como Kafka.

## 2. RDD (_Resilient Distributed Dataset_)

O Spark gira em torno do conceito de um **RDD** (_Resilient Distributed Dataset_), uma coleção tolerante a falhas de elementos operados em paralelo.

Um **RDD (Resilient Distributed Dataset)** é uma estrutura de dados central no Apache Spark, formando a espinha dorsal desde a sua criação. Ele representa uma coleção imutável e tolerante a falhas de elementos que podem ser processados em paralelo em um cluster de máquinas. Os RDDs servem como os blocos fundamentais de construção no Spark, sobre os quais estruturas de dados mais novas, como datasets e data frames, são construídas.

Como a abstração central no Spark, os RDDs abrangem uma ampla gama de operações, incluindo transformações (como map, filter e reduce) e ações (como count e collect). Essas operações permitem que os usuários realizem manipulações complexas de dados e cálculos em RDDs. Os RDDs oferecem tolerância a falhas, mantendo o rastreamento das informações de linhagem necessárias para reconstruir partições perdidas.

### 2.1 Características do RDD no Spark

Um RDD tem as seguintes características:

- RDDs consistem em dados divididos em várias partes chamadas _partições._ Cada executor do Spark armazena uma ou mais partições na memória e os dados são distribuídos entre vários executores.
    
- RDDs são _imutáveis_, o que significa que não podem ser alterados após serem criados. Para alterar a DataFrame, você pode usar _transformações_, que são definidas na seção a seguir.
    
- RDDs replicam dados nos nós disponíveis, para que eles possam se recuperar automaticamente das falhas nos nós.
  
  ![[Pasted image 20250326203707.png]]

### 2.1 Avaliação preguiçosa (lazy evaluation)

RDDs oferecem suporte a dois tipos de operações: _transformações_, que criam um novo conjunto de dados a partir de um existente, e _ações_, que retornam um valor para o programa do driver após executar um cálculo no conjunto de dados.

- **Transformações** — Como RDDs são _imutáveis,_ você só pode alterá-las usando uma transformação.
    
    Por exemplo, `map` é uma transformação que passa cada elemento do conjunto de dados por meio de uma função e retorna um novo RDD representando os resultados. Observe que o `map` método não retorna uma saída. O Spark armazena a transformação abstrata para o futuro, em vez de permitir que você interaja com o resultado. O Spark não atuará nas transformações até que você chame uma ação.
    
- **Ações** — Usando transformações, você constrói seu plano lógico de transformação. Para iniciar o cálculo, você executa uma ação como`write`, `count`, `show`, ou. `collect`
    
    Todas as transformações no Spark são _lentas_, pois não computam seus resultados imediatamente. Em vez disso, o Spark lembra uma série de transformações aplicadas a alguns conjuntos de dados básicos. As transformações são computadas somente quando uma ação exige que um resultado seja retornado ao driver. Esse design permite que o Spark funcione com mais eficiência. 


![[Pasted image 20250326204133.png]]

### 2.2 Terminologia dos aplicativos do Spark

Esta seção aborda a terminologia do aplicativo Spark. O driver do Spark cria um _plano de execução_ e controla o comportamento dos aplicativos em várias abstrações. Os termos a seguir são importantes para desenvolvimento, depuração e ajuste de desempenho com a interface do usuário do Spark.

- **_Application (Aplicativo) _** — Baseado em uma sessão do Spark (contexto do Spark). Identificado por um ID exclusivo, como`<application_XXX>`.
    
- **_Job (Trabalhos_)** — Com base nas ações criadas para um RDD. Um trabalho consiste em um ou mais _estágios_.
    
- **_Stage (Estágios)_** — Com base nos _shuffles_ criados para um RDD. Um estágio consiste em uma ou mais tarefas. O shuffle é o mecanismo do Spark para redistribuir dados para que sejam agrupados de forma diferente nas partições RDD. Certas transformações, como`join()`, exigem um embaralhamento.
    
- **_Task (Tarefas)_** — Uma tarefa é a unidade mínima de processamento agendada pelo Spark. As tarefas são criadas para cada partição RDD, e o número de tarefas é o número máximo de execuções simultâneas no Stage.
  
  ![[Pasted image 20250326204331.png]]
### 2.3 Formas de Criar RDDs:
- Paralelizando uma coleção existente no programa driver.
- Referenciando um conjunto de dados externo (HDFS, HBase, etc.).

### 2.4 Operações em RDD:

- **Transformações** (retornam novos RDDs):
  - `map()`, `filter()`, `flatMap()`, `reduceByKey()`.
- **Ações** (executam imediatamente e retornam valores):
  - `collect()`, `count()`, `first()`.

## 3. Dataset e DataFrame

Um **Dataset** é uma coleção distribuída de dados. O Dataset é uma nova interface adicionada no Spark 1.6 que oferece os benefícios dos RDDs (tipagem forte, capacidade de usar funções lambda poderosas) com os benefícios do motor de execução otimizado do Spark SQL. Um Dataset pode ser construído a partir de objetos JVM e, em seguida, manipulado usando transformações funcionais (map, flatMap, filter, etc.). A API do Dataset está disponível em Scala e Java. O Python não tem suporte para a API do Dataset, mas devido à natureza dinâmica do Python, muitos dos benefícios da API do Dataset já estão disponíveis (ou seja, você pode acessar o campo de uma linha por nome naturalmente, `linha.nomeColuna`). O caso para R é similar.

Um **DataFrame** é um Dataset organizado em colunas nomeadas. Ele é conceitualmente equivalente a uma tabela em um banco de dados relacional ou a um data frame em R/Python, mas com otimizações mais sofisticadas internamente. Os DataFrames podem ser construídos a partir de uma ampla variedade de fontes, como: arquivos de dados estruturados, tabelas no Hive, bancos de dados externos ou RDDs existentes. A API do DataFrame está disponível em Scala, Java, Python e R. Em Scala e Java, um DataFrame é representado por um Dataset de Rows. Na API do Scala, o DataFrame é simplesmente um alias de tipo de `Dataset[Row]`. Já na API do Java, os usuários precisam usar `Dataset<Row>` para representar um DataFrame.
### Operações Comuns de DataFrame:
- **Seleção de Colunas**: `df.select("coluna1", "coluna2")`
- **Filtragem**: `df.filter(df["coluna1"] > 100)`
- **Agregação**: `df.groupBy("coluna1").agg({"coluna2": "sum"})`
- **Leitura e Escrita**:
  - `df.write.csv("output_path")` (escrita)
  - `spark.read.csv("input_path")` (leitura)
## 4. Arquitetura e Componentes

Os aplicativos Spark são executados como conjuntos independentes de processos em um cluster, coordenados pelo `SparkContext` no programa principal (chamado de _driver program_).

Para executar em um cluster, o `SparkContext` pode se conectar a vários tipos de _gerenciadores de cluster_ (Standalone, Mesos, YARN ou Kubernetes), que alocam recursos entre aplicativos. Uma vez conectado, o Spark adquire _executors_ em _nodes_ no cluster, que são processos que executam computações e armazenam dados para seu aplicativo. O `SparkContext` então envia _tasks_ para os executores rodarem.

![[Pasted image 20250320194924.png]]

### Características da Arquitetura

- Cada aplicação tem seus próprios processos de **executor**, que permanecem ativos durante toda a execução e executam tarefas em múltiplas threads.
- O **Spark é independente do gerenciador de cluster subjacente**, sendo possível rodá-lo em diversos gerenciadores (Mesos, YARN ou Kubernetes).
- O **programa driver** deve **escutar e aceitar conexões** dos seus **executores** durante toda a sua execução.

## 4.1 Tipos de Gerenciadores de Cluster

O Apache Spark suporta vários gerenciadores de cluster:

- **[Standalone](https://spark.apache.org/docs/latest/spark-standalone.html)** – Gerenciador de cluster simples incluído no Spark.
- **[Apache Mesos](https://spark.apache.org/docs/latest/running-on-mesos.html)** – Gerenciador de cluster geral para diversas aplicações (**Descontinuado**).
- **[Hadoop YARN](https://spark.apache.org/docs/latest/running-on-yarn.html)** – Gerenciador de recursos do Hadoop 3.
- **[Kubernetes](https://spark.apache.org/docs/latest/running-on-kubernetes.html)** – Sistema de automação para aplicações em contêineres.

## 5. Configurações Spark
### **5.1 SparkContext**

O `SparkContext` é um objeto essencial em uma aplicação Spark. Ele permite que você inicie a aplicação Spark, configure a comunicação com o cluster e coordene a execução de tarefas distribuídas.

Quando você cria um objeto `SparkContext`, você configura o ambiente Spark (como o número de nós, o tipo de cluster, a memória disponível, etc.) e prepara o Spark para executar tarefas. O `SparkContext` também serve como ponto de entrada para acessar RDDs (Resilient Distributed Datasets) e outros componentes do Spark.

### **5.2 SparkConf**

O **`SparkConf`** é um objeto que contém um conjunto de propriedades para configurar o ambiente de execução do Spark. Ele permite que você ajuste as configurações de execução antes de criar um `SparkContext`. O `SparkConf` pode ser configurado de várias formas: diretamente no código, através de arquivos de configuração, ou por meio de variáveis de ambiente.

Quando você cria um `SparkContext`, ele recebe um objeto `SparkConf` como argumento, o qual define a configuração da aplicação Spark.

### **5.3 Tipos de configurações do Spark**

**O Spark oferece três locais para configurar o sistema:**

- **Propriedades do Spark** controlam a maioria dos parâmetros da aplicação e podem ser configuradas usando um objeto `SparkConf` ou através de propriedades do sistema Java.
  
- **Variáveis de ambiente** podem ser usadas para configurar ajustes específicos de cada máquina, como o endereço IP, por meio do script `conf/spark-env.sh` em cada nó.
  
- **Logging** pode ser configurado através do arquivo `log4j2.properties`.

### **5.4 Propriedades do Spark**

As propriedades do Spark controlam a maioria das configurações da aplicação e são configuradas separadamente para cada aplicação. Essas propriedades podem ser definidas diretamente em um `SparkConf` passado para o seu `SparkContext`. O `SparkConf` permite configurar algumas propriedades comuns (por exemplo, URL do mestre e nome da aplicação), assim como pares de chave-valor arbitrários através do método `set()`. Por exemplo, poderíamos inicializar uma aplicação com dois threads da seguinte forma:

```python
from pyspark import SparkConf, SparkContext

# Configuração do Spark
conf = SparkConf().setMaster("local[2]").setAppName("<app_name>")

# Criação do SparkContext
sc = SparkContext(conf=conf)
```

**Carregando Propriedades do Spark Dinamicamente**

Em alguns casos, você pode querer evitar a codificação rígida de certas configurações em um `SparkConf`. Por exemplo, se você deseja executar a mesma aplicação com diferentes mestres ou quantidades diferentes de memória. O Spark permite que você crie simplesmente uma `SparkConf` vazia:

```python
from pyspark import SparkConf, SparkContext

# Criação de uma SparkConf vazia
conf = SparkConf()

# Criação do SparkContext com a configuração
sc = SparkContext(conf=conf)
```

Em seguida, você pode fornecer valores de configuração em tempo de execução:

```bash
./bin/spark-submit --name "My app" --master local[4] --conf spark.eventLog.enabled=false \
  --conf "spark.executor.extraJavaOptions=-XX:+PrintGCDetails -XX:+PrintGCTimeStamps" myApp.jar
```

O **Spark shell** e a ferramenta **spark-submit** suportam duas maneiras de carregar configurações dinamicamente. A primeira é através de opções de linha de comando, como `--master`, conforme mostrado acima. O **spark-submit** pode aceitar qualquer propriedade do Spark usando a flag `--conf/-c`, mas usa flags especiais para propriedades que participam do lançamento da aplicação Spark. Executar `./bin/spark-submit --help` exibirá a lista completa dessas opções.

O **bin/spark-submit** também lerá opções de configuração do arquivo `conf/spark-defaults.conf`, no qual cada linha consiste em uma chave e um valor separados por espaços em branco:

![[Pasted image 20250320200430.png]]
![[Pasted image 20250320203210.png]]

Se você deseja carregar as configurações dinamicamente, pode usar o comando `spark-submit` acima diretamente na linha de comando, como indicado, e o `SparkContext` será configurado com os parâmetros fornecidos.

```bash
spark-submit --master spark://spark-master:7077 /opt/spark/examples/src/main/python/pi.py
```
## 6. Inicialização do Cluster Standalone 

Além de ser executado nos gerenciadores de cluster citados (Mesos ou YARN), o Spark também oferece um modo de implantação standalone simples. Você pode iniciar um cluster standalone manualmente, iniciando um master e os workers manualmente, ou usar os scripts de inicialização fornecidos.

#### 6.1 Instalando o Spark Standalone em um Cluster

Para instalar o modo Standalone do Spark, basta colocar uma versão compilada do Spark em cada nó do cluster. 
Para iniciar um servidor master standalone devemos executar:

```bash
./sbin/start-master.sh
```


Após a inicialização, o master imprimirá uma URL no formato `spark://HOST:PORT`, que pode ser usada para conectar os workers a ele ou ser passada como argumento “master” para o `SparkContext`. Você também pode encontrar essa URL na interface web do master, que, por padrão, está disponível em `http://localhost:8080`.

Da mesma forma, você pode iniciar um ou mais workers e conectá-los ao master com o comando:

```bash
./sbin/start-worker.sh <master-spark-URL>
```

Depois de iniciar um worker, acesse a interface web do master (`http://localhost:8080` por padrão). Você deverá ver o novo nó listado lá, juntamente com seu número de CPUs e memória (menos um gigabyte reservado para o sistema operacional).

![[Pasted image 20250320195832.png]]
#### 6.2 Scripts de Inicialização do Cluster

Para iniciar um cluster Spark standalone usando os scripts de inicialização, você deve criar um arquivo chamado `conf/workers` no diretório do Spark. Esse arquivo deve conter os nomes dos hosts de todas as máquinas onde você deseja iniciar os workers do Spark, um por linha. Se o arquivo `conf/workers` não existir, os scripts de inicialização usarão um único computador (localhost) por padrão, o que é útil para testes.

![[Pasted image 20250320201418.png]]

Depois de configurar esse arquivo, você pode iniciar ou parar seu cluster usando os seguintes scripts shell, que são baseados nos scripts de implantação do Hadoop e estão disponíveis em `SPARK_HOME/sbin`:

- `sbin/start-master.sh` – Inicia uma instância master na máquina onde o script for executado.
- `sbin/start-workers.sh` – Inicia uma instância worker em cada máquina especificada no arquivo `conf/workers`.
- `sbin/start-worker.sh` – Inicia uma instância worker na máquina onde o script for executado.
- `sbin/start-connect-server.sh` – Inicia um servidor Spark Connect na máquina onde o script for executado.
- `sbin/start-all.sh` – Inicia tanto o master quanto os workers conforme descrito acima.
- `sbin/stop-master.sh` – Para a instância master iniciada com `sbin/start-master.sh`.
- `sbin/stop-worker.sh` – Para todas as instâncias worker na máquina onde o script for executado.
- `sbin/stop-workers.sh` – Para todas as instâncias worker nas máquinas especificadas no arquivo `conf/workers`.
- `sbin/stop-connect-server.sh` – Para todas as instâncias do servidor Spark Connect na máquina onde o script for executado.
- `sbin/stop-all.sh` – Para tanto o master quanto os workers conforme descrito acima.

![[Pasted image 20250320201536.png]]

Opcionalmente, você pode configurar ainda mais o cluster definindo variáveis de ambiente no arquivo `conf/spark-env.sh`. Para isso, crie esse arquivo a partir do modelo `conf/spark-env.sh.template` e copie-o para todas as máquinas worker para que as configurações tenham efeito. As seguintes configurações estão disponíveis:

| **Variável de Ambiente**    | **Significado**                                                                                                                                                                                                                                                                         |
| --------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **SPARK_MASTER_HOST**       | Vincula o mestre a um nome de host ou endereço IP específico, por exemplo, um público.                                                                                                                                                                                                  |
| **SPARK_MASTER_PORT**       | Inicia o mestre em uma porta diferente (padrão: 7077).                                                                                                                                                                                                                                  |
| **SPARK_MASTER_WEBUI_PORT** | Porta para a interface web do mestre (padrão: 8080).                                                                                                                                                                                                                                    |
| **SPARK_MASTER_OPTS**       | Propriedades de configuração que se aplicam apenas ao mestre na forma "-Dx=y" (padrão: nenhum). Veja abaixo uma lista de opções possíveis.                                                                                                                                              |
| **SPARK_LOCAL_DIRS**        | Diretório para usar como espaço temporário no Spark, incluindo arquivos de saída de mapas e RDDs que são armazenados no disco. Isso deve estar em um disco rápido e local no seu sistema. Pode também ser uma lista separada por vírgulas de múltiplos diretórios em diferentes discos. |
| **SPARK_WORKER_CORES**      | Número total de núcleos que o Spark pode usar na máquina (padrão: todos os núcleos disponíveis).                                                                                                                                                                                        |
| **SPARK_WORKER_MEMORY**     | Quantidade total de memória que o Spark pode usar na máquina, por exemplo, 1000m, 2g (padrão: memória total menos 1 GiB); observe que a memória individual de cada aplicativo é configurada usando a propriedade spark.executor.memory.                                                 |
| **SPARK_WORKER_PORT**       | Inicia o trabalhador do Spark em uma porta específica (padrão: aleatória).                                                                                                                                                                                                              |
| **SPARK_WORKER_WEBUI_PORT** | Porta para a interface web do trabalhador (padrão: 8081).                                                                                                                                                                                                                               |
| **SPARK_WORKER_DIR**        | Diretório para executar os aplicativos, que incluirá tanto os logs quanto o espaço temporário (padrão: SPARK_HOME/work).                                                                                                                                                                |
| **SPARK_WORKER_OPTS**       | Propriedades de configuração que se aplicam apenas ao trabalhador na forma "-Dx=y" (padrão: nenhum). Veja abaixo uma lista de opções possíveis.                                                                                                                                         |
| **SPARK_DAEMON_MEMORY**     | Memória a ser alocada para os próprios daemons do mestre e trabalhador do Spark (padrão: 1g).                                                                                                                                                                                           |
| **SPARK_DAEMON_JAVA_OPTS**  | Opções JVM para os próprios daemons do mestre e trabalhador do Spark na forma "-Dx=y" (padrão: nenhum).                                                                                                                                                                                 |
| **SPARK_DAEMON_CLASSPATH**  | Caminho de classes para os próprios daemons do mestre e trabalhador do Spark (padrão: nenhum).                                                                                                                                                                                          |
| **SPARK_PUBLIC_DNS**        | O nome DNS público do mestre e trabalhadores do Spark (padrão: nenhum).                                                                                                                                                                                                                 |

## 7. Monitoramento

Cada **programa driver** possui uma interface web (**Web UI**), normalmente disponível na **porta 4040**, que exibe informações sobre tarefas em execução, executores e uso de armazenamento.

Acesse `http://<driver-node>:4040` no navegador para visualizar a interface. Consulte o **[guia de monitoramento](https://spark.apache.org/docs/latest/monitoring.html)** para mais opções.



---
