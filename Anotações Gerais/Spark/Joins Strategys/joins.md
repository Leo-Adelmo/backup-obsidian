
# Estratégias de Join no PySpark

No PySpark, a escolha da estratégia de join é crucial para o desempenho de suas aplicações. O Spark otimiza automaticamente os joins, mas entender as estratégias disponíveis ajuda a escrever código mais eficiente.

## Parte 1: Tipos Tradicionais de Join

Antes de entender as estratégias de otimização, é fundamental conhecer os tipos de join que determinam **quais dados** são retornados.

### 1. **Inner Join (Padrão)**

Retorna apenas as linhas que têm valores correspondentes em ambos os DataFrames.

**Exemplo:**

```python
# Inner join - retorna apenas matches
df_result = df1.join(df2, df1.id == df2.id, "inner")
# ou simplesmente (inner é o padrão)
df_result = df1.join(df2, df1.id == df2.id)
```

**Quando usar:**

- Quando você quer apenas registros que existem em ambas as tabelas
- Para análises que requerem correspondência completa
- É o join mais restritivo e geralmente o mais rápido

**[Documentação oficial sobre Inner Join](https://spark.apache.org/docs/latest/sql-ref-syntax-qry-select-join.html)**

---

### 2. **Left Join (Left Outer Join)**

Retorna todas as linhas do DataFrame da esquerda e as linhas correspondentes do DataFrame da direita. Quando não há correspondência, preenche com `null`.

**Exemplo:**

```python
# Left join - mantém todos da esquerda
df_result = df1.join(df2, df1.id == df2.id, "left")
# ou
df_result = df1.join(df2, df1.id == df2.id, "left_outer")
```

**Quando usar:**

- Quando você quer preservar todos os registros da tabela principal
- Para identificar registros sem correspondência (valores null)
- Comum em análises de cobertura de dados

**[Guia sobre Left Outer Join no PySpark](https://sparkbyexamples.com/pyspark/pyspark-join-explained-with-examples/)**

---

### 3. **Right Join (Right Outer Join)**

O oposto do Left Join: retorna todas as linhas do DataFrame da direita e as correspondentes da esquerda.

**Exemplo:**

```python
# Right join - mantém todos da direita
df_result = df1.join(df2, df1.id == df2.id, "right")
# ou
df_result = df1.join(df2, df1.id == df2.id, "right_outer")
```

**Quando usar:**

- Quando a tabela de referência está à direita
- Menos comum que left join (você pode inverter a ordem para usar left)

**[Tutorial sobre Right Join](https://iomete.com/resources/reference/pyspark/pyspark-join)**

---

### 4. **Full Outer Join**

Retorna todas as linhas de ambos os DataFrames, preenchendo com `null` onde não há correspondência.

**Exemplo:**

```python
# Full outer join - mantém todos de ambos
df_result = df1.join(df2, df1.id == df2.id, "outer")
# Aliases: "full", "fullouter", "full_outer"
df_result = df1.join(df2, df1.id == df2.id, "full")
```

**Quando usar:**

- Quando você quer identificar todas as diferenças entre datasets
- Para análises de reconciliação de dados
- Quando não quer perder nenhum registro de nenhuma tabela

**[Documentação sobre Full Outer Join](https://www.projectpro.io/article/pyspark-joins-for-data-analysis-by-example/564)**

---

### 5. **Left Semi Join**

Retorna apenas as linhas do DataFrame da esquerda que têm correspondência na direita, mas **sem incluir colunas** do DataFrame da direita.

**Exemplo:**

```python
# Left semi join - filtra sem adicionar colunas
df_result = df1.join(df2, df1.id == df2.id, "left_semi")
# ou "leftsemi", "left_semi"
```

**Quando usar:**

- Quando você quer filtrar dados baseado em outra tabela
- Para verificar existência sem duplicar dados
- Alternativa mais eficiente que inner join seguido de select

**[Exemplo prático de Left Semi Join](https://medium.com/@anowerhossain97/cracking-pyspark-joins-inner-outer-left-right-more-explained-bf41912e070d)**

---

### 6. **Left Anti Join**

Retorna apenas as linhas do DataFrame da esquerda que **não têm** correspondência na direita.

**Exemplo:**

```python
# Left anti join - registros sem correspondência
df_result = df1.join(df2, df1.id == df2.id, "left_anti")
# ou "leftanti", "anti"
```

**Quando usar:**

- Para encontrar registros órfãos
- Validação de integridade de dados
- Identificar dados faltantes em outra tabela

**[Comparação entre tipos de Join no PySpark](https://www.datacamp.com/tutorial/pyspark-joins)**

---

### 7. **Cross Join**

Produto cartesiano: combina cada linha de um DataFrame com cada linha do outro.

**Exemplo:**

```python
# Cross join - produto cartesiano (CUIDADO!)
df_result = df1.crossJoin(df2)
```

**Quando usar:**

- Raramente! Apenas quando realmente necessário
- Testes de combinações de parâmetros
- Datasets muito pequenos

**Atenção:** Se df1 tem 1000 linhas e df2 tem 2000 linhas, o resultado terá 2.000.000 de linhas!

**[Documentação oficial de tipos de Join](https://spark.apache.org/docs/latest/api/python/reference/pyspark.sql/api/pyspark.sql.DataFrame.join.html)**

---

## Parte 2: Estratégias de Join (Join Strategies)

As estratégias determinam **como** o Spark executa fisicamente o join para otimizar desempenho. O Spark escolhe automaticamente, mas você pode influenciar essa escolha.

### 1. **Broadcast Hash Join**
![[Pasted image 20251121205247.png]]
É a estratégia mais rápida quando uma das tabelas é pequena o suficiente para caber na memória de cada executor.

**Como funciona:**

- A tabela menor é transmitida (broadcast) para todos os nós
- Cada nó faz o join localmente sem necessidade de shuffle
- Ideal quando uma tabela tem menos de 10MB (configurável)

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

**[Guia AWS sobre Broadcast Hash Join](https://docs.aws.amazon.com/prescriptive-guidance/latest/spark-tuning-glue-emr/using-join-hints-in-spark-sql.html)**

---

### 2. **Shuffle Hash Join**

Usado quando ambas as tabelas são grandes demais para broadcast, mas uma pode caber na memória após o particionamento.

**Como funciona:**

- Ambos os DataFrames são embaralhados (shuffle) pela chave de join
- Dados com a mesma chave vão para a mesma partição
- Constrói uma hash table para a tabela menor em cada partição

**Quando é escolhido:**

- Tabelas médias/grandes
- Uma tabela significativamente menor que a outra
- Hash join habilitado: `spark.sql.join.preferSortMergeJoin=false`

**[Explicação detalhada do Shuffle Hash Join](https://plainenglish.io/blog/a-deep-dive-into-the-inner-workings-of-apache-spark-join-strategies)**

---

### 3. **Sort Merge Join**

É a estratégia padrão para joins de tabelas grandes.

**Como funciona:**

- Ambos os DataFrames são embaralhados e ordenados pela chave de join
- As partições correspondentes são mescladas
- Não requer que os dados caibam na memória

**Exemplo:**

```python
# Sort Merge Join (padrão)
df_result = df1.join(df2, df1.id == df2.id)

# Verificar o plano de execução
df_result.explain()
```

**Vantagens:**

- Funciona com datasets de qualquer tamanho
- Mais eficiente em memória
- Suporta todos os tipos de join

**Desvantagens:**

- Requer shuffle (operação custosa)
- Overhead da ordenação

**[Documentação oficial sobre Performance Tuning](https://spark.apache.org/docs/latest/sql-performance-tuning.html)**

---

### 4. **Cartesian Join (Cross Join)**

Quando não há condição de join ou quando o otimizador não encontra uma estratégia melhor.

**Como funciona:**

- Produto cartesiano de todas as linhas
- Extremamente custoso

**Exemplo:**

```python
# Cross join explícito
df_result = df1.crossJoin(df2)
```

**Atenção:** Evite sempre que possível!

**[Guia sobre otimização de shuffles](https://docs.aws.amazon.com/prescriptive-guidance/latest/tuning-aws-glue-for-apache-spark/optimize-shuffles.html)**

---

### 5. **Broadcast Nested Loop Join**

Usado para joins sem condição equi-join (sem `=`) ou para alguns OUTER joins.

**Como funciona:**

- Similar ao broadcast, mas usa loops aninhados
- Mais lento que Broadcast Hash Join

**[Comparação entre estratégias de Join](https://medium.com/@deepa.account/apache-spark-join-strategies-ee987efc3a3e)**

---

## Parte 3: Como as Estratégias Afetam o Código

### 1. **Impacto na Sintaxe do Código**

As estratégias de join **não mudam a sintaxe básica** do código, mas você pode forçá-las usando hints:

```python
from pyspark.sql.functions import broadcast

# Sintaxe padrão (Spark escolhe a estratégia)
df_result = df1.join(df2, df1.id == df2.id, "inner")

# Forçar Broadcast Join
df_result = df1.join(broadcast(df2), df1.id == df2.id, "inner")

# Forçar Sort Merge Join (usando hint no DataFrame)
df_result = df1.hint("merge").join(df2, df1.id == df2.id, "inner")

# Forçar Shuffle Hash Join
df_result = df1.hint("shuffle_hash").join(df2, df1.id == df2.id, "inner")

# Usando SQL com hints
spark.sql("""
    SELECT /*+ BROADCAST(t2) */ t1.id, t1.name, t2.value
    FROM table1 t1
    INNER JOIN table2 t2 ON t1.id = t2.id
""")
```

---

### 2. **Impacto no Desempenho**

A estratégia escolhida afeta drasticamente o tempo de execução:

```python
# Exemplo: Join de tabela grande (10GB) com pequena (50MB)

# SEM otimização (Sort Merge Join automático)
# Tempo: ~5 minutos com shuffle de 20GB de dados
df_result = df_grande.join(df_pequeno, "id")

# COM broadcast (Broadcast Hash Join)
# Tempo: ~30 segundos sem shuffle!
df_result = df_grande.join(broadcast(df_pequeno), "id")
```

**Diferença:** 10x mais rápido apenas escolhendo a estratégia certa!

---

### 3. **Impacto nas Configurações**

Você pode ajustar o comportamento através de configurações:

```python
# Configurar tamanho máximo para broadcast automático
spark.conf.set("spark.sql.autoBroadcastJoinThreshold", 50 * 1024 * 1024)  # 50MB

# Desabilitar broadcast automático
spark.conf.set("spark.sql.autoBroadcastJoinThreshold", -1)

# Preferir ou não Sort Merge Join
spark.conf.set("spark.sql.join.preferSortMergeJoin", "true")

# Número de partições após shuffle
spark.conf.set("spark.sql.shuffle.partitions", 200)
```

**Impacto:** Essas configurações alteram automaticamente qual estratégia o Spark usa, **sem mudar seu código de join**.

---

### 4. **Impacto no Plano de Execução**

A estratégia determina o plano físico de execução:

```python
df_result = df1.join(df2, "id")

# Ver o plano de execução
df_result.explain()

# Saída exemplo com Broadcast Hash Join:
# == Physical Plan ==
# BroadcastHashJoin [id], [id], Inner, BuildRight
#   :- Scan parquet table1
#   +- BroadcastExchange
#      +- Scan parquet table2

# Saída exemplo com Sort Merge Join:
# == Physical Plan ==
# SortMergeJoin [id], [id], Inner
#   :- Sort [id ASC]
#   :  +- Exchange hashpartitioning(id)
#   +- Sort [id ASC]
#      +- Exchange hashpartitioning(id)
```

**Impacto:** O plano mostra operações diferentes (BroadcastExchange vs Exchange+Sort), afetando recursos usados.

---

### 5. **Impacto no Uso de Recursos**

Cada estratégia consome recursos diferentes:

```python
# Broadcast Join
# - Memória: Alta no driver e executores (para a tabela broadcast)
# - Rede: Uma vez para broadcast
# - CPU: Baixa (hash lookup)
df_result = df_grande.join(broadcast(df_pequeno), "id")

# Sort Merge Join
# - Memória: Moderada (pode spill para disco)
# - Rede: Alta (shuffle de ambas as tabelas)
# - CPU: Alta (sorting)
df_result = df_grande.join(df_muito_grande, "id")
```

**Impacto:** Escolher a estratégia errada pode causar:

- OutOfMemoryError (broadcast muito grande)
- Spill to disk (dados não cabem na memória)
- Slow tasks (skew de dados)

---

### 6. **Impacto na Manutenibilidade do Código**

```python
# Código RUIM: Hard-coded broadcast sem verificação
df_result = df_vendas.join(broadcast(df_clientes), "cliente_id")
# Problema: Se df_clientes crescer muito, vai quebrar!

# Código BOM: Deixa Spark decidir com threshold configurado
spark.conf.set("spark.sql.autoBroadcastJoinThreshold", 100 * 1024 * 1024)
df_result = df_vendas.join(df_clientes, "cliente_id")

# Código MELHOR: Condicional baseado no tamanho
tamanho_clientes = df_clientes.count() * 100  # estimativa em bytes
if tamanho_clientes < 50 * 1024 * 1024:  # 50MB
    df_result = df_vendas.join(broadcast(df_clientes), "cliente_id")
else:
    df_result = df_vendas.join(df_clientes, "cliente_id")
```

---

### 7. **Impacto no Debugging**

Diferentes estratégias requerem diferentes abordagens de debug:

```python
# Verificar qual estratégia está sendo usada
df_result = df1.join(df2, "id")
print(df_result.explain(extended=True))

# Analisar problemas de performance
# Para Broadcast Join: verificar tamanho da tabela broadcast
df_pequeno.cache()
print(f"Tamanho: {df_pequeno.count()} linhas")

# Para Sort Merge Join: verificar skew de dados
df1.groupBy("id").count().orderBy(col("count").desc()).show(10)

# Habilitar logs detalhados
spark.sparkContext.setLogLevel("INFO")
```

---

## Configurações Importantes

```python
# Tamanho máximo para broadcast (padrão: 10MB)
spark.conf.set("spark.sql.autoBroadcastJoinThreshold", 10485760)

# Desabilitar broadcast automático
spark.conf.set("spark.sql.autoBroadcastJoinThreshold", -1)

# Preferir Sort Merge Join
spark.conf.set("spark.sql.join.preferSortMergeJoin", "true")

# Número de partições após shuffle
spark.conf.set("spark.sql.shuffle.partitions", 200)
```

**[Tutorial sobre configurações de performance](https://blog.nashtechglobal.com/advanced-spark-sql-joins-an-optimization-technique/)**

---

## Como o Spark Escolhe a Estratégia

O Catalyst Optimizer analisa:

1. Tamanho estimado dos DataFrames
2. Estatísticas das tabelas
3. Tipo de join (INNER, LEFT, RIGHT, FULL)
4. Condições de join
5. Configurações do cluster

**[Documentação sobre Sort-Merge Join](https://www.canadiandataguy.com/p/spark-join-strategies-explained-sort)**

---

## Dicas de Otimização

**Para melhorar performance:**

- Use broadcast para tabelas pequenas (dimensões)
- Particione adequadamente os dados antes de joins grandes
- Considere fazer cache de DataFrames reutilizados
- Use bucketing para joins frequentes nas mesmas colunas
- Analise o plano de execução com `.explain()`

**Exemplo de bucketing:**

```python
# Salvar com bucketing
df.write.bucketBy(10, "id").sortBy("id").saveAsTable("tabela_bucketed")

# Joins em tabelas bucketed são mais eficientes
df_result = spark.table("tabela1").join(spark.table("tabela2"), "id")
```

**[Tutorial sobre otimização de Joins no PySpark](https://sparkbyexamples.com/pyspark/pyspark-broadcast-join-with-example/)**

---

## Resumo: Quando Usar Cada Estratégia

|Estratégia|Cenário Ideal|Tamanho dos Dados|
|---|---|---|
|**Broadcast Hash Join**|Uma tabela pequena + uma grande|< 10MB broadcast|
|**Shuffle Hash Join**|Tabelas médias com chaves bem distribuídas|Médio|
|**Sort Merge Join**|Ambas as tabelas grandes|Grande|
|**Broadcast Nested Loop**|Joins não-equi ou cross joins pequenos|Pequeno|
|**Cartesian Join**|Evitar! Apenas se necessário|Muito pequeno|

A escolha da estratégia correta pode reduzir o tempo de execução de horas para minutos em grandes volumes de dados!