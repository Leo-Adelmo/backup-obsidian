### Shuffles
Certas operações, como `join()` e `groupByKey()`, exigem que o Spark execute um shuffle. O shuffle é o mecanismo do Spark para redistribuir dados de modo que sejam agrupados de maneira diferente entre as partições.
Embora necessário, o shuffle é uma operação custosa porque:

- **I/O de disco:** Gera muitos arquivos intermediários
- **I/O de rede:** Transfere grandes volumes de dados entre executores
- **CPU e memória:** Exige ordenação e mesclagem de dados, sobrecarregando os executores

O shuffle é um dos fatores mais substanciais na degradação do desempenho de sua aplicação Spark. Ao armazenar os dados intermediários, ele pode esgotar o espaço no disco local do executor, o que causa a falha do trabalho Spark.
## Joins Strategys
### Broadcast Hash Join
![[Pasted image 20251121205247.png]]
O **Broadcast Hash Join** replica o dataset menor para todos os executores, permitindo que o join ocorra localmente sem shuffle da tabela maior. É geralmente a estratégia mais rápida quando uma das tabelas cabe confortavelmente na memória. Controlado por `spark.sql.autoBroadcastJoinThreshold`.  

**Exemplo:**

```python
from pyspark.sql import SparkSession
from pyspark.sql.functions import broadcast

spark = SparkSession.builder.getOrCreate()

# Forçar broadcast join
df_result = df_grande.join(
    broadcast(df_pequeno),
    df_grande.id == df_pequeno.id
)
```

**Vantagens:**

- Muito rápido, sem shuffle de dados
- Baixo uso de rede

**Desvantagens:**

- Limitado pelo tamanho da memória
- Não funciona com joins FULL OUTER

### Shuffle Hash Join
![[Pasted image 20251121231313.png]]
O **Shuffle Hash Join** embaralha ambos os lados usando a chave de join e constrói tabelas hash em cada partição. É útil quando as partições resultantes são pequenas e são adequadas para construção de hash tables.  

A junção Shuffle Hash, como o nome indica, funciona embaralhando ambos os conjuntos de dados. As mesmas chaves de ambos os lados terminam na mesma partição ou tarefa. Após os dados serem embaralhados, o menor dos dois conjuntos de dados passa por hash em buckets e, então, uma junção hash é realizada dentro da partição. Uma junção Shuffle Hash é diferente de uma junção Broadcast Hash porque o conjunto de dados inteiro não é transmitido. Uma junção Shuffle Hash é dividida em duas fases:

- **Fase de Shuffle** – Ambos os conjuntos de dados são embaralhados.
- **Fase de Hash join** – O lado menor dos dados passa por hash, é colocado em buckets e então é unido por hash com o lado maior em todas as partições.

A ordenação não é necessária com junções Shuffle Hash dentro das partições. A imagem a seguir mostra as fases da junção Shuffle Hash. Os dados são lidos inicialmente, depois são embaralhados e, em seguida, um hash é criado e usado para a junção.

![[Pasted image 20251121225552.png]]

Por padrão, o otimizador escolhe a junção Shuffle Hash quando a junção Broadcast Hash não pode ser usada. Com base no tamanho do limite da junção Broadcast Hash (`spark.sql.autoBroadcastJoinThreshold`) e no número de partições de shuffle (`spark.sql.shuffle.partitions`) escolhido, ele usa a junção Shuffle Hash quando a partição única do SQL lógico é pequena o suficiente para construir uma tabela hash local.

### Sort-Merge Join

![[Pasted image 20251121230018.png]]

No **Sort-Merge Join**, ambos os datasets são embaralhados e ordenados por chave antes da mesclagem. É a estratégia mais robusta para grandes volumes de dados e é utilizada quando nem broadcast nem hash join são adequados.  

A junção Shuffle Sort Merge é preferida quando ambos os conjuntos de dados são grandes e não podem caber na memória. Como o nome indica, esta junção envolve as seguintes três fases:

- **Fase de Shuffle** – Ambos os conjuntos de dados na consulta de junção são embaralhados (shuffled).
- **Fase de Sort** – Os registros são ordenados pela chave de junção em ambos os lados.
- **Fase de Merge** – Ambos os lados da condição de junção são iterados, com base na chave de junção.

### Broadcast Nested Loop Join

O **Broadcast Nested Loop Join** é aplicado quando não existe condição de igualdade ou quando o planner não pode utilizar outras estratégias. É custoso, porém necessário em alguns cenários específicos.  

A junção Shuffle-and-Replicate Nested Loop, também conhecida como junção Produto Cartesiano, funciona de forma muito similar a uma junção Broadcast Hash, exceto que o conjunto de dados não é transmitido.

Neste algoritmo de junção, shuffle não se refere a um verdadeiro embaralhamento porque registros com as mesmas chaves não são enviados para a mesma partição. Em vez disso, a partição inteira de ambos os conjuntos de dados é copiada pela rede. Quando as partições dos conjuntos de dados estão disponíveis, uma junção Nested Loop é realizada. Se há `X` número de registros no primeiro conjunto de dados e `Y` número de registros no segundo conjunto de dados em cada partição, cada registro no segundo conjunto de dados é unido com cada registro no primeiro conjunto de dados. Isso continua em um loop `X × Y` vezes em cada partição.

## Hints de Join

O PySpark permite orientar o otimizador usando hints:

- **BROADCAST** — força broadcast.
    
- **MERGE** — favorece sort-merge join.
    
- **SHUFFLE_HASH** — favorece shuffle hash join.
    
- **SHUFFLE_REPLICATE_NL** — força nested loop replicado.


Hints são úteis, mas devem ser aplicados apenas quando há certeza sobre o comportamento dos dados.  

## Otimizações de Joins

### Técnica de Otimização (Tuning)

A operação `join()`, que une tabelas, é a operação de shuffle mais comumente usada, mas frequentemente é um gargalo de desempenho. Como a junção é uma operação custosa, recomendamos não usá-la a menos que seja essencial para seus requisitos de negócio. Verifique novamente se você está fazendo uso eficiente do seu pipeline de dados fazendo as seguintes perguntas:

- Você está recomputando uma junção que também é realizada em outros trabalhos que você pode reutilizar?
- Você está fazendo junções para resolver chaves estrangeiras de valores que não são usados pelos consumidores de sua saída?

Depois de confirmar que suas operações de junção são essenciais para seus requisitos de negócio, veja as seguintes opções para otimizar sua junção de forma a atender seus requisitos.

### Use pushdown antes da junção

Filtre linhas e colunas desnecessárias no DataFrame antes de realizar uma junção. Isso tem as seguintes vantagens:

- Reduz a quantidade de transferência de dados durante o shuffle
- Reduz a quantidade de processamento no executor Spark
- Reduz a quantidade de varredura de dados

```python
# Padrão
df_joined = df1.join(df2, ["product_id"])

# Use Pushdown
df1_select = df1.select("product_id","product_title","star_rating").filter(col("star_rating")>=4.0)
df2_select = df2.select("product_id","category_id")
df_joined = df1_select.join(df2_select, ["product_id"])
```

### Shuffle e broadcast hash joins e hints

O Spark suporta dois categorias principais de tipos de junção: shuffle join e broadcast hash join.

Uma broadcast hash join não requer embaralhamento e pode exigir menos processamento do que uma shuffle join. No entanto, é aplicável apenas ao unir uma tabela pequena a uma grande. Ao unir uma tabela que pode caber na memória de um único executor Spark, considere usar uma broadcast hash join.

O diagrama a seguir mostra a estrutura de alto nível e as etapas de uma broadcast hash join e uma shuffle join.

![[Pasted image 20251121221803.png]]

### Use bucketing

A sort-merge join requer duas fases, shuffle e sort, e depois merge. Essas duas fases podem sobrecarregar o executor Spark e causar problemas de OOM e desempenho quando alguns dos executores estão mesclando e outros estão ordenando simultaneamente. Em tais casos, pode ser possível unir de forma eficiente usando bucketing. O bucketing vai pré-embaralhar e pré-ordenar sua entrada nas chaves de junção e, em seguida, escrever esses dados ordenados em uma tabela intermediária. O custo das etapas de shuffle e sort pode ser reduzido ao unir tabelas grandes definindo as tabelas intermediárias ordenadas com antecedência.

![[Pasted image 20251121232335.png]]

Tabelas com buckets são úteis para o seguinte:

- Dados unidos frequentemente sobre a mesma chave, como `account_id`
- Carregamento de tabelas cumulativas diárias, como tabelas base e delta que poderiam ter buckets em uma coluna comum

Você pode criar uma tabela com buckets usando o seguinte código:

```python
df.write.bucketBy(50, "account_id").sortBy("age").saveAsTable("bucketed_table")
```

### Reparticionar DataFrames nas chaves de junção antes da junção

Para reparticionar os dois DataFrames nas chaves de junção antes da junção, use as seguintes instruções:

```python
df1_repartitioned = df1.repartition(N,"join_key")
df2_repartitioned = df2.repartition(N,"join_key")
df_joined = df1_repartitioned.join(df2_repartitioned,"product_id")
```

Isso irá particionar dois RDDs (ainda separados) na chave de junção antes de iniciar a junção. Se os dois RDDs forem particionados na mesma chave com o mesmo código de particionamento, os registros RDD que você planeja unir têm uma alta probabilidade de estarem co-localizados no mesmo worker antes do embaralhamento para a junção. Isso pode melhorar o desempenho reduzindo a atividade de rede e a distorção de dados durante a junção.

## Superar a distorção de dados (data skew)

A distorção de dados é uma das causas mais comuns de gargalo para trabalhos Spark. Ocorre quando os dados não são distribuídos uniformemente entre as partições de RDD. Isso faz com que as tarefas para essa partição levem muito mais tempo do que outras, atrasando o tempo geral de processamento da aplicação.

Para identificar a distorção de dados, avalie as seguintes métricas na UI do Spark:

- Na aba Stage na UI do Spark, examine a página Event Timeline. Você pode ver uma distribuição desigual de tarefas na captura de tela a seguir. Tarefas que são distribuídas de forma desigual ou que estão demorando muito para executar podem indicar distorção de dados.
- ![[Pasted image 20251121232652.png]]
    
- Outra página importante é Summary Metrics, que mostra estatísticas para tarefas Spark. A captura de tela a seguir mostra métricas com percentis para Duration, GC Time, Spill (memory), Spill (disk) e assim por diante.
- ![[Pasted image 20251121232844.png]]
- Quando as tarefas são distribuídas uniformemente, você verá números semelhantes em todos os percentis. Quando há distorção de dados, você verá valores muito tendenciosos em cada percentil.
  
  - Use chaves com uma grande faixa de valores para as chaves de junção. Em uma shuffle join, as partições são determinadas para cada valor de hash de uma chave. Se a cardinalidade de uma chave de junção for muito baixa, a função hash tem mais probabilidade de fazer um trabalho ruim de distribuir seus dados entre as partições. Portanto, se sua aplicação e lógica de negócio permitirem, considere usar uma chave de cardinalidade mais alta ou uma chave composta.

```python
# Use Chave Primária Única
df_joined = df1_select.join(df2_select, ["primary_key"])

# Use Chave Composta
df_joined = df1_select.join(df2_select, ["primary_key","secondary_key"])
```

## Use cache

Quando você usa DataFrames repetitivos, evite shuffle adicional ou computação usando `df.cache()` ou `df.persist()` para armazenar em cache os resultados do cálculo na memória de cada executor Spark e no disco. O Spark também suporta persistir RDDs no disco ou replicar em vários nós (storage level).

Por exemplo, você pode persistir os DataFrames adicionando `df.persist()`. Quando o cache não for mais necessário, você pode usar `unpersist` para descartar os dados em cache.

```python
df = spark.read.parquet("s3://<Bucket>/parquet/product_category=Books/")
df_high_rate = df.filter(col("star_rating")>=4.0)
df_high_rate.persist()

df_joined1 = df_high_rate.join(<Table1>, ["key"])
df_joined2 = df_high_rate.join(<Table2>, ["key"])
df_joined3 = df_high_rate.join(<Table3>, ["key"])
...

df_high_rate.unpersist()
```

## Remover ações Spark desnecessárias

Evite executar ações desnecessárias como `count`, `show` ou `collect`. Cada RDD transformado pode ser recomputado cada vez que você executa uma ação nele. Quando você usa muitas ações Spark, vários acessos à fonte, cálculos de tarefas e execuções de shuffle para cada ação estão sendo chamados.

Se você não precisa de `collect()` ou outras ações no seu ambiente comercial, considere removê-las.

## Referências
[Apache Spark – Performance Tuning](https://spark.apache.org/docs/latest/sql-performance-tuning.html#broadcast-hash-joins) 
[Join Strategies in Apache Spark — A hands-on approach!](https://medium.com/%40amarkrgupta96/join-strategies-in-apache-spark-a-hands-on-approach-d0696fc0a6c9) 
[Optimize shuffles - AWS](https://docs.aws.amazon.com/prescriptive-guidance/latest/tuning-aws-glue-for-apache-spark/optimize-shuffles.html)
[Using join hints in Spark SQL - AWS](https://docs.aws.amazon.com/prescriptive-guidance/latest/spark-tuning-glue-emr/using-join-hints-in-spark-sql.html)
