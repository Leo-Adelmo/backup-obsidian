
---
## Resumo — Aba “Design & Concepts” (Apache Hudi)

### 1. **Apache Hudi Stack**

- Hudi traz funcionalidades típicas de banco de dados (tabelas, transações, upserts/deletes eficientes, índices avançados, ingestão, clusterização, compactação e controle de concorrência) diretamente para um data lake, mantendo os dados em formatos abertos. Isso permite cargas in-stream, pipelines batch incrementais, e uso com motores como Spark, Flink, Presto, Trino e Hive. Também inclui a implementação `Hudi-rs` em Rust com bindings para Python para ecossistemas não JVM ([hudi.apache.org](https://hudi.apache.org/docs/next/overview/?utm_source=chatgpt.com "Overview | Apache Hudi")).

### 2. **Timeline**

- A _timeline_ é o registro de todas as ações realizadas em uma tabela Hudi, armazenada em `.hoodie` sob o diretório base. Cada ação (instant) inclui tipo, timestamp (instant time), e estado (`REQUESTED`, `INFLIGHT`, `COMPLETED`). Permite time travel, processamento incremental, rastreamento de versões e é o _source of truth_ da tabela ([cwiki.apache.org](https://cwiki.apache.org/confluence/display/HUDI/Design%2BAnd%2BArchitecture?utm_source=chatgpt.com "Design And Architecture - HUDI - Apache Software Foundation"), [kcl10.com](https://kcl10.com/blog/apache-hudi-overview/?utm_source=chatgpt.com "The Lakehouse Series: Apache Hudi Overview - KC's Data & Life Notes")).


### 3. **File Layouts (Storage Layout)**

- As tabelas são organizadas em diretórios (basepath) e particionadas conforme o esquema. Dentro de cada partição, os dados são organizados em _file groups_ (identificados por UUID) e _file slices_, que contêm um _base file_ (Parquet/ORC/HFile) e arquivos de log (.log) correspondentes. Hudi aplica MVCC, realizando compactação para mesclar logs com base files e "cleaning" para remover slices antigos. Os metadados, incluindo a metadata table, estão sob `.hoodie` ([kcl10.com](https://kcl10.com/blog/apache-hudi-overview/?utm_source=chatgpt.com "The Lakehouse Series: Apache Hudi Overview - KC's Data & Life Notes")).
    

### 4. **Table & Query Types**

- **Copy-On-Write (CoW)**: ideal para cenários com leituras intensivas; atualizações ou deleções reescrevem os base files diretamente, sem usar logs, resultando em alta performance de leitura porém latência maior na escrita.
    
- **Merge-On-Read (MoR)**: balanceia escrita + leitura; escritas vão aos logs e são mescladas dinamicamente com base files durante a leitura. Ideal para latência de escrita baixa e dados quase em tempo real, ainda que a performance de leitura dependa da compactação ([kcl10.com](https://kcl10.com/blog/apache-hudi-overview/?utm_source=chatgpt.com "The Lakehouse Series: Apache Hudi Overview - KC's Data & Life Notes")).
    
- **Tipos de consulta compatíveis**:
    
    - _Snapshot Queries_: lê o snapshot mais recente.
        
    - _Time Travel Queries_: lê o estado da tabela em um instant passado.
        
    - _Read-Optimized Queries_ (somente MoR): lê exclusivamente os base files para velocidade otimizada.
        
    - _Incremental Queries_: retornam apenas os registros alterados desde um instant.
        
    - _Incremental Queries (CDC)_: oferecem fluxo de mudanças (change data capture) diretamente da tabela Hudi ([kcl10.com](https://kcl10.com/blog/apache-hudi-overview/?utm_source=chatgpt.com "The Lakehouse Series: Apache Hudi Overview - KC's Data & Life Notes")).
        

### 5. **Indexing**

- Hudi utiliza a _metadata table_ como índice para planejamentos eficientes de leitura/escrita. Oferece suporte a:
    
    - filtros Bloom nos file footers,
        
    - índices de registros,
        
    - índices baseados em expressões,
        
    - indexação externa (ex. HBase),
        
    - bucketing para localizar rapidamente os file groups com chaves específicas ([hudi.apache.org](https://hudi.apache.org/docs/faq_design_and_concepts/?utm_source=chatgpt.com "Design & Concepts | Apache Hudi"), [kcl10.com](https://kcl10.com/blog/apache-hudi-overview/?utm_source=chatgpt.com "The Lakehouse Series: Apache Hudi Overview - KC's Data & Life Notes")).
        

### 6. **Write Operations**

- Diferencia entre:
    
    - _Batch/Bulk Operations_ (ex. `insert_overwrite` para ETL tradicionais).
        
    - _Operações Incrementais_ (ex. `upsert`, `insert`, `bulk_insert`), que aplicam somente mudanças aos registros modificados, minimizando processamento e otimizando pipelines incrementais ([docs.devlive.org](https://docs.devlive.org/read/apache-hudi-en-0.15.0/Overview?utm_source=chatgpt.com "Getting Started - Overview - 《Apache Hudi 0.15.0》 - Devlive 开源社区 ..."), [kcl10.com](https://kcl10.com/blog/apache-hudi-overview/?utm_source=chatgpt.com "The Lakehouse Series: Apache Hudi Overview - KC's Data & Life Notes")).
        

### 7. **Key Generation** _(geração de chaves)_

- Embora não descrito detalhadamente nas fontes selecionadas, refere-se à lógica para construir a chave primária (record-key) — frequentemente composta de valores como ID e caminho de partição — usada para indexação e identificação única de registros.
    

### 8. **Record Payload**

- Também não detalhado especificamente nas fontes acima, mas tradicionalmente refere-se à estrutura de dados de cada registro, incluindo metadados internos como chave de registro, timestamps e campos associados que influenciam fusão e versionamento.
    

### 9. **Schema Evolution**

- Não detalhado nas fontes acessadas, normalmente significa a habilidade de uma tabela Hudi evoluir seu esquema conforme novas colunas são adicionadas, renomeadas ou modificadas, mantendo compatibilidade com dados históricos.
    

### 10. **Metadata Table**

- A _metadata table_ fornece índice para acelerar operações, como listagem de arquivos, pruning, e suporte a operações como query planning e evolução de esquema — além de ser fundamental para variadas otimizações internas ([kcl10.com](https://kcl10.com/blog/apache-hudi-overview/?utm_source=chatgpt.com "The Lakehouse Series: Apache Hudi Overview - KC's Data & Life Notes"), [hudi.apache.org](https://hudi.apache.org/docs/faq_design_and_concepts/?utm_source=chatgpt.com "Design & Concepts | Apache Hudi")).
    

### 11. **Concurrency Control**

- Hudi oferece isolamento de snapshot entre escritores, leitores e serviços de tabela. Implementa:
    
    - commit atômico via _instant time_,
        
    - controle de concorrência otimista (OCC) entre escritores,
        
    - MVCC lock-free para writers e table services,
        
    - controle não bloqueante (NBCC) permitindo múltiplos escritores sem deadlocks. Também permite uso de `InProcessLockProvider` para evitar overhead quando há apenas um escritor e serviços embutidos ([hudi.apache.org](https://hudi.apache.org/docs/faq_design_and_concepts/?utm_source=chatgpt.com "Design & Concepts | Apache Hudi")).
        

---

### Quadro Resumo (Todos os Itens)

|Item|Resumo|
|---|---|
|Apache Hudi Stack|Lakehouse transacional, transações, upsert/delete, índices, ingestão, otimizações, `Hudi-rs`|
|Timeline|Registro de ações e instantes; suporte a time travel e incremental|
|File Layouts|File groups/slices com MVCC, base/log files, compactação e limpeza, metadados `.hoodie`|
|Table & Query Types|CoW vs MoR; snapshots, incremental, CDC, read-optimized, time-travel queries|
|Indexing|Metadata table, bloom, registro, expressões, bucketing, indexação externa|
|Write Operations|Operações batch vs incrementais (`upsert`, `insert`, `bulk_insert`)|
|Key Generation|Construção de chaves primárias (record-key + partição)|
|Record Payload|Estrutura de arquivos por registro com metadados internos|
|Schema Evolution|Evolução do esquema da tabela mantendo compatibilidade|
|Metadata Table|Índice avançado para otimizar operações e leitura|
|Concurrency Control|Isolamento de snapshot, OCC, MVCC lock-free, NBCC, provedor de lock inline|

---

Se quiser, posso ampliar explicações práticas para itens como **Key Generation**, **Record Payload** e **Schema Evolution**, aplicando ao uso real do Hudi ou integrando com casos práticos do ecossistema. É só me dizer!