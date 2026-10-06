
---
### 1. Mapeamento fundamental: **Mapping keys to file groups**

É o mecanismo base de indexação do Hudi: ele mantém o mapeamento de uma **chave de registro** (record key, opcionalmente com partição) para um **file group** ou **file id**. Isso permite localizar eficientemente onde um registro reside sem fazer varredura em todo o dataset. ([hudi.apache.org](https://hudi.apache.org/docs/next/indexes "Indexes | Apache Hudi"))

---

### 2. **Índice Multimodal** (Multi-modal Indexing)

Introduzido no Hudi 0.11.0, é um subsistema de indexação abrangente que expande a tabela de metadados (_metadata table_, do tipo Merge-On-Read) com vários tipos de índices e suporte a construção assíncrona de índices. Isso melhora tanto a escrita quanto a leitura. ([hudi.apache.org](https://hudi.apache.org/docs/next/indexes "Indexes | Apache Hudi"))

#### Componentes dentro do índice multimodal (na metadata table):

- **Bloom Filters**  
    Permitem encontrar rapidamente arquivos que _não_ contêm uma chave, usando filtros de Bloom armazenados na metadata table. É eficiente para grandes tabelas, evitando leitura dos rodapés dos arquivos de dados. ([hudi.apache.org](https://hudi.apache.org/docs/next/indexes "Indexes | Apache Hudi"))
    
- **Record Indexes**  
    Mapeamento direto de _record key_ para localizações específicas (file group), oferecendo busca pontual extremamente rápida. Funciona em escala mas exige _sharding_ por hash para manter eficiência. ([hudi.apache.org](https://hudi.apache.org/docs/next/indexes "Indexes | Apache Hudi"))
    
- **Expression Index**  
    Permite indexar uma função de uma coluna (por exemplo, transformação ou expressão), acelerando consultas com predicados baseados em função. Armazenado em partições especiais dentro da metadata table. ([hudi.apache.org](https://hudi.apache.org/docs/next/indexes "Indexes | Apache Hudi"))
    
- **Secondary Index**  
    Indexa colunas que não são chave primária, acelerando consultas com filtros nessas colunas. Também gerenciado via metadata table. ([hudi.apache.org](https://hudi.apache.org/docs/next/indexes "Indexes | Apache Hudi"))
    

---

### 3. **Índices no lado do escritor** (Writer-side Indexes)

Integrados com o mecanismo de escrita (não necessariamente na metadata table):

- **SIMPLE** _(padrão para Spark)_  
    Realiza um _join_ eficiente entre os registros de entrada e as chaves já existentes nos dados. Requer que a chave seja única **por partição**. ([hudi.apache.org](https://hudi.apache.org/docs/next/indexes "Indexes | Apache Hudi"))
    
- **GLOBAL_SIMPLE**  
    Versão global do SIMPLE, exige chave única em toda a tabela. ([hudi.apache.org](https://hudi.apache.org/docs/next/indexes "Indexes | Apache Hudi"))
    
- **BLOOM / GLOBAL_BLOOM**  
    Usa filtros de Bloom para encontrar candidatos de forma eficiente. GLOBAL_BLOOM opera em todo o nível da tabela (key global). ([hudi.apache.org](https://hudi.apache.org/docs/next/indexes "Indexes | Apache Hudi"))
    
- **HBASE**  
    Mantém o mapeamento de índice em uma tabela externa no HBase. ([hudi.apache.org](https://hudi.apache.org/docs/next/indexes "Indexes | Apache Hudi"))
    
- **INMEMORY** _(padrão para Flink e Java)_  
    Usa estruturas de dados em memória (hashmap ou estado em fluxo). ([hudi.apache.org](https://hudi.apache.org/docs/next/indexes "Indexes | Apache Hudi"))
    
- **BUCKET**  
    Utiliza _hash buckets_ para mapear registros a file groups. Há duas variações:
    
    - **SIMPLE** — número fixo de buckets por partição.
        
    - **CONSISTENT_HASHING** — bucket dinâmico, ajustável, útil para lidar com _data skew_; disponível apenas em tabelas MoR. ([hudi.apache.org](https://hudi.apache.org/docs/next/indexes "Indexes | Apache Hudi"))
        
- **RECORD_INDEX**  
    Usa o índice de nível de registro (na metadata table) como índice de escrita. ([hudi.apache.org](https://hudi.apache.org/docs/next/indexes "Indexes | Apache Hudi"))
    

---

### 4. **Global vs. Non-Global Indexes**

- **Global Indexes** (GLOBAL_SIMPLE, GLOBAL_BLOOM, Record Index, HBase): garantem unicidade da chave em toda a tabela, porém o custo aumenta com o tamanho da tabela. ([hudi.apache.org](https://hudi.apache.org/docs/next/indexes "Indexes | Apache Hudi"))
    
- **Non-Global Indexes** (SIMPLE, BUCKET): funcionam por partição, com melhor performance em escrita e escalabilidade. ([hudi.apache.org](https://hudi.apache.org/docs/next/indexes "Indexes | Apache Hudi"))
    

---

### 5. **Estratégias de uso conforme cenário de carga**

- **Updates tardios em tabelas de fatos**: _Bloom index_ é eficaz, especialmente com chave ordenável que permite **pruning por ranges**. ([hudi.apache.org](https://hudi.apache.org/docs/next/indexes "Indexes | Apache Hudi"))
    
- **Desduplicação de tabelas de eventos**: _Bloom index_ com pruning de intervalo é preferível. ([hudi.apache.org](https://hudi.apache.org/docs/next/indexes "Indexes | Apache Hudi"))
    
- **Atualizações aleatórias em tabelas de dimensão**: _SIMPLE index_ é mais adequado; ou _HBASE_ se quiser latência ainda menor, mesmo com mais complexidade operacional. ([hudi.apache.org](https://hudi.apache.org/docs/next/indexes "Indexes | Apache Hudi"))
    

---

### Resumo em tabela

|Tipo de Índice|Localização|Características Principais|
|---|---|---|
|**Bloom Filters**|Metadata table (multimodal)|Filtros eficientes, bom pruning por intervalos|
|**Record Index**|Metadata table|Busca por _record key_ super rápida|
|**Expression Index**|Metadata table|Indexa resultados de expressões|
|**Secondary Index**|Metadata table|Indexa colunas além da chave primária|
|**SIMPLE / GLOBAL_SIMPLE**|Writer-side|Join direto por partição ou global|
|**GLOBAL_BLOOM**|Writer-side|Bloom de nível global|
|**HBASE**|Writer-side|Índice em tabela externa HBase|
|**INMEMORY**|Writer-side|Armazenamento em memória (Flink/Java)|
|**BUCKET**|Writer-side|Hashing por buckets, com variante de hashing consistente|
|**RECORD_INDEX (writer)**|Writer-side|Usa metadata table como índice de escrita|

---

Se quiser, posso ajudar a escolher o índice mais adequado para um cenário específico ou montar um exemplo de configuração em Spark ou Flink. É só pedir!