# Apache Spark

O **Apache Spark** é um mecanismo de análise unificado para processamento de dados em larga escala, oferecendo APIs em **Java, Scala, Python e R**. Ele suporta diversas ferramentas avançadas, como **Spark SQL**, **Pandas API on Spark**, **MLlib**, **GraphX** e **Structured Streaming**.

## 1. Componentes

Os aplicativos Spark são executados como conjuntos independentes de processos em um cluster, coordenados pelo `SparkContext` no programa principal (chamado de _driver program_).

Para executar em um cluster, o `SparkContext` pode se conectar a vários tipos de _gerenciadores de cluster_ (Standalone, Mesos, YARN ou Kubernetes), que alocam recursos entre aplicativos. Uma vez conectado, o Spark adquire _executors_ em nodes no cluster, que são processos que executam computações e armazenam dados para seu aplicativo. O `SparkContext` então envia _tasks_ para os executores rodarem.

### Características da Arquitetura

- Cada aplicação tem seus próprios processos de **executor**, que permanecem ativos durante toda a execução e executam tarefas em múltiplas threads.
- O **Spark é independente do gerenciador de cluster subjacente**, sendo possível rodá-lo em diversos gerenciadores (Mesos, YARN ou Kubernetes).
- O **programa driver** deve **escutar e aceitar conexões** dos seus **executores** durante toda a sua execução.
- O **driver** deve ser executado **próximo aos nós de trabalho**, de preferência na **mesma rede local**.

## 2. Tipos de Gerenciadores de Cluster

O Apache Spark suporta vários gerenciadores de cluster:

- **[Standalone](https://spark.apache.org/docs/latest/spark-standalone.html)** – Gerenciador de cluster simples incluído no Spark.
- **[Apache Mesos](https://spark.apache.org/docs/latest/running-on-mesos.html)** – Gerenciador de cluster geral para diversas aplicações (**Descontinuado**).
- **[Hadoop YARN](https://spark.apache.org/docs/latest/running-on-yarn.html)** – Gerenciador de recursos do Hadoop 3.
- **[Kubernetes](https://spark.apache.org/docs/latest/running-on-kubernetes.html)** – Sistema de automação para aplicações em contêineres.

## 3. Submissão de Aplicações

As aplicações podem ser submetidas utilizando o comando **`spark-submit`**.

### Exemplo de uso:

```bash
spark-submit --master spark://spark-master:7077 /opt/spark/examples/src/main/python/pi.py
```

## 4. Monitoramento

Cada **programa driver** possui uma interface web (**Web UI**), normalmente disponível na **porta 4040**, que exibe informações sobre tarefas em execução, executores e uso de armazenamento.

Acesse `http://<driver-node>:4040` no navegador para visualizar a interface. Consulte o **[guia de monitoramento](https://spark.apache.org/docs/latest/monitoring.html)** para mais opções.

## 5. RDD (_Resilient Distributed Dataset_)

O Spark gira em torno do conceito de um **RDD** (_Resilient Distributed Dataset_), uma coleção tolerante a falhas de elementos operados em paralelo.

### Formas de Criar RDDs:
- Paralelizando uma coleção existente no programa driver.
- Referenciando um conjunto de dados externo (HDFS, HBase, etc.).

### Operações em RDD:

- **Transformações** (retornam novos RDDs):
  - `map()`, `filter()`, `flatMap()`, `reduceByKey()`.
- **Ações** (executam imediatamente e retornam valores):
  - `collect()`, `count()`, `first()`.

## 6. SparkContext

O **SparkContext** é o ponto de entrada para qualquer aplicação Spark, gerenciando execução, comunicação com o cluster e distribuição de tarefas.

### Principais Funções:
- `sc.textFile(path)`: Cria um RDD a partir de um arquivo de texto.
- `sc.parallelize(data)`: Cria um RDD a partir de uma coleção em memória.
- `sc.stop()`: Finaliza o SparkContext.

## 7. SparkSession

O **SparkSession** foi introduzido no Spark 2.0, combinando funcionalidades do **SQLContext** e do **HiveContext**.

### Exemplo de criação em PySpark:
```python
from pyspark.sql import SparkSession
spark = SparkSession.builder.appName("MinhaAplicacao").getOrCreate()
```

## 8. DataFrame

O **DataFrame** é uma abstração distribuída de dados organizada em colunas nomeadas.

### Operações Comuns:
- **Seleção de Colunas**: `df.select("coluna1", "coluna2")`
- **Filtragem**: `df.filter(df["coluna1"] > 100)`
- **Agregação**: `df.groupBy("coluna1").agg({"coluna2": "sum"})`
- **Leitura e Escrita**:
  - `df.write.csv("output_path")` (escrita)
  - `spark.read.csv("input_path")` (leitura)

## 9. SQLContext

O **SQLContext** é uma API para consultas SQL sobre DataFrames. No Spark 2.0, foi substituído pelo **SparkSession**.

## 10. Dataset

O **Dataset** é uma coleção de dados fortemente tipada, semelhante ao DataFrame, mas com segurança de tipos mais rigorosa (usado principalmente em Scala e Java).

---

Este README fornece uma visão geral sobre os conceitos fundamentais do Apache Spark. Para mais informações detalhadas, consulte a [documentação oficial](https://spark.apache.org/docs/latest/).

