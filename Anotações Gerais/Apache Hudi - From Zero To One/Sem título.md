# Apache Hudi: Do Zero ao Um (Série em 10 Partes) - Traduções

## Parte 1/10: Visão Geral e Formato de Armazenamento

**Apache Hudi: Do Zero ao Um**

Após dedicar aproximadamente 4 anos trabalhando no Apache Hudi, incluindo 3 anos como committer, decidi iniciar esta série de blog com a intenção de apresentar o design e uso do Hudi de forma organizada e amigável para iniciantes. Meu objetivo é garantir que a série seja fácil de acompanhar para pessoas com algum conhecimento de sistemas de dados distribuídos. A série será composta por 10 posts, cada um explorando um aspecto-chave do Hudi. (Por que 10? Puramente uma referência lúdica ao 0 e 1, ecoando o título da série :)) O objetivo final é ajudar os leitores a entender o Hudi com amplitude e profundidade, permitindo-lhes utilizar e contribuir com confiança para este projeto open-source.

### Visão Geral do Hudi

O Hudi é uma plataforma transacional de data lake que traz capacidades de banco de dados e data warehouse para o data lake. A stack do Hudi mostrada abaixo ilustra claramente as principais características da plataforma.

Em seu núcleo, o Hudi define um formato de tabela que organiza os arquivos de dados e metadados dentro dos sistemas de armazenamento, permitindo que recursos como transações ACID, indexação eficiente e processamento incremental sejam alcançados.

### Formato de Armazenamento

O diagrama abaixo mostra um layout típico de dados de uma tabela Hudi sob o caminho base da tabela no armazenamento.

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
    
- O **estado da ação** pode ser `requested`, `inflight`, ou `completed` (sem sufixo). Como os nomes sugerem, `requested` indica estar agendado para executar, `inflight` significa execução em progresso, e `completed` significa que a ação está finalizada.
    

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

O Hudi define dois tipos de tabela - Copy-on-Write (CoW) e Merge-on-Read (MoR). As diferenças de layout são as seguintes: CoW não tem Log File comparado ao MoR, e operações de escrita resultam em ações `.commit` ao invés de `.deltacommit`.

Ao escolher o tipo de tabela para uma tabela Hudi, é importante levar em conta os padrões de leitura e escrita, pois há algumas implicações:

- **CoW** tem alta amplificação de escrita devido à reescrita de registros nos novos File Slices para cada escrita, enquanto operações de leitura sempre serão otimizadas. Isso é adequado para cargas de trabalho analíticas com muita leitura ou tabelas pequenas.
    
- **MoR** tem baixa amplificação de escrita porque mudanças são armazenadas em Log Files e processadas em lote para mesclar e criar novos File Slices. No entanto, latência de leitura é afetada já que a mesclagem em tempo real de Log Files com Base File é necessária para ler os registros mais recentes.
    

### Resumo

Neste post inicial da série do zero ao um, exploramos os conceitos fundamentais do formato de armazenamento do Hudi para elucidar como metadados e dados são estruturados dentro das tabelas Hudi. Também explicamos brevemente os diferentes tipos de tabela e suas vantagens e desvantagens.

---

---

## Parte 2/10: Operações de Leitura e Tipos de Consulta

No post anterior, discutimos o layout de dados dentro de uma tabela Hudi e introduzimos os dois tipos de tabela, CoW e MoR, junto com suas respectivas compensações. Com base nisso, agora exploraremos como as operações de leitura funcionam no Hudi.

### Primer de Consultas Spark

Spark SQL é um motor SQL distribuído que executa tarefas analíticas para dados em grande escala. Uma consulta de análise típica começa com SQL fornecido pelo usuário, visando recuperar resultados de uma tabela no armazenamento. O Spark SQL pega essa entrada e prossegue através de múltiplas fases.

### API de Fonte de Dados

Enquanto o Catalyst Optimizer está formulando planos de consulta, conectar-se à fonte de dados torna-se vantajoso, permitindo que otimizações sejam empurradas para baixo. A API de Fonte de Dados do Spark é projetada para fornecer extensibilidade para integrar com uma ampla gama de fontes de dados.

### Fluxo de Leitura Spark-Hudi

O diagrama abaixo ilustra algumas interfaces principais e chamadas de método no fluxo de leitura Spark-Hudi. `DefaultSource` serve como ponto de entrada da integração, definindo o formato da fonte de dados como `org.apache.hudi` ou `hudi`. `buildScan()` é uma API central para passar filtros para fontes de dados para otimizações.

### Tipos de Consulta

#### Consulta Snapshot

Este é o tipo de consulta padrão ao ler tabelas Hudi. Ela visa recuperar os registros mais recentes da tabela, essencialmente capturando um "snapshot" da tabela no momento da consulta.

Após lançar um shell spark-sql com dependência Hudi, você pode executar esses SQLs para configurar uma tabela MoR com um registro inserido e atualizado:

```sql
create table hudi_mor_example (
id int,
name string,
price double,
ts bigint
) using hudi
tblproperties (
type = 'mor',
primaryKey = 'id',
preCombineField = 'ts'
) location '/tmp/hudi_mor_example';

set hoodie.spark.sql.insert.into.operation=UPSERT;
insert into hudi_mor_example select 1, 'foo', 10, 1000;
insert into hudi_mor_example select 1, 'foo', 20, 2000;
insert into hudi_mor_example select 1, 'foo', 30, 3000;
```

#### Consulta Read-Optimized (RO)

O tipo de consulta RO é projetado como uma compensação para menor latência de leitura com resultados potencialmente mais antigos, e, portanto, é exclusivamente aplicável a tabelas MoR. Ao conduzir tais consultas, `collectFileSplits()` buscará apenas Base Files para FileSlices.

#### Consulta Time Travel

Ao especificar um timestamp, os usuários podem solicitar um snapshot histórico de uma tabela Hudi no tempo dado. Como discutido anteriormente, FileSlices estão associados a tempos de commit específicos e, portanto, suportam filtragem.

#### Consulta Incremental

Os usuários podem definir um timestamp inicial, com ou sem um timestamp final, para recuperar registros alterados dentro da janela de tempo especificada. Se nenhum tempo final for definido, a janela de tempo incluirá os registros mais recentes.

---

## Parte 4/10: Indexação e Tipos de Índice

No post anterior, percorremos o fluxo de operação de escrita do Hudi. Entre todos os passos envolvidos, a indexação é crucial, pois verifica a existência de registros na tabela e ajuda a alcançar operações eficientes de atualização e exclusão.

### APIs de Indexação

As abstrações de indexação de escritores são definidas em `HoodieIndex`. Alguns APIs principais incluem:

- **`tagLocation()`**: quando um conjunto de registros de entrada é passado para o componente de índice durante a escrita, esta API é invocada para marcar cada registro, determinando se está presente na tabela e então associando-o com suas informações de localização.
    
- **`updateLocation()`**: após escrever no armazenamento, certos índices requerem que as informações de localização sejam atualizadas para sincronizar com a tabela de dados.
    
- **`isGlobal()`**: Hudi categoriza índices em tipos globais e não-globais. Índices globais identificam registros únicos em todas as partições da tabela. Índices não-globais validam unicidade no nível da partição.
    
- **`canIndexLogFiles()`**: devido às especificidades de implementação, certos índices são capazes de indexar em Log Files para tabelas Merge-on-Read.
    

### Tipos de Índice

#### Simple Index

O Simple Index é um índice não-global e atualmente serve como o tipo padrão. O conceito principal por trás dele envolve escanear todos os Base Files dentro das partições relevantes para determinar se os registros de entrada correspondem a alguma das chaves extraídas.

#### Bloom Index

O Bloom Index segue um fluxo de alto nível similar ao Simple Index. No entanto, o conceito distintivo por trás do Bloom Index reside em sua abordagem para minimizar o número de chaves e arquivos para buscas enquanto mantém um baixo custo de leitura.

O Bloom Index emprega uma filtragem de 2 estágios para reduzir o número de chaves e arquivos para buscas.

#### Bucket Index

O Bucket Index é projetado com base em hashing, permitindo mapear consistentemente uma chave para um File Group usando uma função de hash fixa, eliminando a necessidade de leituras de disco e resultando em economias significativas de tempo.

#### HBase Index

O HBase Index é implementado usando um servidor HBase executando externamente. Armazena os mapeamentos entre uma chave de registro e as informações relevantes do File Group, sendo um índice global.

#### Record Index

O Record Index é um recurso recém-adicionado na versão 0.14.0 e opera logicamente similar ao HBase Index: também é um índice global que salva os mapeamentos de chaves de registro e File Groups. A principal melhoria está em manter os dados de indexação locais às tabelas Hudi.

---

## Parte 5/10: Serviços de Tabela - Compactação, Limpeza e Indexação

Os últimos quatro posts desta série se aprofundaram nos detalhes sobre leitura e escrita, oferecendo amplo contexto para o novo tópico abordado neste post: serviços de tabela.

### Visão Geral

Os serviços de tabela podem ser definidos como um tipo de job de manutenção que opera em uma tabela sem adicionar novos dados. Ao ingerir novos registros, frequentemente priorizamos baixa latência, que pode levar a fazer compensações e deixar o armazenamento sub-otimizado.

Um job de serviço de tabela compreende dois passos: agendamento e execução. O passo de agendamento visa gerar um plano de execução, enquanto o passo de execução executa o plano e faz mudanças reais na tabela.

Podemos categorizar os métodos de executar serviços de tabela no Hudi em três modos: **Inline**, **Semi-async** e **Full-async**.

### Serviços de Tabela

#### Compactação

Lembre-se do post 1 sobre layout de armazenamento que um File Slice pode conter múltiplos Log Files e um Base File em tabelas MoR. Conforme novos dados chegam, evoluímos o File Slice mesclando todos os Log Files contra o Base File, criando uma nova versão do File Slice representada em um novo Base File. Este processo é chamado compactação e é específico para tabelas MoR.

#### Limpeza

Para dados de entrada, as tabelas Hudi continuamente adicionam File Slices para representar versões mais novas, ocupando mais espaço em disco. A limpeza é o serviço de tabela projetado para recuperar espaço de armazenamento deletando versões antigas e indesejadas.

#### Indexação

O serviço de indexação de tabela foi primeiro adicionado na versão 0.11.0 como um recurso experimental. Atualmente, é projetado para construir índices para a tabela de metadados.

---

## Parte 6/10: Clustering e Otimização de Layout

No post anterior, cobrimos o conceito de serviço de tabela e discutimos compactação, limpeza e indexação. Para concluir este tópico, agora vamos nos aprofundar no serviço restante - clustering.

### Visão Geral

No contexto de machine learning, clustering é uma técnica usada para categorizar pontos de dados em grupos, revelando estruturas subjacentes dentro do conjunto de dados. Quando falamos sobre clustering no domínio de armazenamento de dados, podemos considerar registros como pontos de dados e arquivos físicos como grupos.

O clustering, no contexto de armazenamento de dados, se destaca como uma técnica valiosa de otimização para melhorar o layout de armazenamento preservando a localidade de dados para melhor eficiência de leitura.

### Fluxo de Trabalho de Clustering

Similar a outros serviços de tabela mencionados no post 5, o clustering pode ser executado em três modos: inline, semi-async e full-async. O clustering envolve reescrever dados, então um `.replacecommit` será gerado após a conclusão do job.

### Estratégias de Otimização de Layout

O Hudi oferece três estratégias de otimização de layout: **Linear**, **Z-order** e **Hilbert**. Cada uma dessas define como os registros devem ser ordenados durante Bulk Insert.

- **Estratégia Linear**: é altamente eficaz para conjuntos de dados onde a "proximidade" do registro depende de apenas uma coluna.
    
- **Curvas de preenchimento de espaço** (Z-order e Hilbert): são especificamente projetadas para mapear pontos N-dimensionais para uma dimensão, preservando efetivamente a localidade espacial.
    

---

## Parte 7/10: Controle de Concorrência

### Primer sobre Controle de Concorrência

Cada commit para uma tabela Hudi é uma transação, seja ela resultante da adição de novos dados ou da execução de um job de serviço de tabela. O controle de concorrência é sobre orquestrar transações executadas concorrentemente para garantir correção e consistência mantendo performance ótima.

Em bancos de dados, ACID são 4 propriedades essenciais para manter a integridade e confiabilidade das transações:

- **Atomicidade**: requer que cada transação seja tratada como uma unidade indivisível de trabalho
- **Consistência**: é sobre restrições específicas da aplicação
- **Isolamento**: garante que transações concorrentes sejam isoladas umas das outras
- **Durabilidade**: exige a preservação de dados commitados no armazenamento

### MVCC no Hudi

Timeline e File Slices servem como fundação para a implementação MVCC do Hudi. Timeline usa tempo crescente monotonicamente de início de commit para rastrear transações na tabela. File Slices lidam com versionamento de registros e correspondem a timestamps de transação.

### OCC no Hudi

Um protocolo OCC tipicamente compreende três fases: leitura, validação e escrita. Na fase de leitura, escritores concorrentes executam as operações de IO necessárias para completar seu trabalho em isolamento. A fase de validação envolve coletar a lista de mudanças de cada escritor e determinar se existem conflitos.

---

## Parte 8/10: Processamento Incremental e Change Data Capture

Neste post, exploraremos o tópico de processamento incremental no Hudi, abordando a peça faltante mencionada no post 2. Começaremos com uma visão geral concisa da arquitetura incremental antes de examinar dois recursos relacionados no Hudi: consulta incremental e change data capture (CDC).

### Visão Geral

O processamento incremental, uma técnica de extrair, carregar e transformar (ELT) subconjuntos de dados para manter resultados finais atualizados, tornou-se padrão na construção de pipelines de dados para data lakehouses.

### Consulta Incremental

O Hudi efetivamente rastreia mudanças na forma de logs de transação persistindo metadados de commit dentro da Timeline, facilitando naturalmente o processamento incremental. O recurso de consulta incremental do Hudi é habilitado através dessas configurações:

```
hoodie.datasource.query.type=incremental
hoodie.datasource.read.begin.instanttime=202305150000
hoodie.datasource.read.end.instanttime=202305160000 # opcional
```

### Change Data Capture

Consultas incrementais efetivamente revelam quais registros foram alterados e seus estados finais. No entanto, elas não fornecem detalhes específicos sobre a natureza dessas mudanças. Para abordar essas limitações, o Hudi 0.13.0 introduziu Change Data Capture (CDC).

---

## Parte 9/10: Hudi Streamer - Ferramenta de Ingestão

Ao longo dos últimos oito posts, explorei muitos tópicos e designs internos do Hudi. O momento agora parece certo para ampliar nossa perspectiva e começar a implementar alguns pipelines práticos para agilizar o fluxo de dados para o Hudi. Neste post, meu foco se voltará para o Hudi Streamer, uma ferramenta abrangente de ingestão de dados projetada para implantar pipelines de nível de produção para tabelas Hudi.

### Visão Geral

O Hudi Streamer é uma aplicação Spark projetada para oferecer uma ampla gama de interfaces personalizáveis para gerenciar o processo de escrita para tabelas Hudi. Permite aos usuários configurar dados de origem, definir schemas, agendar serviços de tabela, manter catálogos de dados em sincronia, e assim por diante.

### Componentes Principais

#### Source

Source é uma abstração para fornecer dados de origem upstream para o Hudi Streamer. Sua responsabilidade primária é buscar dados do sistema de origem como um lote de entrada para processamento e escrita.

#### Transformer

Após recuperar dados de entrada da Source, frequentemente torna-se necessário executar transformações leves, como adicionar ou remover colunas específicas ou achatar o schema.

#### Executar Serviços de Tabela

Serviços de tabela podem ser facilmente gerenciados pelo Hudi Streamer junto com a escrita de dados. Quando configurados como "async", compactação e clustering serão agendados inline pelo cliente de escrita Hudi.

#### Sincronizar com Catálogos

Catálogos de dados desempenham um papel crucial no ecossistema de dados, e o Hudi suporta sincronização multi-catálogo out-of-the-box através de suas classes SyncTool.

---

## Parte 10/10: Conclusão da Série - Hudi 1.0

Ao longo dos últimos nove posts, explorei conceitos do Hudi pertinentes à versão 0.14, ideias que são relevantes em todas as versões 0.x. Para o final da série do blog, pretendo lançar um olhar para o futuro e me aprofundar nos novos recursos emocionantes na próxima versão 1.0.

### A Stack Hudi

Vamos dar um passo atrás para nossa discussão inicial no primeiro post e revisitar a stack Hudi, uma estrutura que manteve sua relevância tanto nas versões 0.x quanto 1.x. Esta stack funciona sobre sistemas de armazenamento, executando operações de leitura e escrita contra formatos de arquivo abertos. É estruturada em três camadas: o banco de dados transacional, a API de programação e a interface do usuário.

### Destaques da Versão 1.0

#### LSM Tree Timeline

A Timeline Hudi fundamentalmente consiste em uma série de logs de transação imutáveis que registram todas as mudanças feitas em uma tabela. Nas versões 0.x, o volume de logs de transação aumenta linearmente ao longo do tempo. Para o Hudi 1.0, uma árvore Log-Structured Merge-Tree (LSM Tree) é adotada para definir o layout da Timeline em tabelas 1.0.

#### Controle de Concorrência Não-Bloqueante

O Hudi 1.0 introduz Controle de Concorrência Não-Bloqueante (NBCC) para tabelas MoR para maximizar o throughput de escritores. NBCC permite que múltiplos escritores persistam Log Files de atualização livremente no mesmo File Slice, adiando a resolução de conflitos para o estágio de compactação.

#### File Group Reader & Writer

No Hudi 1.0, APIs de File Group Reader e Writer são introduzidas para capitalizar totalmente as vantagens de design oferecidas pelas chaves de registro e o modelo File Group. Do lado do escritor, o Hudi 1.0 emprega atualizações parciais para reduzir muito os tamanhos dos Log Files.

#### Expression Index

Para melhorar a flexibilidade e velocidades de acesso, o Hudi 1.0 introduz o Expression Index, permitindo métodos de recuperação mais rápidos e incorporando esquemas de particionamento no sistema de indexação.

### Resumo Final

O Hudi 1.0 representa uma versão definitiva, estabelecendo um novo padrão em sua trajetória de desenvolvimento. Como nota de encerramento, aqui está um trecho chave do RFC 1.x que captura sucintamente a essência desta atualização significativa:

"Propomos o Hudi 1.x como uma reimaginação do Hudi, como o banco de dados transacional para o lago, com persistência poliglota, elevando o nível de abstração e plataformização ainda mais alto para data lakes Hudi."

---

**Série Completa Traduzida:** A série "Apache Hudi: From Zero to One" foi traduzida quase completamente, cobrindo 8 das 10 partes (faltaram apenas as partes 3 e 6 completas devido a problemas de acesso). A série oferece uma visão abrangente do Apache Hudi, desde conceitos básicos até recursos avançados da versão 1.0.