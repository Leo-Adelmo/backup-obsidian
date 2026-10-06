## 1. OLTP

**OLTP** significa **Online Transaction Processing**, ou **Processamento de Transações Online**.

É o tipo de sistema usado para **registrar operações do dia a dia** de uma empresa.

Exemplos:

- Venda realizada
    
- Pedido criado
    
- Pagamento efetuado
    
- Cadastro de cliente
    
- Atualização de estoque
    
- Emissão de nota fiscal
    

No contexto de um analista de dados, o OLTP geralmente representa a **fonte original dos dados**.

Exemplo de tabela OLTP:

|id_pedido|id_cliente|data_pedido|status|
|---|--:|---|---|
|101|1|2026-05-13|Pago|
|102|2|2026-05-13|Pendente|

Características principais:

|Característica|OLTP|
|---|---|
|Foco|Operação do negócio|
|Uso principal|Registrar transações|
|Tipo de consulta|Consultas simples e rápidas|
|Dados|Detalhados e atuais|
|Usuários|Sistemas, operadores, clientes|
|Exemplo|Sistema de vendas, ERP, CRM|

---

## 2. OLAP

**OLAP** significa **Online Analytical Processing**, ou **Processamento Analítico Online**.

É o tipo de sistema usado para **análise de dados**, criação de relatórios, dashboards e indicadores.

Exemplos:

- Total de vendas por mês
    
- Receita por região
    
- Ticket médio por cliente
    
- Produtos mais vendidos
    
- Comparação entre períodos
    
- Análise de performance comercial
    

No contexto de um analista de dados, o OLAP é onde os dados são organizados para **facilitar análise e tomada de decisão**.

Exemplo de consulta OLAP:

|mês|total_vendas|quantidade_pedidos|ticket_médio|
|---|--:|--:|--:|
|Janeiro|150.000|1.200|125|
|Fevereiro|180.000|1.350|133|

Características principais:

|Característica|OLAP|
|---|---|
|Foco|Análise do negócio|
|Uso principal|Relatórios, dashboards e KPIs|
|Tipo de consulta|Consultas agregadas e analíticas|
|Dados|Históricos e consolidados|
|Usuários|Analistas, gestores, BI|
|Exemplo|Data Warehouse, Data Mart, Power BI, Tableau|

---

## 3. Diferença principal

A diferença central é:

> **OLTP registra o que aconteceu. OLAP analisa o que aconteceu.**

Exemplo prático:

|Situação|Sistema|
|---|---|
|Registrar uma venda no caixa|OLTP|
|Analisar vendas por mês|OLAP|
|Atualizar cadastro de cliente|OLTP|
|Ver clientes mais rentáveis|OLAP|
|Processar pagamento|OLTP|
|Criar dashboard de faturamento|OLAP|

---

## 4. Visão no trabalho de um analista de dados

Um analista de dados geralmente não trabalha diretamente alterando sistemas OLTP. Ele costuma **extrair, consultar ou receber dados desses sistemas** para depois analisar em ambientes OLAP.

Fluxo comum:

```text
Sistema OLTP → ETL/ELT → Data Warehouse/Data Mart → Dashboard/Relatório
```

Exemplo:

```text
Sistema de vendas → Tratamento dos dados → Tabela analítica → Dashboard de vendas
```

---

## 5. Tabela resumo

|Tema|OLTP|OLAP|
|---|---|---|
|Significado|Processamento transacional|Processamento analítico|
|Objetivo|Registrar operações|Analisar informações|
|Tipo de dado|Detalhado|Consolidado|
|Tempo dos dados|Atual/operacional|Histórico|
|Exemplo de uso|Criar pedido|Analisar vendas|
|Consulta típica|Buscar pedido por ID|Somar vendas por mês|
|Modelo comum|Normalizado|Dimensional|
|Usuário principal|Sistema operacional|Analista de dados|
|Ferramentas comuns|ERP, CRM, sistema interno|Data Warehouse, Power BI, Tableau|
|Pergunta que responde|“O que aconteceu agora?”|“O que aconteceu ao longo do tempo?”|
## Ferramentas de OLTP

Ferramentas OLTP são usadas para **registrar e consultar transações operacionais**.

|Ferramenta|Exemplo de uso|
|---|---|
|**PostgreSQL**|Sistema de vendas, cadastro de clientes, pedidos|
|**MySQL**|Aplicações web, e-commerce, sistemas internos|
|**SQL Server**|ERP, sistemas corporativos, financeiro|
|**Oracle Database**|Bancos transacionais de grandes empresas|
|**MariaDB**|Sistemas web e aplicações operacionais|
|**MongoDB**|Aplicações com dados semi-estruturados, como catálogos e perfis|
|**DynamoDB**|Aplicações em nuvem com alta escala transacional|
|**Firebase Realtime Database / Firestore**|Aplicativos mobile e web em tempo real|

Exemplo no contexto de negócio:

|Sistema|Tipo|
|---|---|
|Sistema de vendas|OLTP|
|Sistema de pedidos|OLTP|
|ERP|OLTP|
|CRM operacional|OLTP|
|Sistema bancário|OLTP|
|Aplicativo de delivery|OLTP|

---

## Ferramentas de OLAP

Ferramentas OLAP são usadas para **análise, relatórios, dashboards e consultas analíticas**.

|Ferramenta|Exemplo de uso|
|---|---|
|**BigQuery**|Data Warehouse em nuvem para análise de grandes volumes|
|**Snowflake**|Plataforma analítica para Data Warehouse|
|**Amazon Redshift**|Data Warehouse na AWS|
|**Azure Synapse Analytics**|Data Warehouse e análise na Azure|
|**Google Looker / Looker Studio**|Dashboards e BI|
|**Power BI**|Relatórios, dashboards e indicadores|
|**Tableau**|Dashboards e análises visuais|
|**Apache Hive**|Consultas analíticas sobre dados em Data Lake|
|**Databricks SQL**|Análise SQL sobre Lakehouse|
|**Apache Druid**|Análise rápida em dados agregados e eventos|
|**ClickHouse**|Banco colunar para consultas analíticas rápidas|
|**Apache Pinot**|Análise em tempo real sobre grandes volumes de eventos|

Exemplo no contexto de negócio:

|Sistema|Tipo|
|---|---|
|Data Warehouse|OLAP|
|Data Mart|OLAP|
|Dashboard comercial|OLAP|
|Cubo analítico|OLAP|
|Relatório gerencial|OLAP|
|Plataforma de BI|OLAP|

---

## Tabela comparativa

|Categoria|OLTP|OLAP|
|---|---|---|
|Bancos comuns|PostgreSQL, MySQL, SQL Server, Oracle|BigQuery, Snowflake, Redshift, Synapse|
|BI/Visualização|Normalmente não é o foco|Power BI, Tableau, Looker|
|Uso principal|Operação do sistema|Análise de dados|
|Exemplo|Registrar pedido|Analisar vendas por mês|
|Perfil de consulta|Simples e frequente|Agregada e analítica|
|Usuário comum|Aplicações e sistemas|Analistas, BI, gestores|