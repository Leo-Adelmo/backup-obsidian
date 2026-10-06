# Modelagem de Dados para Analista de Dados

## 1. O que é modelagem de dados

**Modelagem de dados** é a forma de organizar os dados para que eles possam ser armazenados, consultados e analisados com clareza.

Para um **analista de dados**, o objetivo principal da modelagem é facilitar respostas para perguntas de negócio, como:

- Qual foi a receita do mês?
    
- Quais produtos venderam mais?
    
- Quais clientes compraram mais?
    
- Qual região teve melhor desempenho?
    
- Como os indicadores evoluíram ao longo do tempo?
    

Em análise de dados, a modelagem precisa tornar os dados mais fáceis de entender, consultar e usar em relatórios, dashboards e indicadores.

---

# 2. Entidades

Uma **entidade** representa algo importante do negócio.

Exemplos:

|Entidade|Significado|
|---|---|
|Cliente|Pessoa ou empresa que compra|
|Produto|Item vendido|
|Pedido|Compra realizada|
|Loja|Local ou canal de venda|
|Vendedor|Pessoa responsável pela venda|

Em um banco de dados, uma entidade geralmente vira uma **tabela**.

Exemplo:

```text
Entidade: Cliente

Atributos:
- id_cliente
- nome
- cidade
- estado
- segmento
```

---

# 3. Atributos

**Atributos** são as informações que descrevem uma entidade.

Exemplo:

|Entidade|Atributos|
|---|---|
|Cliente|nome, CPF, cidade, estado|
|Produto|nome, categoria, marca, preço|
|Pedido|data, cliente, valor, status|

Na prática, os atributos viram **colunas** em uma tabela.

Exemplo:

|id_cliente|nome|cidade|estado|
|--:|---|---|---|
|1|Ana Souza|São Paulo|SP|
|2|Bruno Lima|Campinas|SP|

---

# 4. Relacionamentos

**Relacionamento** é a ligação entre duas ou mais entidades.

Exemplos:

```text
Cliente faz Pedido
Pedido contém Produto
Produto pertence a Categoria
Vendedor realiza Venda
```

Os principais tipos são:

|Tipo|Explicação|Exemplo|
|---|---|---|
|1:1|Um registro se relaciona com apenas um outro registro|Pessoa e CPF|
|1:N|Um registro se relaciona com vários registros|Cliente e Pedido|
|N:N|Muitos registros se relacionam com muitos registros|Pedido e Produto|

Exemplo de relacionamento 1:N:

```text
Um cliente pode fazer vários pedidos.
Mas cada pedido pertence a um cliente.
```

Tabela `clientes`:

|id_cliente|nome|
|--:|---|
|1|Ana|
|2|Bruno|

Tabela `pedidos`:

|id_pedido|id_cliente|data_pedido|
|--:|--:|---|
|100|1|2026-05-01|
|101|1|2026-05-03|
|102|2|2026-05-05|

---

# 5. Chaves

As chaves servem para identificar registros e conectar tabelas.

## Chave primária

A **chave primária** identifica cada linha de forma única.

Exemplo:

```text
clientes.id_cliente
```

|id_cliente|nome|
|--:|---|
|1|Ana|
|2|Bruno|

## Chave estrangeira

A **chave estrangeira** conecta uma tabela a outra.

Exemplo:

```text
pedidos.id_cliente aponta para clientes.id_cliente
```

|id_pedido|id_cliente|
|--:|--:|
|100|1|
|101|1|

Nesse caso, `id_cliente` em `pedidos` indica qual cliente fez o pedido.

---

# 6. Modelagem transacional vs modelagem analítica

Para um analista de dados, é importante diferenciar dois tipos de modelo.

|Tipo|Finalidade|Exemplo|
|---|---|---|
|Transacional|Registrar operações do sistema|Cadastro, compra, pagamento|
|Analítico|Facilitar análises e indicadores|Receita mensal, vendas por produto|

## Modelo transacional

É usado em sistemas operacionais, como ERP, CRM ou sistema de vendas.

Características:

- muitas tabelas;
    
- dados mais normalizados;
    
- foco em cadastro e operação;
    
- usado para registrar eventos do dia a dia.
    

Exemplo:

```text
clientes
pedidos
itens_pedido
produtos
pagamentos
```

## Modelo analítico

É usado para BI, dashboards, relatórios e indicadores.

Características:

- dados organizados para consulta;
    
- foco em análise;
    
- uso de fatos e dimensões;
    
- mais fácil para criar métricas.
    

Exemplo:

```text
fato_vendas
dim_cliente
dim_produto
dim_tempo
dim_loja
```

---

# 7. Tabelas dimensão

Uma **tabela dimensão** descreve o contexto da análise.

Ela responde perguntas como:

- Quem?
    
- O quê?
    
- Onde?
    
- Quando?
    
- Qual categoria?
    
- Qual região?
    

Exemplos:

|Dimensão|Descreve|
|---|---|
|`dim_cliente`|Dados do cliente|
|`dim_produto`|Dados do produto|
|`dim_tempo`|Datas, meses, anos|
|`dim_loja`|Lojas, cidades, regiões|
|`dim_vendedor`|Vendedores e equipes|

Exemplo de `dim_produto`:

|id_produto|nome_produto|categoria|marca|
|--:|---|---|---|
|10|Notebook|Eletrônicos|Dell|
|11|Mouse|Acessórios|Logitech|

A dimensão não costuma armazenar o valor da venda. Ela descreve o item analisado.

---

# 8. Tabelas fato

Uma **tabela fato** armazena eventos do negócio e valores que podem ser medidos.

Ela responde perguntas como:

- Quanto vendeu?
    
- Quantos produtos foram vendidos?
    
- Qual foi o valor total?
    
- Qual foi o desconto?
    
- Qual foi o custo?
    

Exemplos:

|Fato|Evento analisado|
|---|---|
|`fato_vendas`|Vendas realizadas|
|`fato_pagamentos`|Pagamentos feitos|
|`fato_estoque`|Movimentações de estoque|
|`fato_entregas`|Entregas realizadas|

Exemplo de `fato_vendas`:

|id_venda|id_cliente|id_produto|id_data|quantidade|valor_total|
|--:|--:|--:|--:|--:|--:|
|1|100|10|20260513|2|7000.00|
|2|101|11|20260513|1|150.00|

A tabela fato normalmente contém:

- chaves para dimensões;
    
- medidas numéricas;
    
- eventos de negócio.
    

---

# 9. Diferença entre fato e dimensão

A forma mais simples de memorizar é:

```text
Dimensão descreve.
Fato mede.
```

Exemplo:

```text
A cliente Ana comprou 2 notebooks em 13/05/2026 por R$ 7.000.
```

Separando:

|Informação|Tipo|
|---|---|
|Ana|Dimensão Cliente|
|Notebook|Dimensão Produto|
|13/05/2026|Dimensão Tempo|
|2 unidades|Medida na Fato|
|R$ 7.000|Medida na Fato|

---

# 10. Medidas

**Medidas** são valores numéricos usados em análises.

Exemplos:

|Medida|Significado|
|---|---|
|quantidade|Número de itens vendidos|
|valor_total|Valor total da venda|
|desconto|Valor descontado|
|custo|Custo da venda|
|margem|Diferença entre receita e custo|

As medidas normalmente são agregadas com SQL:

```sql
SELECT
    produto.categoria,
    SUM(vendas.valor_total) AS receita_total
FROM fato_vendas vendas
JOIN dim_produto produto
    ON vendas.id_produto = produto.id_produto
GROUP BY
    produto.categoria;
```

Esse tipo de consulta responde:

```text
Qual foi a receita total por categoria de produto?
```

---

# 11. Granularidade

**Granularidade** é o nível de detalhe de uma tabela fato.

Exemplos:

|Granularidade|Significado|
|---|---|
|Uma linha por venda|Cada linha representa uma venda|
|Uma linha por item vendido|Cada linha representa um produto dentro da venda|
|Uma linha por cliente por mês|Cada linha resume um cliente em um mês|
|Uma linha por produto por dia|Cada linha resume vendas diárias por produto|

Exemplo:

Se a granularidade for **uma linha por item vendido**, uma venda com três produtos terá três linhas.

|id_pedido|id_produto|quantidade|valor|
|--:|--:|--:|--:|
|100|10|1|3500.00|
|100|11|2|300.00|
|100|12|1|500.00|

A granularidade é importante porque define como as métricas devem ser calculadas.

Erro comum:

```text
Somar valores sem entender a granularidade pode gerar métricas duplicadas.
```

---

# 12. Esquema estrela

O **esquema estrela** é um modelo muito usado em BI.

Ele possui uma tabela fato no centro e várias dimensões ao redor.

Exemplo:

```text
             dim_cliente
                  |
dim_produto — fato_vendas — dim_tempo
                  |
              dim_loja
```

Esse modelo facilita consultas como:

- vendas por cliente;
    
- vendas por produto;
    
- vendas por mês;
    
- vendas por loja;
    
- receita por categoria.
    

Exemplo de estrutura:

```text
fato_vendas
- id_cliente
- id_produto
- id_data
- id_loja
- quantidade
- valor_total

dim_cliente
- id_cliente
- nome
- cidade
- estado

dim_produto
- id_produto
- nome_produto
- categoria
- marca

dim_tempo
- id_data
- data
- mes
- ano
```

---

# 13. Exemplo prático para analista de dados

Imagine uma área comercial que precisa analisar vendas.

A pergunta principal é:

```text
Quanto vendemos por mês, produto e região?
```

Modelo possível:

## Tabela fato

```text
fato_vendas
- id_data
- id_produto
- id_cliente
- id_loja
- quantidade
- valor_total
```

## Tabelas dimensão

```text
dim_tempo
- id_data
- data
- mes
- trimestre
- ano

dim_produto
- id_produto
- nome_produto
- categoria
- marca

dim_cliente
- id_cliente
- nome_cliente
- cidade
- estado

dim_loja
- id_loja
- nome_loja
- cidade
- estado
- regiao
```

Consulta exemplo:

```sql
SELECT
    tempo.ano,
    tempo.mes,
    produto.categoria,
    loja.regiao,
    SUM(vendas.valor_total) AS receita_total
FROM fato_vendas vendas
JOIN dim_tempo tempo
    ON vendas.id_data = tempo.id_data
JOIN dim_produto produto
    ON vendas.id_produto = produto.id_produto
JOIN dim_loja loja
    ON vendas.id_loja = loja.id_loja
GROUP BY
    tempo.ano,
    tempo.mes,
    produto.categoria,
    loja.regiao;
```

Resultado esperado:

|ano|mes|categoria|regiao|receita_total|
|--:|--:|---|---|--:|
|2026|5|Eletrônicos|Sudeste|150000.00|
|2026|5|Acessórios|Sul|45000.00|

---

# 14. Boas práticas para analista de dados

|Boa prática|Por que importa|
|---|---|
|Entender a granularidade|Evita métricas duplicadas|
|Separar fatos e dimensões|Facilita análise|
|Usar nomes claros|Melhora entendimento do modelo|
|Documentar métricas|Evita divergência entre áreas|
|Validar regras de negócio|Garante que o indicador está correto|
|Conferir chaves de relacionamento|Evita joins incorretos|
|Evitar tabelas muito confusas|Facilita manutenção dos dashboards|
|Conhecer a origem dos dados|Ajuda a interpretar corretamente os números|

---

# 15. Tabela resumo geral

|Conceito|Definição simples|Exemplo|
|---|---|---|
|Entidade|Algo importante do negócio|Cliente, Produto, Pedido|
|Atributo|Característica de uma entidade|Nome, cidade, categoria|
|Relacionamento|Ligação entre entidades|Cliente faz Pedido|
|Chave primária|Identifica uma linha única|`id_cliente`|
|Chave estrangeira|Conecta uma tabela a outra|`id_cliente` em `pedidos`|
|Dimensão|Descreve o contexto da análise|`dim_cliente`, `dim_produto`|
|Fato|Registra eventos e medidas|`fato_vendas`|
|Medida|Valor numérico analisável|Receita, quantidade, custo|
|Granularidade|Nível de detalhe da fato|Uma linha por item vendido|
|Esquema estrela|Fato central ligada a dimensões|`fato_vendas` + dimensões|

---

# 16. Tabela resumo: fato vs dimensão

|Critério|Tabela Fato|Tabela Dimensão|
|---|---|---|
|Função|Medir eventos|Descrever contexto|
|Exemplo|`fato_vendas`|`dim_produto`|
|Dados principais|Valores numéricos|Atributos descritivos|
|Responde|Quanto? Quantos?|Quem? O quê? Onde? Quando?|
|Volume|Geralmente maior|Geralmente menor|
|Exemplo de coluna|`valor_total`|`categoria`|
|Uso em BI|Base dos indicadores|Filtros e segmentações|

---

# 17. Resumo final

Para um **analista de dados**, o mais importante em modelagem é entender:

```text
Entidades representam objetos do negócio.
Relacionamentos mostram como esses objetos se conectam.
Dimensões descrevem o contexto da análise.
Fatos armazenam eventos e medidas.
Medidas são os números analisados.
Granularidade define o nível de detalhe da análise.
```

Em BI e análise de dados, o modelo mais comum é o **modelo dimensional**, geralmente organizado em **esquema estrela**, com uma tabela fato central e várias tabelas dimensão ao redor.


# Tabela 1 — Resumo geral de modelagem de dados

|Conceito|Definição simples|Exemplo|Importância para análise de dados|
|---|---|---|---|
|Modelagem de dados|Organização dos dados para armazenamento, consulta e análise|Modelo de vendas|Facilita relatórios, dashboards e indicadores|
|Entidade|Algo relevante do negócio|Cliente, Produto, Pedido|Ajuda a identificar quais tabelas precisam existir|
|Atributo|Característica de uma entidade|Nome, cidade, categoria, data|Define as colunas das tabelas|
|Registro|Uma linha de dados em uma tabela|Cliente Ana Souza|Representa uma ocorrência real|
|Relacionamento|Ligação entre entidades|Cliente faz Pedido|Permite combinar dados corretamente|
|Chave primária|Campo que identifica uma linha única|`id_cliente`|Evita duplicidade e garante identificação|
|Chave estrangeira|Campo que conecta uma tabela a outra|`id_cliente` em `pedidos`|Permite realizar `JOINs` entre tabelas|
|Cardinalidade|Quantidade de relações entre entidades|1:1, 1:N, N:N|Ajuda a entender como os dados se conectam|
|Modelo analítico|Modelo voltado para análise e BI|Fatos e dimensões|Facilita consumo por analistas e dashboards|
|Modelo transacional|Modelo voltado para operação do sistema|Clientes, pedidos, pagamentos|Registra eventos do dia a dia|

---

# Tabela 2 — Entidades, atributos e registros

|Item|O que representa|Exemplo prático|
|---|---|---|
|Entidade|Objeto ou conceito do negócio|Cliente|
|Atributo|Informação que descreve a entidade|Nome, CPF, cidade, estado|
|Registro|Uma ocorrência da entidade|Ana Souza, CPF X, São Paulo, SP|
|Tabela|Estrutura que armazena a entidade|`clientes`|
|Coluna|Estrutura que armazena o atributo|`nome_cliente`|
|Linha|Estrutura que armazena o registro|Um cliente específico|

Exemplo:

|id_cliente|nome_cliente|cidade|estado|
|--:|---|---|---|
|1|Ana Souza|São Paulo|SP|
|2|Bruno Lima|Campinas|SP|

Nesse exemplo:

|Elemento|Representação|
|---|---|
|Entidade|Cliente|
|Tabela|`clientes`|
|Atributos|`id_cliente`, `nome_cliente`, `cidade`, `estado`|
|Registros|Ana Souza, Bruno Lima|

---

# Tabela 3 — Tipos de relacionamento

|Tipo de relacionamento|Significado|Exemplo|Como aparece no banco|
|---|---|---|---|
|1:1|Um registro se relaciona com apenas um outro registro|Pessoa e CPF|Uma tabela aponta para outra de forma única|
|1:N|Um registro se relaciona com vários registros|Cliente e Pedido|A chave do cliente aparece na tabela de pedidos|
|N:N|Muitos registros se relacionam com muitos registros|Pedido e Produto|Usa uma tabela intermediária|

Exemplo de relacionamento 1:N:

|Cliente|Pedido|
|---|---|
|Ana|Pedido 100|
|Ana|Pedido 101|
|Bruno|Pedido 102|

Interpretação:

|Regra|Explicação|
|---|---|
|Um cliente pode ter vários pedidos|Ana possui mais de um pedido|
|Cada pedido pertence a um cliente|Pedido 100 pertence à Ana|

---

# Tabela 4 — Chave primária vs chave estrangeira

|Tipo de chave|Função|Exemplo|Onde fica|
|---|---|---|---|
|Chave primária|Identifica uma linha de forma única|`clientes.id_cliente`|Na tabela principal|
|Chave estrangeira|Cria ligação com outra tabela|`pedidos.id_cliente`|Na tabela relacionada|

Exemplo:

|Tabela|Campo|Tipo de chave|
|---|---|---|
|`clientes`|`id_cliente`|Chave primária|
|`pedidos`|`id_pedido`|Chave primária|
|`pedidos`|`id_cliente`|Chave estrangeira|

Representação:

```text
clientes.id_cliente  ←  pedidos.id_cliente
```

---

# Tabela 5 — Modelo transacional vs modelo analítico

|Critério|Modelo transacional|Modelo analítico|
|---|---|---|
|Também chamado de|OLTP|OLAP|
|Finalidade|Registrar operações|Analisar informações|
|Uso comum|Sistemas, ERPs, CRMs|BI, dashboards, relatórios|
|Estrutura|Mais normalizada|Mais orientada a fatos e dimensões|
|Principal operação|Inserir, atualizar e excluir dados|Consultar e agregar dados|
|Exemplo de pergunta|Qual pedido foi cadastrado?|Qual foi a receita por mês?|
|Público principal|Sistemas e aplicações|Analistas, gestores e áreas de negócio|
|Exemplo de tabelas|`clientes`, `pedidos`, `itens_pedido`|`fato_vendas`, `dim_cliente`, `dim_produto`|

---

# Tabela 6 — Tabela dimensão

|Aspecto|Explicação|
|---|---|
|O que é|Tabela que descreve o contexto da análise|
|Principal função|Permitir filtros, agrupamentos e segmentações|
|Tipo de dado mais comum|Dados descritivos|
|Responde perguntas como|Quem? O quê? Onde? Quando? Qual categoria?|
|Exemplo|`dim_cliente`, `dim_produto`, `dim_tempo`, `dim_loja`|
|Uso em dashboard|Filtros, quebras, categorias e segmentações|

Exemplo:

|id_produto|nome_produto|categoria|marca|
|--:|---|---|---|
|10|Notebook|Eletrônicos|Dell|
|11|Mouse|Acessórios|Logitech|

---

# Tabela 7 — Tabela fato

|Aspecto|Explicação|
|---|---|
|O que é|Tabela que registra eventos ou ocorrências do negócio|
|Principal função|Armazenar medidas analisáveis|
|Tipo de dado mais comum|Valores numéricos e chaves de relacionamento|
|Responde perguntas como|Quanto? Quantos? Qual valor? Qual volume?|
|Exemplo|`fato_vendas`, `fato_pagamentos`, `fato_estoque`|
|Uso em dashboard|Base para indicadores e métricas|

Exemplo:

|id_venda|id_cliente|id_produto|id_data|quantidade|valor_total|
|--:|--:|--:|--:|--:|--:|
|1|100|10|20260513|2|7000.00|
|2|101|11|20260513|1|150.00|

---

# Tabela 8 — Fato vs dimensão

|Critério|Tabela fato|Tabela dimensão|
|---|---|---|
|Função|Medir eventos|Descrever contexto|
|Exemplo|`fato_vendas`|`dim_produto`|
|Conteúdo principal|Métricas e chaves|Descrições e classificações|
|Responde|Quanto? Quantos?|Quem? O quê? Onde? Quando?|
|Volume de dados|Geralmente maior|Geralmente menor|
|Exemplo de coluna|`valor_total`, `quantidade`|`categoria`, `marca`, `cidade`|
|Uso no BI|Criar indicadores|Criar filtros e agrupamentos|
|Tipo de informação|Quantitativa|Descritiva|

Regra simples:

```text
Dimensão descreve.
Fato mede.
```

---

# Tabela 9 — Medidas

|Medida|Significado|Exemplo de uso|
|---|---|---|
|`quantidade`|Número de itens vendidos|Total de produtos vendidos|
|`valor_total`|Valor total da venda|Receita total|
|`desconto`|Valor descontado|Total de descontos concedidos|
|`custo`|Custo da operação ou produto|Custo total vendido|
|`margem`|Diferença entre receita e custo|Margem por produto|
|`ticket_medio`|Valor médio por venda|Receita dividida pelo número de vendas|

Exemplo de análise:

|Pergunta|Medida usada|
|---|---|
|Quanto vendemos no mês?|`SUM(valor_total)`|
|Quantos produtos foram vendidos?|`SUM(quantidade)`|
|Qual foi o desconto total?|`SUM(desconto)`|
|Qual foi o ticket médio?|`AVG(valor_total)` ou `SUM(valor_total) / COUNT(vendas)`|

---

# Tabela 10 — Granularidade

|Granularidade|Significado|Exemplo|
|---|---|---|
|Uma linha por venda|Cada linha representa uma venda completa|Pedido 100|
|Uma linha por item vendido|Cada linha representa um produto dentro da venda|Pedido 100, Produto 10|
|Uma linha por cliente por mês|Cada linha resume um cliente em um mês|Cliente 1 em maio/2026|
|Uma linha por produto por dia|Cada linha resume um produto em uma data|Produto 10 em 13/05/2026|

Impactos da granularidade:

|Decisão|Impacto|
|---|---|
|Mais detalhada|Mais flexível, porém maior volume|
|Mais agregada|Menor volume, porém menor nível de detalhe|
|Mal definida|Pode gerar métricas duplicadas|
|Bem definida|Facilita cálculo correto dos indicadores|

---

# Tabela 11 — Esquema estrela

|Elemento|Função|
|---|---|
|Tabela fato|Fica no centro do modelo|
|Tabelas dimensão|Ficam ao redor da fato|
|Chaves|Conectam fato e dimensões|
|Medidas|Ficam na tabela fato|
|Descrições|Ficam nas dimensões|

Exemplo:

```text
             dim_cliente
                  |
dim_produto — fato_vendas — dim_tempo
                  |
              dim_loja
```

Uso típico:

|Pergunta de negócio|Tabelas usadas|
|---|---|
|Receita por mês|`fato_vendas` + `dim_tempo`|
|Receita por produto|`fato_vendas` + `dim_produto`|
|Receita por cliente|`fato_vendas` + `dim_cliente`|
|Receita por loja|`fato_vendas` + `dim_loja`|

---

# Tabela 12 — Resumo final para analista de dados

|O que o analista precisa entender|Por que isso importa|
|---|---|
|Qual é a entidade analisada|Para saber o que cada tabela representa|
|Como as tabelas se relacionam|Para fazer `JOINs` corretos|
|Qual é a chave primária|Para identificar registros únicos|
|Qual é a chave estrangeira|Para conectar tabelas|
|O que é dimensão|Para criar filtros e agrupamentos|
|O que é fato|Para calcular indicadores|
|Qual é a granularidade|Para evitar métricas duplicadas|
|Quais são as medidas|Para saber o que pode ser somado, contado ou calculado|
|Qual modelo atende o BI|Para estruturar dados voltados a dashboards|
|Qual regra de negócio define a métrica|Para evitar divergência entre relatórios|

---

# Resumo em uma frase

|Conceito central|Frase para memorizar|
|---|---|
|Entidade|Representa algo do negócio|
|Atributo|Descreve uma entidade|
|Relacionamento|Conecta entidades|
|Chave primária|Identifica uma linha|
|Chave estrangeira|Liga uma tabela a outra|
|Dimensão|Descreve o contexto|
|Fato|Registra eventos e medidas|
|Medida|Valor numérico analisável|
|Granularidade|Define o nível de detalhe|
|Esquema estrela|Organiza fatos e dimensões para análise|