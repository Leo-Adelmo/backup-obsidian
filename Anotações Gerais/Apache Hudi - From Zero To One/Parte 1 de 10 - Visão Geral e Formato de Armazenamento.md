link: https://blog.datumagic.ai/p/apache-hudi-from-zero-to-one-110

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

## Visão Geral da Série Completa (10 Partes)

Com base na pesquisa realizada, a série "Apache Hudi: From Zero to One" é composta por 10 partes que cobrem:

1. **Parte 1/10** - Visão geral e formato de armazenamento (traduzida acima)
2. **Parte 2/10** - Operações de leitura e tipos de consulta com Apache Spark SQL
3. **Parte 3/10** - Operações de escrita e tipos de escrita
4. **Parte 4/10** - Indexação e tipos de índice
5. **Parte 5/10** - Serviços de tabela: compactação, limpeza e indexação
6. **Parte 6/10** - Clustering e otimização de layout
7. **Parte 7/10** - Controle de concorrência
8. **Parte 8/10** - Processamento incremental e Change Data Capture (CDC)
9. **Parte 9/10** - Hudi Streamer - ferramenta de ingestão
10. **Parte 10/10** - Conclusão da série

A série foi publicada entre agosto de 2023 e abril de 2024, e todos os conceitos são baseados na versão 0.14.0 do Apache Hudi.

**Nota:** Para acessar e traduzir as demais partes da série, seria necessário obter os links específicos ou que você forneça o conteúdo das outras partes individualmente, pois algumas das URLs não estão acessíveis através dos métodos de busca utilizados.