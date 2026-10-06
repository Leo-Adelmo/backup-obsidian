## Tipos de Join

### Inner Join

O **inner join** retorna apenas os registros que possuem correspondência em ambos os datasets. É o tipo de join mais comum e serve como base para a maioria das análises relacionais. Como envolve apenas interseções, geralmente é mais leve que joins externos.  
[Apache Spark – Join Operations](https://spark.apache.org/docs/latest/sql-ref-syntax-qry-select-join.html)

### Left, Right e Full Outer

Os **joins externos** preservam parcialmente ou totalmente os registros de um dos lados, mesmo quando não existe correspondência.

- **Left Outer**: mantém todos os registros da esquerda.
    
- **Right Outer**: mantém todos os registros da direita.
    
- **Full Outer**: mantém todos os registros de ambos os lados.
    

Esses joins ampliam a cobertura analítica, mas tendem a exigir mais shuffle e memória devido à necessidade de combinar registros não pareados.  
[Databricks – Join Fundamentals](https://docs.databricks.com/en/sql/language-manual/sql-ref-syntax-qry-select-join.html)

### Left Semi e Left Anti

Os **joins semi e anti** executam verificações de existência sem duplicar colunas do lado direito:

- **Left Semi**: retorna linhas da esquerda que possuem match.
    
- **Left Anti**: retorna linhas da esquerda que não possuem match.
    

São úteis para filtragens eficientes, reduzindo movimentações e evitando materializações desnecessárias.  
[Apache Spark – Join Types](https://spark.apache.org/docs/latest/sql-ref-syntax-qry-select-join.html)

### Cross Join

O **cross join** combina cada linha do dataset A com cada linha do dataset B. Por gerar explosão combinatória, só deve ser usado com extrema intenção e controle.  
[Databricks – Cross Join Behavior](https://docs.databricks.com/en/sql/language-manual/sql-ref-syntax-qry-select-join.html#cross)

---

## Estratégias de Execução (Join Strategies)

### Broadcast Hash Join

O **Broadcast Hash Join** replica o dataset menor para todos os executores, permitindo que o join ocorra localmente sem shuffle da tabela maior. É geralmente a estratégia mais rápida quando uma das tabelas cabe confortavelmente na memória. Controlado por `spark.sql.autoBroadcastJoinThreshold`.  
[Apache Spark – Performance Tuning](https://spark.apache.org/docs/latest/sql-performance-tuning.html#broadcast-hash-joins)

### Shuffle Hash Join

O **Shuffle Hash Join** embaralha ambos os lados usando a chave de join e constrói tabelas hash em cada partição. É útil quando as partições resultantes são pequenas e são adequadas para construção de hash tables.  
[Databricks – Join Optimization](https://docs.databricks.com/en/query-optimization/index.html)

### Sort-Merge Join

No **Sort-Merge Join**, ambos os datasets são embaralhados e ordenados por chave antes da mesclagem. É a estratégia mais robusta para grandes volumes de dados e é utilizada quando nem broadcast nem hash join são adequados.  
[Apache Spark – Sort Merge Join](https://spark.apache.org/docs/latest/sql-performance-tuning.html#sort-merge-joins)

### Broadcast Nested Loop Join

O **Broadcast Nested Loop Join** é aplicado quando não existe condição de igualdade ou quando o planner não pode utilizar outras estratégias. É custoso, porém necessário em alguns cenários específicos.  
[Apache Spark – Join Hints](https://spark.apache.org/docs/latest/sql-performance-tuning.html#join-hints)

---

## Como o Spark Escolhe a Estratégia

A decisão depende de estatísticas, heurísticas internas e configurações:

- Avaliação de tamanho das tabelas.
    
- Threshold de broadcast (`autoBroadcastJoinThreshold`).
    
- Necessidade de ordenação.
    
- Presença de condições de igualdade.
    

O uso de **Adaptive Query Execution (AQE)** permite conversões dinâmicas, como SMJ → BHJ quando uma side se mostrar pequena após um shuffle.  
[Databricks – Adaptive Query Execution](https://docs.databricks.com/en/query-optimization/adaptive-query-execution.html)

---

## Hints de Join

O PySpark permite orientar o otimizador usando hints:

- **BROADCAST** — força broadcast.
    
- **MERGE** — favorece sort-merge join.
    
- **SHUFFLE_HASH** — favorece shuffle hash join.
    
- **SHUFFLE_REPLICATE_NL** — força nested loop replicado.
    

Hints são úteis, mas devem ser aplicados apenas quando há certeza sobre o comportamento dos dados.  
[Apache Spark – Join Hints](https://spark.apache.org/docs/latest/sql-performance-tuning.html#join-hints)

---

## Boas Práticas

- Projete apenas as colunas necessárias antes do join.
    
- Avalie o tamanho real da tabela menor antes de usar broadcast.
    
- Ajuste `spark.sql.shuffle.partitions` para refletir a capacidade do cluster.
    
- Evite cross joins sem necessidade explícita.
    
- Use particionamento por chave quando há joins frequentes na mesma coluna.
    
- Utilize hints apenas quando necessário; priorize AQE.
    

---

Se quiser, ajusto o artigo para o seu template institucional (ex.: Confluence, Notion, Markdown corporativo).