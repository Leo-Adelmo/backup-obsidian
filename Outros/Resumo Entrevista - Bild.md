
---
# 1. SQL analítico

SQL analítico é o uso de SQL para transformar dados em informações úteis para análise. Ele é usado para criar consultas, métricas, relatórios, views e bases que alimentam dashboards.

Na prática, não é apenas “buscar dados no banco”. É usar SQL para responder perguntas de negócio, como:

> Qual foi a receita por mês?  
> Quais clientes compraram mais?  
> Qual produto teve maior crescimento?  
> Como o indicador mudou em relação ao mês anterior?

---

## 1.1 JOIN

`JOIN` é usado para combinar dados de duas ou mais tabelas.

Em bases de dados reais, as informações geralmente ficam separadas. Por exemplo:

Uma tabela de vendas pode ter:

|id_venda|id_cliente|valor|
|---|--:|--:|
|1|10|500|

E uma tabela de clientes pode ter:

|id_cliente|nome|cidade|
|--:|---|---|
|10|João|São Paulo|

Para montar uma análise com venda, nome do cliente e cidade, você precisa juntar as tabelas usando `JOIN`.

Exemplo:

```sql
SELECT
    vendas.id_venda,
    clientes.nome,
    clientes.cidade,
    vendas.valor
FROM vendas
JOIN clientes
    ON vendas.id_cliente = clientes.id_cliente;
```

### O que falar na entrevista

> JOIN é usado para combinar dados de tabelas diferentes, geralmente usando uma chave em comum. Em análises, ele é essencial para enriquecer uma tabela principal com informações descritivas, como cliente, produto, região ou categoria.

---

## 1.2 GROUP BY

`GROUP BY` é usado para agrupar dados e calcular métricas.

Por exemplo, imagine uma tabela com várias vendas. Cada linha representa uma venda individual. Se você quiser saber o total vendido por cliente, precisa agrupar as vendas por cliente.

Exemplo:

```sql
SELECT
    id_cliente,
    SUM(valor) AS total_vendido
FROM vendas
GROUP BY id_cliente;
```

Resultado esperado:

|id_cliente|total_vendido|
|--:|--:|
|10|1500|
|20|900|

O `GROUP BY` é muito usado com funções agregadoras, como:

|Função|Uso|
|---|---|
|`SUM()`|Soma valores|
|`COUNT()`|Conta registros|
|`AVG()`|Calcula média|
|`MIN()`|Menor valor|
|`MAX()`|Maior valor|

### O que falar na entrevista

> GROUP BY é usado para criar agregações. Ele permite transformar dados detalhados em métricas resumidas, como receita por mês, quantidade por produto ou média por região.

---

## 1.3 CASE WHEN

`CASE WHEN` é usado para criar regras condicionais dentro da consulta SQL.

Ele funciona de forma parecida com um `if/else`.

Exemplo:

```sql
SELECT
    id_venda,
    valor,
    CASE
        WHEN valor >= 1000 THEN 'Alto valor'
        WHEN valor >= 500 THEN 'Médio valor'
        ELSE 'Baixo valor'
    END AS categoria_venda
FROM vendas;
```

Nesse caso, cada venda será classificada conforme o valor.

### Casos comuns de uso

|Uso|Exemplo|
|---|---|
|Classificar registros|Alto, médio ou baixo valor|
|Criar faixas|Faixa etária, faixa de receita|
|Tratar regras de negócio|Cliente ativo ou inativo|
|Criar indicadores|Venda aprovada ou recusada|

### O que falar na entrevista

> CASE WHEN permite aplicar regras de negócio dentro da consulta SQL. Ele é muito útil para classificar dados, criar categorias e transformar regras operacionais em campos analíticos.

---

## 1.4 CTE / WITH

CTE significa **Common Table Expression**. Em SQL, ela é criada usando `WITH`.

Ela serve para organizar consultas complexas em blocos menores e mais legíveis.

Exemplo sem CTE pode ficar confuso. Com CTE, a consulta fica mais organizada:

```sql
WITH vendas_por_cliente AS (
    SELECT
        id_cliente,
        SUM(valor) AS total_vendido
    FROM vendas
    GROUP BY id_cliente
)
SELECT
    id_cliente,
    total_vendido
FROM vendas_por_cliente
WHERE total_vendido > 1000;
```

Primeiro, a CTE calcula o total vendido por cliente. Depois, a consulta principal filtra apenas clientes com mais de 1000 em vendas.

### Por que usar CTE?

|Benefício|Explicação|
|---|---|
|Organização|Divide a consulta em etapas|
|Legibilidade|Facilita entender a lógica|
|Manutenção|Facilita alterar partes da query|
|Reutilização lógica|Permite usar resultados intermediários|

### O que falar na entrevista

> CTE é uma forma de organizar consultas complexas em etapas. Eu gosto de usar CTEs para deixar a lógica mais clara, principalmente quando preciso calcular métricas intermediárias antes do resultado final.

---

## 1.5 Window Functions

Window Functions são funções analíticas que calculam valores considerando um conjunto de linhas relacionadas, sem necessariamente agrupar tudo em uma única linha.

A diferença principal é:

`GROUP BY` reduz várias linhas em uma linha agregada.

`Window Function` mantém as linhas originais e adiciona cálculos analíticos.

Exemplo: calcular ranking de vendas por cliente.

```sql
SELECT
    id_cliente,
    valor,
    RANK() OVER (ORDER BY valor DESC) AS ranking_venda
FROM vendas;
```

Exemplo: comparar venda atual com venda anterior.

```sql
SELECT
    data_venda,
    valor,
    LAG(valor) OVER (ORDER BY data_venda) AS valor_anterior
FROM vendas;
```

### Funções comuns

|Função|Uso|
|---|---|
|`ROW_NUMBER()`|Numera linhas|
|`RANK()`|Cria ranking com empates|
|`LAG()`|Busca valor da linha anterior|
|`LEAD()`|Busca valor da próxima linha|
|`SUM() OVER()`|Soma acumulada|
|`AVG() OVER()`|Média móvel ou média por grupo|

### O que falar na entrevista

> Window Functions são muito úteis para análises mais avançadas, como rankings, acumulados, comparações com períodos anteriores e cálculos por grupo sem perder o detalhe das linhas originais.

---

## 1.6 Views

Uma `view` é uma consulta SQL salva no banco de dados.

Ela funciona como uma tabela lógica. A view não precisa necessariamente armazenar os dados fisicamente; ela guarda a lógica da consulta.

Exemplo:

```sql
CREATE VIEW vw_vendas_por_cliente AS
SELECT
    clientes.nome,
    clientes.cidade,
    SUM(vendas.valor) AS total_vendido
FROM vendas
JOIN clientes
    ON vendas.id_cliente = clientes.id_cliente
GROUP BY
    clientes.nome,
    clientes.cidade;
```

Depois, em vez de escrever a consulta inteira novamente, você pode usar:

```sql
SELECT *
FROM vw_vendas_por_cliente;
```

### Por que views são importantes para BI?

|Motivo|Explicação|
|---|---|
|Reutilização|A mesma lógica pode alimentar vários dashboards|
|Padronização|Evita cálculos diferentes para a mesma métrica|
|Simplicidade|Usuário ou BI consome uma estrutura pronta|
|Governança|Centraliza regras de negócio|

### O que falar na entrevista

> Views ajudam a organizar e padronizar a camada analítica. Elas permitem disponibilizar dados já tratados e modelados para consumo em dashboards, reduzindo duplicidade de lógica e inconsistência entre indicadores.

---

## 1.7 Performance em SQL

Performance em SQL significa escrever consultas que retornem os dados corretamente e em tempo adequado.

Uma query pode ficar lenta por vários motivos:

|Problema|Exemplo|
|---|---|
|Trazer colunas demais|Usar `SELECT *` sem necessidade|
|Trazer linhas demais|Não aplicar filtros|
|Joins mal feitos|Juntar tabelas sem chave adequada|
|Agregações pesadas|Calcular tudo em cima de milhões de linhas|
|Falta de índices/partições|Banco precisa escanear dados demais|
|Cálculos complexos|Muitos campos calculados em tempo de consulta|

Boas práticas:

```sql
SELECT
    id_cliente,
    SUM(valor) AS total_vendido
FROM vendas
WHERE data_venda >= '2026-01-01'
GROUP BY id_cliente;
```

Essa query é melhor do que trazer todos os dados e filtrar depois no BI.

### O que falar na entrevista

> Performance envolve reduzir volume desnecessário, aplicar filtros corretamente, selecionar apenas colunas relevantes, estruturar bem os joins e, quando possível, criar views ou tabelas agregadas para facilitar o consumo pelo BI.

---

# 2. Tableau

Tableau é uma ferramenta de BI usada para criar dashboards e visualizações de dados.

Ele permite conectar dados de diferentes fontes, criar gráficos, filtros, indicadores e painéis interativos. A função principal do Tableau é transformar dados em visualizações que ajudem a área de negócio a interpretar informações.

---

## 2.1 Dashboard

Dashboard é um painel visual que reúne gráficos, indicadores, filtros e tabelas em uma única tela.

Exemplo de dashboard comercial:

|Elemento|Exemplo|
|---|---|
|KPI principal|Receita total|
|Gráfico de linha|Receita por mês|
|Gráfico de barras|Receita por produto|
|Filtro|Região, período, vendedor|
|Tabela|Detalhamento por cliente|

Um dashboard deve responder rapidamente a uma pergunta de negócio.

### O que falar na entrevista

> Dashboard é um painel que reúne indicadores e visualizações para facilitar a análise e tomada de decisão. Um bom dashboard precisa ser claro, objetivo e orientado ao problema de negócio.

---

## 2.2 Worksheet

Worksheet é a área do Tableau onde cada gráfico é construído.

Em geral, você cria uma visualização por worksheet. Depois, combina várias worksheets em um dashboard.

Exemplo:

|Worksheet|Conteúdo|
|---|---|
|Worksheet 1|Receita mensal|
|Worksheet 2|Vendas por produto|
|Worksheet 3|Receita por região|
|Worksheet 4|Tabela de clientes|

Depois, essas visualizações são organizadas no dashboard.

### O que falar na entrevista

> Worksheet é o espaço onde cada visualização individual é criada. Depois, essas visualizações podem ser combinadas em dashboards para formar uma análise mais completa.

---

## 2.3 Data Source

Data Source é a fonte de dados conectada ao Tableau.

Pode ser:

|Tipo|Exemplo|
|---|---|
|Banco relacional|PostgreSQL, SQL Server, Oracle|
|Arquivo|Excel, CSV|
|Data Lake|Parquet, tabelas externas|
|Plataforma analítica|Dremio, BigQuery, Snowflake|
|View SQL|View criada para BI|

A qualidade do Data Source impacta diretamente a qualidade do dashboard.

### O que falar na entrevista

> Data Source é a origem dos dados utilizada pelo Tableau. Quanto mais bem modelada e tratada estiver essa fonte, mais simples e confiável será a construção do dashboard.

---

## 2.4 Extract

Extract é uma cópia otimizada dos dados dentro do Tableau.

Em vez de consultar o banco de dados toda vez que o usuário abre o dashboard, o Tableau pode usar uma cópia local otimizada para melhorar performance.

### Quando usar Extract?

|Situação|Motivo|
|---|---|
|Fonte lenta|Reduz dependência do banco|
|Muitos dados|Tableau otimiza a consulta|
|Dashboard muito acessado|Melhora tempo de resposta|
|Dados atualizados por agenda|Extract pode ser atualizado periodicamente|

### O que falar na entrevista

> Extract é uma cópia otimizada dos dados no Tableau. Ele pode melhorar performance porque evita consultar a fonte original em tempo real a cada interação do usuário.

---

## 2.5 Live Connection

Live Connection é quando o Tableau consulta diretamente a fonte de dados em tempo real.

Nesse caso, o dashboard não usa uma cópia local. Ele envia consultas para o banco ou plataforma toda vez que precisa atualizar a visualização.

### Diferença entre Extract e Live Connection

|Conceito|Característica|
|---|---|
|Extract|Usa cópia otimizada dos dados|
|Live Connection|Consulta direto na fonte|
|Extract|Melhor performance em muitos casos|
|Live Connection|Dados mais atualizados|
|Extract|Depende de agenda de atualização|
|Live Connection|Depende da performance da fonte|

### O que falar na entrevista

> Live Connection consulta diretamente a fonte de dados, enquanto Extract usa uma cópia otimizada. A escolha depende da necessidade de atualização dos dados e da performance da fonte.

---

## 2.6 Calculated Field

Calculated Field é um campo criado dentro do Tableau a partir de uma fórmula.

Ele permite criar novas métricas ou classificações sem alterar a fonte original.

Exemplo:

```text
[Receita] / [Quantidade]
```

Isso poderia gerar um campo chamado `Ticket Médio`.

Outro exemplo:

```text
IF [Receita] >= 1000 THEN "Alto"
ELSE "Baixo"
END
```

Isso criaria uma classificação de vendas.

### O que falar na entrevista

> Calculated Field permite criar métricas e regras dentro do Tableau, como ticket médio, classificações, percentuais ou indicadores derivados dos campos originais.

---

## 2.7 Parameters

Parameters são controles dinâmicos que permitem ao usuário alterar comportamentos no dashboard.

Um parâmetro pode permitir, por exemplo:

|Uso|Exemplo|
|---|---|
|Escolher métrica|Receita, quantidade ou margem|
|Escolher período|Mês, trimestre ou ano|
|Definir limite|Mostrar produtos acima de determinado valor|
|Controlar cenário|Meta otimista, realista ou pessimista|

### O que falar na entrevista

> Parameters são controles que tornam o dashboard mais dinâmico. Eles permitem que o usuário altere critérios de análise, escolha métricas ou ajuste valores usados nos cálculos.

---

## 2.8 Filters

Filters são usados para segmentar os dados exibidos no dashboard.

Exemplos:

|Filtro|Uso|
|---|---|
|Período|Ver apenas determinado mês ou ano|
|Região|Ver apenas uma cidade ou estado|
|Produto|Ver uma categoria específica|
|Cliente|Analisar um cliente específico|
|Status|Filtrar ativos, cancelados, pendentes etc.|

Filtros são importantes, mas em excesso podem deixar o dashboard confuso ou lento.

### O que falar na entrevista

> Filters permitem segmentar a análise e dar autonomia ao usuário. Porém, eles devem ser usados com cuidado para não poluir o dashboard nem prejudicar performance.

---

# 3. Modelagem analítica

Modelagem analítica é a organização dos dados para facilitar análise, criação de métricas e consumo por ferramentas de BI.

A ideia é transformar dados operacionais em estruturas mais simples, consistentes e performáticas para consulta.

---

## 3.1 Tabela fato

Tabela fato é a tabela que armazena eventos de negócio e métricas.

Exemplos de eventos:

|Área|Evento|
|---|---|
|Vendas|Uma venda realizada|
|Atendimento|Um chamado aberto|
|Financeiro|Um pagamento|
|Logística|Uma entrega|
|Marketing|Um lead gerado|

Exemplo de tabela fato de vendas:

|id_venda|id_cliente|id_produto|data_venda|quantidade|valor|
|---|--:|--:|---|--:|--:|
|1|10|200|2026-05-01|2|500|

Normalmente, a tabela fato contém:

|Elemento|Exemplo|
|---|---|
|Chaves|`id_cliente`, `id_produto`, `id_data`|
|Métricas|`valor`, `quantidade`, `custo`|
|Evento|Venda, pedido, pagamento|

### O que falar na entrevista

> Tabela fato representa eventos de negócio e contém métricas analisáveis, como valor, quantidade, custo ou tempo. Ela geralmente se conecta a dimensões para permitir diferentes visões de análise.

---

## 3.2 Tabela dimensão

Tabela dimensão contém informações descritivas que dão contexto para a tabela fato.

Exemplo de dimensão cliente:

|id_cliente|nome|cidade|segmento|
|--:|---|---|---|
|10|João|São Paulo|Varejo|

Exemplo de dimensão produto:

|id_produto|produto|categoria|marca|
|--:|---|---|---|
|200|Notebook|Eletrônicos|Dell|

Dimensões respondem perguntas como:

|Pergunta|Dimensão|
|---|---|
|Quem comprou?|Cliente|
|O que foi vendido?|Produto|
|Quando aconteceu?|Data|
|Onde aconteceu?|Região|
|Por qual canal?|Canal|

### O que falar na entrevista

> Tabelas dimensão armazenam características descritivas dos dados. Elas permitem analisar uma métrica por diferentes perspectivas, como cliente, produto, região, tempo ou canal.

---

## 3.3 Star Schema

Star Schema, ou modelo estrela, é uma forma de modelagem em que uma tabela fato fica no centro e várias dimensões ficam ao redor.

Exemplo:

```text
             dim_cliente
                  |
dim_produto — fato_vendas — dim_data
                  |
              dim_regiao
```

A tabela fato guarda os eventos e métricas. As dimensões guardam o contexto.

### Vantagens

|Vantagem|Explicação|
|---|---|
|Simplicidade|Fácil de entender|
|Performance|Bom para consultas analíticas|
|Organização|Separa métricas e descrições|
|BI-friendly|Funciona bem com ferramentas de BI|

### O que falar na entrevista

> Star Schema é um modelo dimensional com uma tabela fato central ligada a várias dimensões. Ele facilita análises porque organiza claramente métricas e contextos de negócio.

---

## 3.4 Granularidade

Granularidade é o nível de detalhe de uma tabela.

Esse é um dos conceitos mais importantes em modelagem analítica.

Exemplo:

|Granularidade|Significado|
|---|---|
|Uma linha por venda|Cada registro representa uma venda|
|Uma linha por item vendido|Cada registro representa um item dentro da venda|
|Uma linha por cliente por mês|Cada registro representa um resumo mensal por cliente|
|Uma linha por produto por dia|Cada registro representa vendas diárias por produto|

A granularidade define o que cada linha da tabela representa.

### Exemplo prático

Se uma tabela tem uma linha por venda, você consegue analisar venda individual.

Se uma tabela tem uma linha por mês, você não consegue ver detalhes diários.

### O que falar na entrevista

> Granularidade é o nível de detalhe do dado. Antes de criar uma métrica ou tabela analítica, é essencial entender o que cada linha representa, porque isso impacta diretamente os cálculos e a interpretação dos indicadores.

---

## 3.5 Métrica

Métrica é um valor numérico que pode ser medido e analisado.

Exemplos:

|Métrica|Significado|
|---|---|
|Receita|Total vendido|
|Quantidade|Número de itens ou eventos|
|Ticket médio|Receita dividida pela quantidade de vendas|
|Tempo médio|Média de duração de um processo|
|Margem|Diferença entre receita e custo|
|Taxa de conversão|Percentual de conversões|

Métricas geralmente ficam na tabela fato.

### O que falar na entrevista

> Métrica é um valor numérico usado para medir algum aspecto do negócio. Ela precisa ter regra de cálculo clara para evitar interpretações diferentes entre áreas.

---

## 3.6 Indicador / KPI

KPI significa **Key Performance Indicator**, ou Indicador-Chave de Desempenho.

Todo KPI é uma métrica, mas nem toda métrica é um KPI.

A diferença é que o KPI está diretamente ligado a um objetivo de negócio.

Exemplo:

|Métrica|Pode virar KPI quando...|
|---|---|
|Receita|Existe uma meta de receita|
|SLA|Mede eficiência operacional|
|Retenção|Mede fidelização de clientes|
|Conversão|Mede eficiência comercial|
|Tempo médio|Mede produtividade ou atendimento|

### Exemplo

Métrica:

> Receita mensal.

KPI:

> Receita mensal comparada com a meta do mês.

### O que falar na entrevista

> KPI é uma métrica associada a um objetivo de negócio. Ele serve para acompanhar desempenho e apoiar decisões, geralmente comparando o resultado com uma meta, período anterior ou referência esperada.

---

# 4. Métricas de negócio

Métricas de negócio traduzem perguntas da empresa em números.

A lógica é:

```text
Pergunta de negócio → Regra de cálculo → Métrica → Dashboard → Decisão
```

---

## 4.1 Estamos vendendo mais? Receita mensal

Receita mensal mede quanto a empresa vendeu em determinado mês.

Exemplo:

```sql
SELECT
    DATE_TRUNC('month', data_venda) AS mes,
    SUM(valor) AS receita_mensal
FROM vendas
GROUP BY DATE_TRUNC('month', data_venda);
```

Essa métrica ajuda a entender crescimento, queda ou sazonalidade.

### O que explicar

> Receita mensal é uma métrica usada para acompanhar o volume financeiro de vendas ao longo do tempo. Ela permite comparar meses, identificar tendências e avaliar crescimento.

---

## 4.2 O atendimento está eficiente? Tempo médio de atendimento

Tempo médio de atendimento mede quanto tempo, em média, um atendimento leva para ser concluído.

Exemplo:

```text
tempo_médio = soma do tempo de atendimento / quantidade de atendimentos
```

Pode ser usado em suporte, SAC, operação, logística ou atendimento interno.

### O que explicar

> Tempo médio de atendimento mede a eficiência operacional. Se ele aumenta muito, pode indicar gargalo, sobrecarga ou processo pouco eficiente.

---

## 4.3 A operação está atrasada? SLA ou backlog

SLA significa **Service Level Agreement**. É um prazo esperado para executar uma atividade.

Exemplo:

> Resolver chamados em até 24 horas.

Se um chamado passou desse prazo, ele está fora do SLA.

Backlog é o volume de trabalho pendente.

Exemplo:

|Métrica|Significado|
|---|---|
|SLA|Mede cumprimento de prazo|
|Backlog|Mede quantidade de pendências|

### O que explicar

> SLA mede se a operação está cumprindo os prazos acordados. Backlog mede o volume de trabalho acumulado. Juntos, ajudam a entender atrasos e gargalos operacionais.

---

## 4.4 Quais produtos performam melhor? Receita por produto

Receita por produto mostra quais produtos geram mais valor financeiro.

Exemplo:

```sql
SELECT
    produto,
    SUM(valor) AS receita
FROM vendas
GROUP BY produto
ORDER BY receita DESC;
```

Essa análise pode apoiar decisões de estoque, marketing, vendas e precificação.

### O que explicar

> Receita por produto permite identificar quais produtos têm maior participação nos resultados. Essa análise ajuda a priorizar ações comerciais, estoque e campanhas.

---

## 4.5 Os clientes estão retornando? Retenção ou recompra

Retenção mede se os clientes continuam comprando ou usando o serviço ao longo do tempo.

Recompra mede se o cliente comprou novamente.

Exemplo:

|Conceito|Significado|
|---|---|
|Cliente novo|Comprou pela primeira vez|
|Cliente recorrente|Comprou mais de uma vez|
|Retenção|Percentual de clientes que continuam ativos|
|Recompra|Cliente que voltou a comprar|

### O que explicar

> Retenção e recompra ajudam a medir fidelização. Uma empresa pode vender muito em um mês, mas se os clientes não retornam, pode haver problema de experiência, produto ou relacionamento.

---

# 5. Visualização e storytelling

Visualização de dados é a forma de apresentar informações usando gráficos, tabelas, indicadores e dashboards.

Storytelling é organizar essas informações em uma sequência lógica, para facilitar a interpretação e conduzir o usuário a uma conclusão.

---

## 5.1 KPI principal

O KPI principal é o indicador mais importante do dashboard.

Ele deve aparecer em destaque, normalmente no topo.

Exemplo:

|Dashboard|KPI principal|
|---|---|
|Vendas|Receita total|
|Atendimento|SLA|
|Marketing|Taxa de conversão|
|Financeiro|Margem|
|Operação|Backlog|

### O que explicar

> O KPI principal deve ficar em destaque porque ele mostra rapidamente se o resultado está bom, ruim ou dentro do esperado.

---

## 5.2 Gráfico de linha

Gráfico de linha é usado para mostrar evolução ao longo do tempo.

Exemplos:

|Análise|Uso|
|---|---|
|Receita mensal|Ver crescimento ou queda|
|Chamados por dia|Ver picos de demanda|
|Clientes ativos por mês|Ver tendência|
|SLA semanal|Ver estabilidade operacional|

### O que explicar

> Gráfico de linha é adequado para mostrar tendências temporais, porque facilita visualizar crescimento, queda, sazonalidade e variações ao longo do tempo.

---

## 5.3 Gráfico de barras

Gráfico de barras é usado para comparar categorias.

Exemplos:

|Análise|Categoria|
|---|---|
|Receita por produto|Produto|
|Vendas por região|Região|
|Chamados por tipo|Tipo de chamado|
|Clientes por segmento|Segmento|

### O que explicar

> Gráfico de barras é bom para comparação entre categorias. Ele facilita identificar maiores, menores e diferenças entre grupos.

---

## 5.4 Tabelas

Tabelas mostram dados detalhados.

Elas são úteis quando o usuário precisa consultar informações específicas, mas não são ideais para visão executiva.

Exemplo:

|Uso adequado|Uso inadequado|
|---|---|
|Listar clientes com detalhes|Mostrar 30 colunas no painel principal|
|Exportar dados|Substituir todos os gráficos|
|Consultar registros|Mostrar resumo estratégico|

### O que explicar

> Tabelas são úteis para detalhamento, mas não devem ser o principal elemento de um dashboard executivo. Para visão rápida, gráficos e KPIs geralmente funcionam melhor.

---

## 5.5 Cores

Cores devem ter função analítica, não apenas estética.

Exemplos:

|Cor|Uso comum|
|---|---|
|Verde|Resultado positivo|
|Vermelho|Alerta ou problema|
|Cinza|Informação neutra|
|Azul|Destaque informativo|

O problema é usar muitas cores sem critério. Isso confunde o usuário.

### O que explicar

> Cores devem ser usadas com propósito, para destacar alertas, categorias ou situações relevantes. O excesso de cores pode prejudicar a leitura do dashboard.

---

## 5.6 Filtros

Filtros permitem que o usuário refine a análise.

Exemplo:

|Filtro|Uso|
|---|---|
|Período|Selecionar mês ou ano|
|Região|Ver uma localidade|
|Produto|Analisar categoria específica|
|Cliente|Consultar cliente específico|

Filtros ajudam, mas muitos filtros podem deixar o painel confuso.

### O que explicar

> Filtros dão flexibilidade ao usuário, mas precisam ser bem escolhidos. O ideal é manter apenas filtros realmente úteis para a análise.

---

## 5.7 Storytelling

Storytelling em dados é organizar o dashboard como uma narrativa.

Exemplo de sequência lógica:

```text
1. Resultado principal
2. Evolução ao longo do tempo
3. Quebra por categoria
4. Detalhamento
5. Possíveis causas ou pontos de atenção
```

Isso evita que o dashboard seja apenas um conjunto solto de gráficos.

### O que explicar

> Storytelling é estruturar a análise para conduzir o usuário. Em vez de apenas mostrar gráficos, o dashboard deve ajudar a entender o que aconteceu, onde aconteceu e quais pontos merecem atenção.

---

# 6. Qualidade e governança de dados

Qualidade e governança garantem que os dados usados nos dashboards sejam confiáveis, consistentes e bem documentados.

---

## 6.1 Qualidade

Qualidade de dados significa verificar se os dados estão corretos, completos e coerentes.

Validações comuns:

|Validação|Exemplo|
|---|---|
|Nulos|Cliente sem identificação|
|Duplicados|Venda registrada duas vezes|
|Tipo de dado|Data salva como texto|
|Formato|CPF/CNPJ inválido|
|Regra de negócio|Venda com valor negativo indevido|

### O que explicar

> Qualidade de dados envolve validar se os dados estão completos, corretos e coerentes com as regras de negócio antes de usá-los em análises ou dashboards.

---

## 6.2 Consistência

Consistência significa garantir que a mesma métrica tenha o mesmo cálculo em todos os lugares.

Exemplo de problema:

|Área|Cálculo de receita|
|---|---|
|Comercial|Soma vendas brutas|
|Financeiro|Soma vendas líquidas|
|Diretoria|Soma vendas faturadas|

Se todos chamam isso de “receita”, mas calculam diferente, existe inconsistência.

### O que explicar

> Consistência significa garantir que conceitos e métricas sejam calculados da mesma forma. Isso evita que áreas diferentes apresentem números diferentes para o mesmo indicador.

---

## 6.3 Documentação

Documentação é registrar as principais informações sobre dados, métricas e processos.

Exemplo do que documentar:

|Item|Exemplo|
|---|---|
|Fonte|Sistema de vendas|
|Campo|`valor_total`|
|Regra|Receita = valor total - descontos|
|Atualização|Diária às 08h|
|Responsável|Área de dados|
|Observação|Cancelamentos são removidos|

### O que explicar

> Documentação ajuda a manter clareza sobre origem, significado e regra de cálculo dos dados. Ela facilita manutenção, governança e alinhamento entre áreas.

---

## 6.4 Linhagem

Linhagem de dados é saber de onde o dado veio, por onde passou e como foi transformado.

Exemplo:

```text
Sistema de vendas → Pipeline SQL → View analítica → Tableau → Dashboard
```

A linhagem ajuda a responder:

|Pergunta|Exemplo|
|---|---|
|De onde veio esse número?|Sistema de vendas|
|Como foi calculado?|Soma de pedidos aprovados|
|Onde foi transformado?|View SQL|
|Quem consome?|Dashboard comercial|

### O que explicar

> Linhagem permite rastrear a origem e as transformações dos dados. Isso é importante para auditoria, correção de problemas e confiabilidade dos indicadores.

---

## 6.5 Validação

Validação é comparar o resultado final com uma referência esperada.

Exemplo:

Se o dashboard mostra receita mensal de R$ 500 mil, você pode validar esse número comparando com:

|Referência|Exemplo|
|---|---|
|Sistema origem|Total no ERP|
|Query independente|Consulta SQL separada|
|Relatório antigo|Relatório financeiro|
|Regra de negócio|Apenas vendas aprovadas|
|Amostra manual|Conferência de alguns registros|

### O que explicar

> Validação é o processo de conferir se o resultado final está correto. Antes de publicar um dashboard, é importante comparar números com a fonte ou com uma regra de referência.

---

# 7. Dremio e Data Lake

Esse tópico aparece na vaga porque a empresa provavelmente usa dados armazenados em ambiente de Data Lake e consulta esses dados via Dremio.

---

## 7.1 Data Lake

Data Lake é um ambiente usado para armazenar grandes volumes de dados em formatos variados.

Diferente de um banco relacional tradicional, um Data Lake pode armazenar dados mais flexíveis, como:

|Formato|Uso|
|---|---|
|CSV|Arquivos tabulares simples|
|JSON|Dados semiestruturados|
|Parquet|Formato colunar otimizado para análise|
|Avro|Formato comum em pipelines|
|Delta/Iceberg/Hudi|Formatos de tabela para Data Lake|

Exemplo de fluxo:

```text
Sistemas origem → Data Lake → Processamento → Camada analítica → BI
```

### O que explicar

> Data Lake é um ambiente para armazenar dados de diferentes fontes e formatos. Ele é muito usado em arquiteturas analíticas porque permite centralizar dados brutos, tratados e preparados para consumo.

---

## 7.2 Dremio

Dremio é uma plataforma que permite consultar dados usando SQL, principalmente em ambientes de Data Lake.

Ele funciona como uma camada entre os dados e as ferramentas de consumo, como Tableau.

Exemplo:

```text
Arquivos no Data Lake → Dremio → View SQL → Tableau
```

Com Dremio, a empresa pode consultar dados em arquivos Parquet, fontes relacionais ou outros sistemas usando SQL.

### O que explicar

> Dremio funciona como uma camada SQL para consultar dados em Data Lakes e outras fontes. Ele facilita a criação de views analíticas e a disponibilização dos dados para ferramentas de BI.

---

## 7.3 View analítica

View analítica é uma consulta organizada para consumo por análise ou BI.

Ela normalmente já entrega os dados em uma estrutura mais amigável para o dashboard.

Exemplo:

Em vez de o Tableau acessar várias tabelas brutas, ele pode acessar uma view como:

```text
vw_receita_mensal_por_produto
```

Essa view já pode conter:

|Campo|Exemplo|
|---|---|
|Mês|Janeiro|
|Produto|Notebook|
|Categoria|Eletrônicos|
|Receita|150000|
|Quantidade|300|

### O que explicar

> View analítica é uma camada organizada de dados para consumo em BI. Ela ajuda a padronizar regras, simplificar consultas e melhorar a confiabilidade dos dashboards.

---

## 7.4 Performance em Data Lake / Dremio

Performance nesse contexto significa tornar as consultas rápidas mesmo quando os dados estão em grandes volumes.

Fatores que influenciam performance:

|Fator|Impacto|
|---|---|
|Formato dos dados|Parquet costuma ser melhor para análise|
|Particionamento|Reduz volume lido|
|Volume consultado|Menos dados melhora tempo|
|Views bem modeladas|Facilitam consumo|
|Agregações|Podem reduzir processamento|
|Reflections no Dremio|Podem acelerar consultas|

### O que explicar

> Em Data Lake, performance depende muito da organização dos dados, formato dos arquivos, particionamento e estrutura das consultas. No Dremio, views e otimizações podem ajudar a entregar dados mais rapidamente para ferramentas como Tableau.

---

# Resumo final para a entrevista

Você pode consolidar tudo assim:

> Para essa vaga, entendo que o foco está em transformar dados em informação analítica confiável. Para isso, é importante dominar SQL analítico, modelagem de dados, criação de views, qualidade dos dados e boas práticas de visualização. O Tableau entra como ferramenta de consumo e apresentação, enquanto Dremio e Data Lake fazem parte da camada de acesso e disponibilização dos dados. Minha base em Engenharia de Dados contribui principalmente na preparação, estruturação, validação e documentação dos dados para que os dashboards tenham consistência, performance e valor para o negócio.