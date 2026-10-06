Tradução do seguinte artigo: https://docs.aws.amazon.com/prescriptive-guidance/latest/spark-tuning-glue-emr/using-join-hints-in-spark-sql.html

Com o Spark 3.0, você pode especificar o tipo de algoritmo de junção que deseja que o Spark use em tempo de execução. As dicas de estratégia de junção, `BROADCAST`, `MERGE`, `SHUFFLE_HASH` e `SHUFFLE_REPLICATE_NL`, instruem o Spark a usar a estratégia indicada em cada relação especificada ao juntá-las com outra relação. Esta seção discute as dicas de estratégia de junção em detalhes.

## BROADCAST

Em uma junção Broadcast Hash, um dos conjuntos de dados é significativamente menor que o outro. Como o conjunto de dados menor pode caber na memória, ele é transmitido (broadcasted) para todos os executores no cluster. Após os dados serem transmitidos, uma junção hash padrão é realizada. Uma junção Broadcast Hash acontece em duas etapas:

- **Broadcast** – O conjunto de dados menor é transmitido para todos os executores dentro do cluster.
- **Hash join** – O conjunto de dados menor passa por um hash em todos os executores e então é unido ao conjunto de dados maior.

Não há operação de `sort` ou `merge`. Ao juntar grandes tabelas fato com tabelas dimensão menores usadas para realizar uma junção de esquema estrela (star schema join), o Broadcast Hash é o algoritmo de junção mais rápido. O exemplo a seguir demonstra como uma junção Broadcast Hash funciona. O lado da junção com a dica é transmitido, independentemente do limite de tamanho especificado na propriedade `spark.sql.autoBroadcastJoinThreshold`. Se ambos os lados da junção tiverem dicas de broadcast, o de menor tamanho (com base nas estatísticas) é transmitido. O valor padrão para a propriedade `spark.sql.autoBroadcastJoinThreshold` é 10 MB. Isso configura o tamanho máximo, em bytes, para uma tabela que é transmitida para todos os nós worker ao realizar uma junção.

Os exemplos a seguir fornecem a consulta, o plano físico `EXPLAIN` e o tempo necessário para a consulta ser executada. A consulta requer menos tempo de processamento se você usar a dica `BROADCASTJOIN` para forçar uma junção broadcast, como mostrado no segundo exemplo do plano `EXPLAIN`.

**Consulta SQL:**

```sql
select table1.id,table1.col,table2.id,table2.int_col 
from table1 join table2 on table1.id = table2.id
```

**Plano Físico:**

```
== Physical Plan ==
AdaptiveSparkPlan isFinalPlan=false
+- SortMergeJoin [id#80L], [id#95L], Inner
   :- Sort [id#80L ASC NULLS FIRST], false, 0
   :  +- Exchange hashpartitioning(id#80L, 36), ENSURE_REQUIREMENTS, [id=#725]
   :     +- Filter isnotnull(id#80L)
   :        +- Scan ExistingRDD[id#80L,col#81]
   +- Sort [id#95L ASC NULLS FIRST], false, 0
      +- Exchange hashpartitioning(id#95L, 36), ENSURE_REQUIREMENTS, [id=#726]
         +- Filter isnotnull(id#95L)
            +- Scan ExistingRDD[id#95L,int_col#96L]
```

Número de registros processados: 799541  
Tempo de consulta: 21.87715196 segundos

**Consulta SQL com hint:**

```sql
select /*+ BROADCASTJOIN(table1)*/ table1.id,table1.col,table2.id,table2.int_col 
from table1 join table2 on table1.id = table2.id
```

**Plano Físico:**

```
AdaptiveSparkPlan isFinalPlan=false
+- BroadcastHashJoin [id#271L], [id#286L], Inner, BuildLeft, false
   :- BroadcastExchange HashedRelationBroadcastMode(List(input[0, bigint, false]),false), [id=#955]
   :  +- Filter isnotnull(id#271L)
   :     +- Scan ExistingRDD[id#271L,col#272]
   +- Filter isnotnull(id#286L)
      +- Scan ExistingRDD[id#286L,int_col#287L]
```

Número de registros processados: 799541  
Tempo de consulta: 15.35717314 segundos

## MERGE

A junção Shuffle Sort Merge é preferida quando ambos os conjuntos de dados são grandes e não podem caber na memória. Como o nome indica, esta junção envolve as seguintes três fases:

- **Fase de Shuffle** – Ambos os conjuntos de dados na consulta de junção são embaralhados (shuffled).
- **Fase de Sort** – Os registros são ordenados pela chave de junção em ambos os lados.
- **Fase de Merge** – Ambos os lados da condição de junção são iterados, com base na chave de junção.

A imagem a seguir mostra uma visualização de Grafo Acíclico Dirigido (DAG) de uma junção Shuffle Sort Merge. Ambas as tabelas são lidas nos primeiros dois estágios. No próximo estágio (estágio 17), elas são embaralhadas, ordenadas e então mescladas no final.

**Nota:** Alguns dos estágios nesta imagem aparecem como ignorados porque essas etapas foram concluídas em estágios anteriores. Esses dados foram armazenados em cache ou persistidos para uso nesses estágios.

O seguinte é um plano físico que indica uma junção Sort Merge:

![[Pasted image 20251121211711.png]]

```
== Physical Plan ==
AdaptiveSparkPlan isFinalPlan=false
+- SortMergeJoin [id#320L], [id#335L], Inner
   :- Sort [id#320L ASC NULLS FIRST], false, 0
   :  +- Exchange hashpartitioning(id#320L, 36), ENSURE_REQUIREMENTS, [id=#1018]
   :     +- Filter isnotnull(id#320L)
   :        +- Scan ExistingRDD[id#320L,col#321]
   +- Sort [id#335L ASC NULLS FIRST], false, 0
      +- Exchange hashpartitioning(id#335L, 36), ENSURE_REQUIREMENTS, [id=#1019]
         +- Filter isnotnull(id#335L)
            +- Scan ExistingRDD[id#335L,int_col#336L]
```

## SHUFFLE_HASH

A junção Shuffle Hash, como o nome indica, funciona embaralhando ambos os conjuntos de dados. As mesmas chaves de ambos os lados terminam na mesma partição ou tarefa. Após os dados serem embaralhados, o menor dos dois conjuntos de dados passa por hash em buckets e, então, uma junção hash é realizada dentro da partição. Uma junção Shuffle Hash é diferente de uma junção Broadcast Hash porque o conjunto de dados inteiro não é transmitido. Uma junção Shuffle Hash é dividida em duas fases:

- **Fase de Shuffle** – Ambos os conjuntos de dados são embaralhados.
- **Fase de Hash join** – O lado menor dos dados passa por hash, é colocado em buckets e então é unido por hash com o lado maior em todas as partições.

A ordenação não é necessária com junções Shuffle Hash dentro das partições. A imagem a seguir mostra as fases da junção Shuffle Hash. Os dados são lidos inicialmente, depois são embaralhados e, em seguida, um hash é criado e usado para a junção.

Por padrão, o otimizador escolhe a junção Shuffle Hash quando a junção Broadcast Hash não pode ser usada. Com base no tamanho do limite da junção Broadcast Hash (`spark.sql.autoBroadcastJoinThreshold`) e no número de partições de shuffle (`spark.sql.shuffle.partitions`) escolhido, ele usa a junção Shuffle Hash quando a partição única do SQL lógico é pequena o suficiente para construir uma tabela hash local.

## SHUFFLE_REPLICATE_NL

A junção Shuffle-and-Replicate Nested Loop, também conhecida como junção Produto Cartesiano, funciona de forma muito similar a uma junção Broadcast Hash, exceto que o conjunto de dados não é transmitido.

Neste algoritmo de junção, shuffle não se refere a um verdadeiro embaralhamento porque registros com as mesmas chaves não são enviados para a mesma partição. Em vez disso, a partição inteira de ambos os conjuntos de dados é copiada pela rede. Quando as partições dos conjuntos de dados estão disponíveis, uma junção Nested Loop é realizada. Se há `X` número de registros no primeiro conjunto de dados e `Y` número de registros no segundo conjunto de dados em cada partição, cada registro no segundo conjunto de dados é unido com cada registro no primeiro conjunto de dados. Isso continua em um loop `X × Y` vezes em cada partição.


















# Otimizar shuffles

Tradução do seguinte artigo: https://docs.aws.amazon.com/prescriptive-guidance/latest/tuning-aws-glue-for-apache-spark/optimize-shuffles.html

Certas operações, como `join()` e `groupByKey()`, exigem que o Spark execute um shuffle. O shuffle é o mecanismo do Spark para redistribuir dados de modo que sejam agrupados de maneira diferente entre as partições de RDD. O shuffle pode ajudar a remediar gargalos de desempenho. No entanto, como o shuffle geralmente envolve a cópia de dados entre executores Spark, o shuffle é uma operação complexa e custosa. Por exemplo, shuffles geram os seguintes custos:

- **I/O de disco:**
    
    - Gera um grande número de arquivos intermediários no disco.
- **I/O de rede:**
    
    - Precisa de muitas conexões de rede (Número de conexões = Mapper × Reducer).
    - Como os registros são agregados a novas partições de RDD que podem estar hospedadas em um executor Spark diferente, uma fração substancial do seu conjunto de dados pode se mover entre executores Spark pela rede.
- **Carga de CPU e memória:**
    
    - Ordena valores e mescla conjuntos de dados. Essas operações são planejadas no executor, colocando uma carga pesada no executor.

O shuffle é um dos fatores mais substanciais na degradação do desempenho de sua aplicação Spark. Ao armazenar os dados intermediários, ele pode esgotar o espaço no disco local do executor, o que causa a falha do trabalho Spark.

Você pode avaliar o desempenho do seu shuffle nas métricas do CloudWatch e na UI do Spark.

## Métricas do CloudWatch

Se o valor de Shuffle Bytes Written for alto em comparação com Shuffle Bytes Read, seu trabalho Spark pode estar usando operações de shuffle como `join()` ou `groupByKey()`.

## UI do Spark

Na aba Stage da UI do Spark, você pode verificar os valores de Shuffle Read Size / Records. Você também pode vê-los na aba Executors.

Na captura de tela a seguir, cada executor troca aproximadamente 18.6GB/4020000 registros com o processo de shuffle, para um tamanho total de leitura de shuffle de cerca de 75 GB.

A coluna Shuffle Spill (Disk) mostra uma grande quantidade de memória de dados derramada para o disco, o que pode causar um disco cheio ou um problema de desempenho.

Se você observar esses sintomas e o estágio demorar muito em comparação com suas metas de desempenho, ou falhar com erros `Out Of Memory` ou `No space left on device`, considere as seguintes soluções.

## Otimizar a junção (join)

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

### Use DataFrame Join

Tente usar uma API de alto nível do Spark, como SparkSQL, DataFrame e Datasets, em vez da API RDD ou junção de DynamicFrame. Você pode converter DynamicFrame para DataFrame com uma chamada de método como `dyf.toDF()`. Como discutido na seção Tópicos-chave no Apache Spark, essas operações de junção aproveitam internamente a otimização de consulta pelo otimizador Catalyst.

### Shuffle e broadcast hash joins e hints

O Spark suporta dois tipos de junção: shuffle join e broadcast hash join. Uma broadcast hash join não requer embaralhamento e pode exigir menos processamento do que uma shuffle join. No entanto, é aplicável apenas ao unir uma tabela pequena a uma grande. Ao unir uma tabela que pode caber na memória de um único executor Spark, considere usar uma broadcast hash join.

O diagrama a seguir mostra a estrutura de alto nível e as etapas de uma broadcast hash join e uma shuffle join.
![[Pasted image 20251121221803.png]]

Os detalhes de cada junção são os seguintes:

- **Shuffle join:**
    - A shuffle hash join une duas tabelas sem ordenação e distribui a junção entre as duas tabelas. É adequada para junções de tabelas pequenas que podem ser armazenadas na memória do executor Spark.
    - A sort-merge join distribui as duas tabelas a serem unidas por chave e as ordena antes de unir. É adequada para junções de tabelas grandes.
- **Broadcast hash join:**
    - Uma broadcast hash join envia o RDD ou tabela menor para cada um dos nós worker. Em seguida, faz uma combinação do lado do mapa com cada partição do RDD ou tabela maior.

É adequada para junções quando um de seus RDDs ou tabelas pode caber na memória ou pode ser feito caber na memória. É benéfico fazer uma broadcast hash join quando possível, porque não requer um shuffle. Você pode usar uma dica de junção para solicitar uma broadcast join do Spark da seguinte forma:

```python
# DataFrame
from pySpark.sql.functions import broadcast
df_joined= df_big.join(broadcast(df_small), right_df[key] == left_df[key], how='inner')
```

```sql
-- SparkSQL
SELECT /*+ BROADCAST(t1) */ FROM t1 INNER JOIN t2 ON t1.key = t2.key;
```

Para mais informações sobre dicas de junção, veja Join hints.

No AWS Glue 3.0 e posterior, você pode aproveitar broadcast hash joins automaticamente habilitando Adaptive Query Execution e parâmetros adicionais. Adaptive Query Execution converte uma sort-merge join em uma broadcast hash join quando as estatísticas de tempo de execução de qualquer lado da junção forem menores que o limite adaptativo de broadcast hash join.

No AWS Glue 3.0, você pode habilitar Adaptive Query Execution configurando `spark.sql.adaptive.enabled=true`. Adaptive Query Execution está habilitado por padrão no AWS Glue 4.0.

Você pode definir parâmetros adicionais relacionados a shuffles e broadcast hash joins:

- `spark.sql.adaptive.localShuffleReader.enabled`
- `spark.sql.adaptive.autoBroadcastJoinThreshold`

Para mais informações sobre parâmetros relacionados, veja Converting sort-merge join to broadcast join.

No AWS Glue 3.0 e posterior, você pode usar outras dicas de junção para shuffle para ajustar seu comportamento:

```sql
-- Join Hints para shuffle sort merge join
SELECT /*+ SHUFFLE_MERGE(t1) */ FROM t1 INNER JOIN t2 ON t1.key = t2.key;
SELECT /*+ MERGEJOIN(t2) */ FROM t1 INNER JOIN t2 ON t1.key = t2.key;
SELECT /*+ MERGE(t1) */ FROM t1 INNER JOIN t2 ON t1.key = t2.key;

-- Join Hints para shuffle hash join
SELECT /*+ SHUFFLE_HASH(t1) */ FROM t1 INNER JOIN t2 ON t1.key = t2.key;

-- Join Hints para shuffle-and-replicate nested loop join
SELECT /*+ SHUFFLE_REPLICATE_NL(t1) */ FROM t1 INNER JOIN t2 ON t1.key = t2.key;
```

### Use bucketing

A sort-merge join requer duas fases, shuffle e sort, e depois merge. Essas duas fases podem sobrecarregar o executor Spark e causar problemas de OOM e desempenho quando alguns dos executores estão mesclando e outros estão ordenando simultaneamente. Em tais casos, pode ser possível unir de forma eficiente usando bucketing. O bucketing vai pré-embaralhar e pré-ordenar sua entrada nas chaves de junção e, em seguida, escrever esses dados ordenados em uma tabela intermediária. O custo das etapas de shuffle e sort pode ser reduzido ao unir tabelas grandes definindo as tabelas intermediárias ordenadas com antecedência.

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
    
- Outra página importante é Summary Metrics, que mostra estatísticas para tarefas Spark. A captura de tela a seguir mostra métricas com percentis para Duration, GC Time, Spill (memory), Spill (disk) e assim por diante.
    

Quando as tarefas são distribuídas uniformemente, você verá números semelhantes em todos os percentis. Quando há distorção de dados, você verá valores muito tendenciosos em cada percentil. No exemplo, a duração da tarefa é menor que 13 segundos em Min, 25º percentil, Mediana e 75º percentil. Enquanto a tarefa Max processou 100 vezes mais dados do que o 75º percentil, sua duração de 6,4 minutos é cerca de 30 vezes mais longa. Isso significa que pelo menos uma tarefa (ou até 25% das tarefas) levou muito mais tempo do que o restante das tarefas.

Se você ver distorção de dados, tente o seguinte:

- Se você usa AWS Glue 3.0, habilite Adaptive Query Execution configurando `spark.sql.adaptive.enabled=true`. Adaptive Query Execution está habilitado por padrão no AWS Glue 4.0.
    
    Você também pode usar Adaptive Query Execution para distorção de dados introduzida por junções configurando os seguintes parâmetros relacionados:
    
    - `spark.sql.adaptive.skewJoin.skewedPartitionFactor`
    - `spark.sql.adaptive.skewJoin.skewedPartitionThresholdInBytes`
    - `spark.sql.adaptive.advisoryPartitionSizeInBytes=128m` (128 mebibytes ou maior deve ser bom)
    - `spark.sql.adaptive.coalescePartitions.enabled=true` (quando você quiser unir partições)
    
    Para mais informações, veja a documentação do Apache Spark.
    
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

**Nota:** Evite usar Spark `collect()` em ambientes comerciais o máximo possível. A ação `collect()` retorna todos os resultados de um cálculo no executor Spark para o driver Spark, o que pode causar um erro OOM no driver Spark. Para evitar um erro OOM, o Spark define `spark.driver.maxResultSize = 1GB` por padrão, o que limita o tamanho máximo de dados retornados ao driver Spark para 1 GB.