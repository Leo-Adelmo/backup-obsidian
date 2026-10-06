### Visão Geral do Hudi

O Hudi é uma plataforma transacional de data lake que traz capacidades de banco de dados e data warehouse para o data lake. A stack do Hudi mostrada abaixo ilustra claramente as principais características da plataforma.

![[Pasted image 20250915185301.png]]

Em sua essência, o Hudi define um formato de tabela que organiza os arquivos de dados e metadados em sistemas de armazenamento, permitindo a implementação de recursos como transações ACID, indexação eficiente e processamento incremental.

## Formato de armazenamento

O diagrama abaixo descreve um layout de dados típico de uma tabela Hudi sob o caminho base da tabela no armazenamento.

![[Pasted image 20250915200851.png]]

Existem dois tipos principais de arquivos:

- **Arquivos de metadados** localizados em `<base_path>/.hoodie/`
- **Arquivos de dados** armazenados dentro de caminhos de partição para tabelas particionadas, ou sob o caminho base para tabelas não particionadas

#### Metadados

O arquivo `<base_path>/.hoodie/hoodie.properties` contém configurações essenciais da tabela, como nome e versão da tabela, que tanto escritores quanto leitores da tabela irão aderir e utilizar.

Junto com o `hoodie.properties`, há meta-arquivos que registram ações transacionais na tabela, formando a Timeline da tabela Hudi.

```
# exemplo de ações deltacommit na Timeline
20230827233828740.deltacommit.requested
20230827233828740.deltacommit.inflight
20230827233828740.deltacommit
```

Esses meta-arquivos seguem o padrão de nomenclatura abaixo: `<timestamp da ação>.<tipo da ação>[.<estado da ação>]`

- O **timestamp da ação** marca quando uma ação foi agendada para executar pela primeira vez, identifica unicamente uma ação na Timeline, e é monotonicamente crescente entre diferentes ações em uma Timeline.
    
- O **tipo da ação** mostra que tipo de mudanças foram feitas pela ação. Há tipos de ações de escrita, como `commit` e `deltacommit`, que indicam novas operações de escrita (insert, update ou delete) que ocorreram na tabela. Adicionalmente, há ações de serviços de tabela, como `compaction` e `clean`, bem como ações de recuperação como `savepoint` e `restore`.
    
- O **estado da ação** pode ser `requested`, `inflight`, ou `completed` (sem sufixo).
	- `requested` indica estar agendado para executar 
	- `inflight` significa execução em progresso, e 
	- `completed` significa que a ação está finalizada.
    

Os meta-arquivos para essas ações, em formato JSON ou AVRO, contêm informações sobre as mudanças que devem ser aplicadas à tabela ou que foram aplicadas. Manter esses logs de transação torna possível recriar o estado da tabela, alcançar isolamento de snapshot, e reconciliar conflitos de escritores através de mecanismos de controle de concorrência.

#### Dados

O Hudi categoriza arquivos de dados físicos em Base File e Log File:

- **Base File** contém os registros principais armazenados em uma tabela Hudi e é otimizado para leitura.
- **Log File** contém as mudanças dos registros sobre seu Base File associado e é otimizado para escrita.

Dentro de um caminho de partição de uma tabela Hudi, um único Base File e seus Log Files associados (que podem ser nenhum ou muitos) são agrupados juntos como um **File Slice**. Múltiplos File Slices constituem um **File Group**. Tanto o File Group quanto o File Slice são conceitos lógicos projetados para englobar arquivos físicos, simplificando o acesso e manipulação para leitores e escritores.

Ao definir esses modelos, o Hudi pode:

1. Atender tanto aos requisitos de eficiência de leitura quanto de escrita. Tipicamente, Base File é configurado como formato de arquivo colunar (ex: Apache Parquet) e Log File é definido como formato baseado em linhas (ex: Apache Avro).
2. Alcançar versionamento através de ações de commit. Cada File Slice está vinculado a um timestamp específico de uma ação na Timeline, e os File Slices dentro de um File Group essencialmente rastreiam como os registros contidos evoluíram ao longo do tempo.

### Tipos de Tabela
Fontes:  [# Apache Hudi: From Zero To One (3/10)](https://blog.datumagic.ai/p/apache-hudi-from-zero-to-one-310)
		https://www.onehouse.ai/blog/comparing-apache-hudis-mor-and-cow-tables-use-cases-from-uber-and-shopee
		https://hudi.apache.org/docs/table_types/
O Hudi define dois tipos de tabela - Copy-on-Write (CoW) e Merge-on-Read (MoR). As diferenças de layout são as seguintes: CoW não tem Log File comparado ao MoR, e operações de escrita resultam em ações `.commit` ao invés de `.deltacommit`.

Ao escolher o tipo de tabela para uma tabela Hudi, é importante levar em conta os padrões de leitura e escrita, pois há algumas implicações:

- **Copy On Write (CoW):** O tipo de tabela _Copy-on-Write_ é otimizado para cargas de trabalho com predominância de leitura. Nesse modo, atualizações ou exclusões de registros acionam a criação de novos arquivos base dentro de um _file group_, e não são gerados arquivos de log. Isso garante que cada consulta leia apenas os arquivos base, oferecendo alto desempenho de leitura sem a necessidade de mesclar arquivos de log de forma dinâmica. Embora as tabelas CoW sejam ideais para varreduras/consultas OLAP, suas operações de escrita podem ser mais lentas devido ao custo de regravar arquivos base durante atualizações ou exclusões, mesmo que apenas uma pequena porcentagem de registros seja modificada em cada arquivo.

	![[Pasted image 20250918205727.png]]

- **Merge On Read (MoR):** O tipo de tabela _Merge-on-Read_ equilibra o desempenho de escrita e leitura combinando arquivos de log leves com o arquivo base por meio de compactações periódicas. Atualizações e exclusões de dados são gravadas em arquivos de log (em formatos baseados em linha, como Avro, ou em formatos colunares/arquivos base), e essas alterações nos arquivos de log são mescladas dinamicamente com os arquivos base durante a execução da consulta. Essa abordagem reduz a latência de escrita e oferece suporte à disponibilidade quase em tempo real dos dados. No entanto, o desempenho das consultas pode variar dependendo de os arquivos de log estarem ou não compactados.

	![[Pasted image 20250918205850.png]]
  
  ![[Pasted image 20250918164018.png]]
## Fluxo geral de gravação

A figura abaixo ilustra as etapas de alto nível envolvidas em uma operação de gravação no Apache Hudi no contexto de um mecanismo de execução. Cada etapa é descrita de forma sintética a seguir.  
![[Pasted image 20250915185648.png]]

### Criação do cliente de escrita

O cliente de escrita constitui o ponto de entrada para operações de escrita. A implementação é dependente do mecanismo de execução:

- Spark: `SparkRDDWriteClient`
    
- Flink: `HoodieFlinkWriteClient`
    
- Kafka Connect: `HoodieJavaWriteClient`
    

Nesta etapa, as configurações fornecidas pelo usuário são reconciliadas com as propriedades existentes da tabela e propagadas ao cliente.  
Fonte: [Apache Hudi – Writing Data](https://hudi.apache.org/docs/writing_data/)

### Transformação da entrada

Os dados de entrada são transformados em instâncias de `HoodieRecord` e submetidos à reconciliação de esquema.

Cada `HoodieRecord` é identificado por um `HoodieKey`, composto por `recordKey` e `partitionPath`, valores extraídos por meio da API `KeyGenerator`. Essa API permite configurar a lógica de geração da chave com base no esquema de entrada.  

![[Pasted image 20250915204757.png]]

Fonte: [Apache Hudi – Key Generation](https://hudi.apache.org/docs/key_generation/)

Os registros mantêm referências a:

- **currentLocation**: localização lógica do registro já existente na tabela.
    
- **newLocation**: destino do registro a ser gravado.
    
- **data**: payload, implementado por padrão como `HoodieRecordPayload`, responsável pela lógica de merge.

### Inicialização do commit

Antes de criar um novo commit, o cliente verifica a existência de operações incompletas na Timeline. Se necessário, executa rollback. Em seguida, registra na Timeline uma ação de commit do tipo **requested**.  
Fonte: [Apache Hudi – Timeline](https://hudi.apache.org/docs/timeline/)

### Preparação dos registros

Os registros podem ser submetidos a:

- **Deduplicação**: mesclagem de registros com a mesma chave.
    
- **Indexação**: atribuição de `currentLocation` quando o registro já existe.
    

O índice é o mecanismo que localiza arquivos físicos para registros. O Hudi disponibiliza diferentes implementações (Bloom, Global, Simple, HBase, Bucket, entre outros).  
Fonte: [Apache Hudi – Indexing](https://hudi.apache.org/docs/indexing/)

### Particionamento dos registros

Os registros são distribuídos entre **FileGroups**, que definem a organização física dos dados. A alocação ocorre em buckets de inserção ou de atualização. Cada bucket corresponde a uma unidade de paralelismo do mecanismo de execução (por exemplo, uma partição RDD no Spark).  
Fonte: [Apache Hudi – Write Paths](https://hudi.apache.org/docs/write_paths/)

### Escrita no armazenamento

Nesta etapa são realizadas as operações de I/O. Arquivos de dados são criados ou atualizados via _file handles_.

Arquivos marcadores podem ser gerados no diretório `.hoodie/.temp/` para indicar o tipo de operação planejada, possibilitando rollback eficiente em caso de falhas.  
Fonte: [Apache Hudi – Rollback](https://hudi.apache.org/docs/rollbacks)

### Atualização do índice

Após a escrita, índices externos ou assíncronos (por exemplo, HBase index) podem requerer atualização imediata para assegurar consistência.

### Confirmação das mudanças

Na fase final, o cliente executa:

- validações pré-commit (se configuradas),
    
- verificação de conflitos com escritores concorrentes,
    
- persistência dos metadados de commit na Timeline,
    
- reconciliação de `WriteStatus` com os arquivos marcadores.
    

Esse processo garante atomicidade e consistência da transação de escrita.  
Fonte: [Apache Hudi – Concurrency Control](https://hudi.apache.org/docs/concurrency_control/)

## Operações de Escrita

# Upsert

fonte: [# Apache Hudi: From Zero To One (3/10)](https://blog.datumagic.ai/p/apache-hudi-from-zero-to-one-310)
Fazer upsert de dados é um cenário comum em pipelines de Lakehouse. Nesta seção, vamos nos aprofundar no fluxo de Upsert para tabela CoW em detalhes, seguido por uma breve visão geral de todas as outras operações de escrita suportadas.


![[Pasted image 20250915205400.png]]


1. O cliente de escrita inicia o commit e cria a ação "solicitada" na Timeline.
2. Registros de entrada passam pela etapa de preparação: duplicatas são mescladas, e localizações de arquivo de destino são preenchidas pelo índice. Neste ponto do processo, temos os registros exatos a serem escritos e sabemos quais deles existem na tabela, junto com suas respectivas localizações (FileGroups).
3. Registros preparados são categorizados em buckets de "atualização" e "inserção". Inicialmente, um WorkloadProfile é construído para coletar informações sobre o número de atualizações e inserções nas partições físicas relevantes. Esses dados são então serializados em uma ação "em andamento" na Timeline. Subsequentemente, baseados no WorkloadProfile, buckets são gerados para conter os registros. Para atualizações, cada FileGroup em atualização é atribuído como um bucket de atualização. No caso de inserções, a lógica de tratamento de arquivos pequenos entra em ação: qualquer BaseFile menor que um limite especificado (determinado por `hoodie.parquet.small.file.limit`) se torna candidato para acomodar as inserções, com seu FileGroup envolvente sendo designado como bucket de atualização. Se nenhum BaseFile assim existir, buckets de inserção serão alocados, e novos FileGroups serão criados para eles posteriormente.
4. Os registros organizados em buckets são então processados através de handles de escrita de arquivo para persistência real no storage. No caso de registros nos buckets de atualização, handles de "mesclagem" são usados, resultando na criação de novos FileSlices dentro dos FileGroups existentes (alcançado mesclando com dados dos FileSlices antigos). Para registros nos buckets de inserção, handles de "criação" são utilizados, levando à criação de FileGroups totalmente novos. Este processo é feito por `HoodieExecutor`s, que empregam um padrão produtor-consumidor para leitura e escrita de registros.
5. Uma vez que todos os dados foram escritos, os handles de escrita de arquivo retornam coleções de WriteStatus que contêm metadados sobre as escritas, incluindo o número de erros, o número de inserções realizadas, o tamanho total escrito em bytes, e mais. Esta informação é enviada de volta para o driver Spark para agregação. Se nenhum erro ocorreu, o cliente de escrita gerará metadados de commit e os persistirá como uma ação completa na Timeline.

Fazer upsert em uma tabela MoR segue um fluxo muito similar, com um conjunto diferente de condições para determinar os tipos de handles de escrita de arquivo usados tanto para atualizações quanto para inserções.
# Resumo fluxo de UPSERT no Apache Hudi (Feito por IA)

## O que é UPSERT?

**UPSERT** = **UP**date + In**SERT** - Uma operação que:

- **Atualiza** registros se já existem na tabela
- **Insere** registros se são novos
- É a operação mais comum em pipelines de Lakehouse

---
## Como Funciona: 5 Passos Principais

### 1️⃣ **Iniciar Transação**

```
Cliente de escrita → Cria ação "requested" na Timeline
```

- Marca o início da operação
- Prepara o sistema para receber dados

### 2️⃣ **Preparar Dados**

```
Dados brutos → Deduplicação → Indexação → Dados preparados
```

**O que acontece:**

- **Remove duplicatas** entre os dados de entrada
- **Indexação identifica** quais registros já existem na tabela
- **Resultado**: Sabemos exatamente o que inserir vs atualizar

### 3️⃣ **Organizar em Buckets**

```
Dados preparados → WorkloadProfile → Buckets de UPDATE + INSERT
```

**Buckets de UPDATE:**

- Cada FileGroup sendo atualizado vira um bucket
- Registros vão para seus FileGroups existentes

**Buckets de INSERT:**

- **Otimização smart**: Primeiro tenta usar arquivos pequenos existentes
- Se arquivo < `hoodie.parquet.small.file.limit` → vira bucket de update
- Senão → cria novos FileGroups

> 💡 **Smart File Handling**: Evita criar muitos arquivos pequenos reutilizando espaço em arquivos existentes

### 4️⃣ **Escrever no Storage**

```
Buckets → File Writing Handles → Arquivos físicos
```

**Para Updates (Merge Handles):**

- Mescla dados novos com existentes
- Cria novos FileSlices nos FileGroups existentes

**Para Inserts (Create Handles):**

- Cria FileGroups completamente novos
- Usa padrão produtor-consumidor para eficiência

### 5️⃣ **Finalizar Transação**

```
WriteStatus → Validação → Commit na Timeline → Sucesso
```

**Metadados coletados:**

- ✅ Número de registros inseridos/atualizados
- ✅ Tamanho total escrito
- ✅ Erros (se houver)

Se tudo OK → Marca ação como "completed" na Timeline

---

## Diferenças: CoW vs MoR

|Aspecto|**Copy-on-Write (CoW)**|**Merge-on-Read (MoR)**|
|---|---|---|
|**Updates**|Reescreve arquivo inteiro|Adiciona ao Log File|
|**Performance Escrita**|Mais lenta|Mais rápida|
|**Performance Leitura**|Mais rápida|Mais lenta|
|**Arquivo gerado**|`.commit`|`.deltacommit`|

---
## Por que UPSERT é Importante?

### ✅ **Vantagens**

- **Automatiza** a lógica insert vs update
- **Otimiza arquivos pequenos** automaticamente
- **Garante consistência** via transações ACID
- **Máxima eficiência** para cenários reais

### 🎯 **Casos de Uso Comuns**

- **CDC (Change Data Capture)**: Sincronizar mudanças de bancos transacionais
- **Atualizações de status**: Pedidos, usuários, produtos
- **Correção de dados**: Reprocessamento com dados corrigidos
- **Aggregações incrementais**: Métricas que são recalculadas

---
## Resumo Executivo

**UPSERT = A operação inteligente do Hudi** que:

1. **Detecta automaticamente** se é insert ou update
2. **Otimiza arquivos** evitando fragmentação
3. **Garante consistência** com transações ACID
4. **Escala eficientemente** para bilhões de registros

É a **operação padrão** para a maioria dos casos de uso em data lakes modernos!