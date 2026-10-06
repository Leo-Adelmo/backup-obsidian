## SCD Tipo 2 — definição e princípios (Kimball)

No contexto de modelagem dimensional, Ralph Kimball define o SCD tipo 2 como a técnica que preserva o histórico completo de mudanças nos atributos de uma entidade, inserindo novas linhas para cada alteração (em vez de sobrescrever). ([Kimball Group](https://www.kimballgroup.com/wp-content/uploads/2013/08/2013.09-Kimball-Dimensional-Modeling-Techniques11.pdf"Kimball Dimensional Modeling Techniques"))

No documento _Kimball Dimensional Modeling Techniques_ (versão PDF) aparece:

> “Type 2: Add New Row … cria nova linha com os valores atualizados e preserva a linha antiga com seus valores anteriores. Essa técnica exige generalizar a chave primária do dimensionamento além da chave natural, pois pode haver múltiplas linhas para um mesmo membro da dimensão.” ([Kimball Group](https://www.kimballgroup.com/wp-content/uploads/2013/08/2013.09-Kimball-Dimensional-Modeling-Techniques11.pdf))

O artifício de “generalizar a chave” significa que não podemos usar apenas a natural key (business key) como PK, porque essa key não diferencia diferentes versões históricas. É necessário um identificador adicional (surrogate) que é único por versão. ([Kimball Group](https://www.kimballgroup.com/wp-content/uploads/2013/08/2013.09-Kimball-Dimensional-Modeling-Techniques11.pdf)
Outra fonte ( de Kimball) explica o raciocínio da partição lógica da história:

> “Sempre que encontramos uma mudança em um registro de dimensão, emitimos um novo registro e o adicionamos à tabela de dimensão existente. Não alteramos registros de fatos anteriores para apontar à nova chave de dimensão.” ([Kimball Group](https://www.kimballgroup.com/wp-content/uploads/2012/05/DT8Perfectly.pdf"))

Para operacionalizar isso, cada linha de dimensão deve conter pelo menos três colunas extras:

1. **effective_from** (data/hora de início da validade)
    
2. **effective_to** (data/hora de expiração ou valor sem ano de expiração, como: `9999-12-31`)
    
3. **current_flag** (ou `is_current`) para indicar a versão ativa atual. ([Kimball Group](https://www.kimballgroup.com/wp-content/uploads/2013/08/2013.09-Kimball-Dimensional-Modeling-Techniques11.pdf?utm_source=chatgpt.com "Kimball Dimensional Modeling Techniques"))
    

Ou seja, no SCD2, ao receber uma nova versão de um membro de dimensão:

- Localizamos a versão atualmente ativa (current_flag = true) para aquela business key.
    
- Ajustamos seu `effective_to` (e marcamos `is_current = false`)
    
- Inserimos uma nova linha com `effective_from = tempo da mudança`, `effective_to = null` ou `sentinel`, e `is_current = true`.
    
- Registros de fato que ocorrerem depois da mudança apontam para esse novo surrogate key.
    
Imagem 1: ([Implement historical record lookup and Slowly Changing Dimensions Type-2 using](https://aws.amazon.com/pt/blogs/big-data/implement-historical-record-lookup-and-slowly-changing-dimensions-type-2-using-apache-iceberg/))
![[Pasted image 20251003203146.png]]

Essa estratégia preserva uma “partição lógica” da história: cada linha histórica pertence a um “intervalo de validade” e ao consultarmos um fato, usamos a linha de dimensão válida no momento do fato. ([Kimball Group](https://www.kimballgroup.com/wp-content/uploads/2012/05/DT8Perfectly.pdf ))

---
## Recomendações de chave primária / surrogate keys

### Uso de surrogate key

- A surrogate key (chave artificial) deve ser usada como chave primária da tabela de dimensão, porque permite diferenciar versões históricas (várias versões de um mesmo business key) sem conflito. (Kimball, vários textos) ([Kimball Group](https://www.kimballgroup.com/wp-content/uploads/2012/05/DT8Perfectly.pdf?utm_source=chatgpt.com "Perfectly Partitioning History With The Type 2 Slowly ..."))
    
- A surrogate key típica é um número sequencial, identidade (auto increment) ou um identificador gerado no pipeline, conforme diz os textos abaixo:
  

- **Kimball Group — _Slowly Changing Dimensions, Part 2_**
> 	“Em vez disso, você deve criar chaves primárias completamente artificiais que sejam simplesmente inteiros atribuídos sequencialmente. Chamamos essas chaves de _surrogate keys_. É necessário criar uma nova chave substituta sempre que um tipo 2 de mudança é processado em uma dimensão.”  
> 	_(Tradução livre de: “Rather, you should create completely artificial primary keys that are simply sequentially assigned integers. We call these keys surrogate keys. You must make a new surrogate primary key whenever you process a Type 2 change in a dimension.”)_  
> 	**Fonte:** [Kimball Group – Slowly Changing Dimensions, Part 2](https://www.kimballgroup.com/2008/09/slowly-changing-dimensions-part-2/?utm_source=chatgpt.com)

> 	**Kimball Group — _Design Tip #100: Keep Your Keys Simple_**

> 	“Cada nova linha criada como resultado de uma mudança do tipo 2 deve receber sua própria chave substituta. Essas chaves devem permanecer simples — normalmente inteiros sequenciais — e nunca devem ser reutilizadas ou derivadas da chave natural.”  
> 	_(Tradução livre de: “Each new row created as a result of a type 2 change should be assigned its own surrogate key. These keys should remain simple — typically sequential integers — and should never be reused or derived from the natural key.”)_  
> 	**Fonte:** [Kimball Group – Design Tip #100: Keep Your Keys Simple (PDF)](https://www.kimballgroup.com/wp-content/uploads/2012/05/DT100KeepYourKeysSimple.pdf?utm_source=chatgpt.com)
    
- **Business key** (ou chave natural) deve ser armazenada como coluna separada (não como PK), para permitir junção das novas versões com o registro original.
---
## Chave primária sequencial: útil ou não?

### Vantagens de usar sequência (incremental) como _surrogate key_

- Simples de gerar, eficiente para indexação, ordenação e compactação de dados.
    
- Boa para bancos tradicionais ou componentes centralizados (quando não há particionamento extremo).
    
- Facilita _scans_ ordenados por _surrogate_.

> “Essas chaves substitutas de dimensão são inteiros simples, atribuídos em sequência, começando pelo valor 1, toda vez que uma nova chave é necessária.”  
> _(Tradução livre de: “These dimension surrogate keys are simple integers, assigned in sequence, starting with the value 1, every time a new key is needed.”)_  
> **Fonte:** [Kimball Group – Dimension Surrogate Key](https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/kimball-techniques/dimensional-modeling-techniques/dimension-surrogate-key/?utm_source=chatgpt.com)

---

### Desvantagens / riscos em ambiente distribuído

- Em sistemas distribuídos (Spark, Lakehouses), gerar sequência globalmente consistente pode exigir coordenação central (_bottleneck_).
    
- Se várias partições tentam gerar sequências independentes, há risco de colisão ou _gaps_.
    
- Em pipelines de ingestão paralela, usar sequência global pode se tornar gargalo.
    

> “Cada nova linha criada como resultado de uma mudança do tipo 2 deve receber sua própria chave substituta. Essas chaves devem permanecer simples — normalmente inteiros sequenciais — e nunca devem ser reutilizadas ou derivadas da chave natural.”  
> _(Tradução livre de: “Each new row created as a result of a type 2 change should be assigned its own surrogate key. These keys should remain simple — typically sequential integers — and should never be reused or derived from the natural key.”)_  
> **Fonte:** [Kimball Group – Design Tip #100: Keep Your Keys Simple](https://www.kimballgroup.com/wp-content/uploads/2012/05/DT100KeepYourKeysSimple.pdf?utm_source=chatgpt.com)

---
(Fontes Estranhas!! Corrigir e validar !!)
### Abordagens modernas

Por esses motivos, muitos _designs_ modernos evitam sequência estrita e preferem:

- Gerar _surrogate key_ no pipeline de ingestão com coordenação (e.g. zona de coordenação ou serviço de sequência).
    
- Usar identificadores distribuídos como **UUIDs** ou combinar _hash_ + incremento local.
    
- Ou utilizar mecanismos nativos da engine (ex: `IDENTITY` em Delta/Iceberg) se suportado.


> “A surrogate key típica é um número sequencial, identidade (_auto increment_) ou um identificador gerado no pipeline.”  
> _(Tradução livre baseada na definição de surrogate key do Kimball Group e da Microsoft Docs)_  
> **Fonte:**
> - [Kimball Group – Slowly Changing Dimensions, Part 2](https://www.kimballgroup.com/2008/09/slowly-changing-dimensions-part-2/?utm_source=chatgpt.com)
>     
> - [Microsoft Learn – Surrogate Keys (Data Warehousing)](https://learn.microsoft.com/en-us/sql/ssas/data-mining/surrogate-keys-in-data-warehousing?utm_source=chatgpt.com)
>     

Em _lakehouses_ modernos, muitas implementações evitam sequência global explícita e adotam _approaches_ baseados em **chaves lógicas + versionamento**, especialmente com Apache Iceberg e Delta Lake.

---

## Uso de hash para detecção de mudanças

### Por que usar hash?

O _hash_ resolve o problema de detectar quando **qualquer campo rastreado** em uma dimensão mudou.  
Comparar campo a campo é custoso; com _hash_, basta verificar se `old_hash != new_hash`.

> “Calcule um hash da linha a partir dos campos rastreados. Se o hash da nova linha for diferente do hash armazenado, então os dados mudaram e uma nova versão deve ser criada.”  
> _(Tradução livre de: “Calculate a row hash from the tracked fields. If the hash of the new row differs from the stored hash, then the data has changed and a new version should be created.”)_  
> **Fonte:** [Iteration Insights – How to Implement SCD Type 2 Using Delta Table](https://iterationinsights.com/article/how-to-implement-slowly-changing-dimensions-scd-type-2-using-delta-table/?utm_source=chatgpt.com)

Vantagens:

- Determinístico
    
- Compacto (representa o “fingerprint” da linha)
    
- Facilita _merge_ e detecção de alterações
    

---

### É “necessário”?

Não é obrigatório, mas é **fortemente recomendado** em:

- Cenários com muitas colunas rastreadas
    
- Alto volume de dados
    
- Pipelines distribuídos (Spark, Flink etc.)
    
- Fontes sem CDC explícito
    

Quando **não usar hash**:

- Poucas colunas → comparação direta pode bastar.
    
- Custo de CPU alto.
    
- Engine já detecta elas nativamente.
    
---

### Algoritmos comuns

- **Murmur3** – rápido e leve, ideal para _throughput_ alto. (Alto risco de colisão)
    
- **SHA-256 / SHA-512** – mais seguro contra colisões, usado quando integridade é crítica.
    
- **Fallback** – em caso de colisão suspeita, comparar campo-a-campo.
    

> “O uso de funções de hash como Murmur3 ou SHA-256 é comum para detecção de mudanças em ambientes distribuídos, pois permite identificar modificações sem comparar diretamente todas as colunas.”  
> _(Tradução livre de: Databricks – Delta Live Tables: Implementing Slowly Changing Dimensions)_  
> **Fonte:** [Databricks – Implementing SCD Type 2 in Delta Live Tables](https://docs.databricks.com/en/delta-live-tables/delta-live-tables-scd.html?utm_source=chatgpt.com)

---
(Trecho não validado!)

## Caso Iceberg / pipeline moderno

O blog da AWS “Implement historical record lookup and Slowly Changing Dimensions Type-2 using Apache Iceberg” propõe uma forma moderna de usar Iceberg para resolver parte da complexidade de manter SCD2. ([Amazon Web Services, Inc.](https://aws.amazon.com/blogs/big-data/implement-historical-record-lookup-and-slowly-changing-dimensions-type-2-using-apache-iceberg/?utm_source=chatgpt.com "Implement historical record lookup and Slowly Changing ..."))

Resumo:

- Em vez de manter manualmente colunas de `effective_from`, `effective_to` e `current_flag` na tabela física, usa-se _views_ ou _lógicas de consulta_ para construir a “visão SCD2” a partir do log de mudanças (change log) que o Iceberg mantém.
    
- Ou seja, a tabela Iceberg registra as mudanças incrementais (por exemplo operações INSERT/UPDATE) e uma **view** pode projetar o estado histórico conforme você consulta num ponto no tempo ou usando lógica temporal. ([Amazon Web Services, Inc.](https://aws.amazon.com/blogs/big-data/implement-historical-record-lookup-and-slowly-changing-dimensions-type-2-using-apache-iceberg/?utm_source=chatgpt.com "Implement historical record lookup and Slowly Changing ..."))
    
- Isso elimina (ou reduz) a necessidade de gerenciar explicitamente versões na tabela física — o engine abstratamente gera snapshots históricos.
    
- Esse approach é mais eficiente em ambientes distribuídos modernos, pois delega ao motor de armazenamento (Iceberg) a gestão da história.
    

Além disso, há um artigo em Medium (“Building Efficient SCD2 Pipelines with Apache Iceberg …”) que discute implementação com Spark SQL em Iceberg, combinando a lógica de versionamento com partições de tempo e mecanismos nativos do Iceberg. ([Medium](https://medium.com/%40usefusefi/building-efficient-scd2-pipelines-with-apache-iceberg-and-spark-sql-a-complete-guide-to-time-based-8e5324de0d04?utm_source=chatgpt.com "Building Efficient SCD2 Pipelines with Apache Iceberg and ..."))

No mundo Delta Lake / Delta Table (caso paralelo ao uso de Iceberg), o artigo do IterationInsights mostra passo a passo como implementar o SCD2: calcular hash, detectar modificações, fechar versões antigas, inserir novas, lidar com deleções lógicas. ([Iteration Insights](https://iterationinsights.com/article/how-to-implement-slowly-changing-dimensions-scd-type-2-using-delta-table/?utm_source=chatgpt.com "How to Implement SCD Type 2 Using Delta Table"))

---
### Referências

**Kimball Group (autoria de Ralph Kimball e Margy Ross)**

1. Kimball, R. — _Kimball Dimensional Modeling Techniques_ (2013).  
    [https://www.kimballgroup.com/wp-content/uploads/2013/08/2013.09-Kimball-Dimensional-Modeling-Techniques11.pdf](https://www.kimballgroup.com/wp-content/uploads/2013/08/2013.09-Kimball-Dimensional-Modeling-Techniques11.pdf)
    
2. Kimball, R. — _Slowly Changing Dimensions, Part 2_ (2008).  
    [https://www.kimballgroup.com/2008/09/slowly-changing-dimensions-part-2/](https://www.kimballgroup.com/2008/09/slowly-changing-dimensions-part-2/)
    
3. Kimball, R. — _Design Tip #8: Perfectly Partitioning History with the Type 2 Slowly Changing Dimension_ (2002).  
    [https://www.kimballgroup.com/wp-content/uploads/2012/05/DT8Perfectly.pdf](https://www.kimballgroup.com/wp-content/uploads/2012/05/DT8Perfectly.pdf)
    
4. Kimball, R. — _Design Tip #100: Keep Your Keys Simple_ (2012).  
    [https://www.kimballgroup.com/wp-content/uploads/2012/05/DT100KeepYourKeysSimple.pdf](https://www.kimballgroup.com/wp-content/uploads/2012/05/DT100KeepYourKeysSimple.pdf)
    
5. Kimball Group — _Dimension Surrogate Key_ (técnicas de modelagem dimensional).  
    [https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/kimball-techniques/dimensional-modeling-techniques/dimension-surrogate-key/](https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/kimball-techniques/dimensional-modeling-techniques/dimension-surrogate-key/)
    

---

**Microsoft Learn**  
6. _Surrogate Keys in Data Warehousing_.  
[https://learn.microsoft.com/en-us/sql/ssas/data-mining/surrogate-keys-in-data-warehousing](https://learn.microsoft.com/en-us/sql/ssas/data-mining/surrogate-keys-in-data-warehousing)

---

**Iteration Insights**  
7. _How to Implement Slowly Changing Dimensions (SCD) Type 2 Using Delta Table_.  
[https://iterationinsights.com/article/how-to-implement-slowly-changing-dimensions-scd-type-2-using-delta-table/](https://iterationinsights.com/article/how-to-implement-slowly-changing-dimensions-scd-type-2-using-delta-table/)

---

**Amazon Web Services (AWS Big Data Blog)**  
8. _Implement Historical Record Lookup and Slowly Changing Dimensions Type 2 Using Apache Iceberg_.  
[https://aws.amazon.com/blogs/big-data/implement-historical-record-lookup-and-slowly-changing-dimensions-type-2-using-apache-iceberg/](https://aws.amazon.com/blogs/big-data/implement-historical-record-lookup-and-slowly-changing-dimensions-type-2-using-apache-iceberg/)

---

**Databricks Documentation**  
9. _Implementing SCD Type 2 in Delta Live Tables_.  
[https://docs.databricks.com/en/delta-live-tables/delta-live-tables-scd.html](https://docs.databricks.com/en/delta-live-tables/delta-live-tables-scd.html)

---

**Medium (UseFuseFI)**  
10. _Building Efficient SCD2 Pipelines with Apache Iceberg and Spark SQL: A Complete Guide to Time-Based Dimensions_ (tradução livre).  
[https://medium.com/@usefusefi/building-efficient-scd2-pipelines-with-apache-iceberg-and-spark-sql-a-complete-guide-to-time-based-8e5324de0d04](https://medium.com/@usefusefi/building-efficient-scd2-pipelines-with-apache-iceberg-and-spark-sql-a-complete-guide-to-time-based-8e5324de0d04)

---

> **Nota:** Alguns trechos incluídos no corpo principal foram traduzidos livremente dos textos originais em inglês para fins didáticos e de contextualização técnica.