## 1. Consulta de dados

| Comando    | Para que serve                            | Exemplo                                   |
| ---------- | ----------------------------------------- | ----------------------------------------- |
| `SELECT`   | Selecionar colunas de uma tabela          | `SELECT nome, idade FROM clientes;`       |
| `FROM`     | Indicar de qual tabela os dados vêm       | `SELECT * FROM vendas;`                   |
| `WHERE`    | Filtrar linhas com uma condição           | `SELECT * FROM vendas WHERE valor > 100;` |
| `DISTINCT` | Remover valores duplicados                | `SELECT DISTINCT cidade FROM clientes;`   |
| `LIMIT`    | Limitar a quantidade de linhas retornadas | `SELECT * FROM vendas LIMIT 10;`          |

---

## 2. Ordenação e filtros

|Comando|Para que serve|Exemplo|
|---|---|---|
|`ORDER BY`|Ordenar os resultados|`SELECT * FROM vendas ORDER BY data_venda DESC;`|
|`ASC`|Ordenar em ordem crescente|`ORDER BY valor ASC;`|
|`DESC`|Ordenar em ordem decrescente|`ORDER BY valor DESC;`|
|`BETWEEN`|Filtrar valores dentro de um intervalo|`WHERE valor BETWEEN 100 AND 500;`|
|`IN`|Filtrar valores dentro de uma lista|`WHERE estado IN ('SP', 'RJ', 'MG');`|
|`LIKE`|Buscar padrões em textos|`WHERE nome LIKE 'Ana%';`|
|`IS NULL`|Verificar valores nulos|`WHERE email IS NULL;`|
|`IS NOT NULL`|Verificar valores não nulos|`WHERE email IS NOT NULL;`|

---

## 3. Agregações

|Comando/Função|Para que serve|Exemplo|
|---|---|---|
|`COUNT()`|Contar registros|`SELECT COUNT(*) FROM vendas;`|
|`SUM()`|Somar valores|`SELECT SUM(valor) FROM vendas;`|
|`AVG()`|Calcular média|`SELECT AVG(valor) FROM vendas;`|
|`MIN()`|Retornar o menor valor|`SELECT MIN(valor) FROM vendas;`|
|`MAX()`|Retornar o maior valor|`SELECT MAX(valor) FROM vendas;`|
|`GROUP BY`|Agrupar dados para cálculo de métricas|`SELECT cidade, COUNT(*) FROM clientes GROUP BY cidade;`|
|`HAVING`|Filtrar resultados após agregação|`HAVING COUNT(*) > 10;`|

---

## 4. Combinação de tabelas

|Comando|Para que serve|Exemplo|
|---|---|---|
|`INNER JOIN`|Retorna apenas registros com correspondência nas duas tabelas|`clientes INNER JOIN vendas ON clientes.id = vendas.cliente_id`|
|`LEFT JOIN`|Retorna todos os registros da tabela da esquerda|`clientes LEFT JOIN vendas ON clientes.id = vendas.cliente_id`|
|`RIGHT JOIN`|Retorna todos os registros da tabela da direita|`clientes RIGHT JOIN vendas ON clientes.id = vendas.cliente_id`|
|`FULL JOIN`|Retorna registros correspondentes e não correspondentes das duas tabelas|`clientes FULL JOIN vendas ON clientes.id = vendas.cliente_id`|

Exemplo:

```sql
SELECT
    clientes.nome,
    vendas.valor
FROM clientes
LEFT JOIN vendas
    ON clientes.id = vendas.cliente_id;
```

---

## 5. Regras condicionais

|Comando|Para que serve|Exemplo|
|---|---|---|
|`CASE WHEN`|Criar classificações ou regras condicionais|Classificar vendas como alta ou baixa|

Exemplo:

```sql
SELECT
    id_venda,
    valor,
    CASE
        WHEN valor >= 1000 THEN 'Venda alta'
        WHEN valor >= 500 THEN 'Venda média'
        ELSE 'Venda baixa'
    END AS categoria_venda
FROM vendas;
```

---

## 6. Organização de consultas

|Comando|Para que serve|Exemplo|
|---|---|---|
|`WITH` / `CTE`|Criar consultas temporárias para organizar a lógica|Usar uma consulta intermediária antes da consulta final|
|`VIEW`|Criar uma consulta reutilizável|Criar uma visão para dashboards|

Exemplo com CTE:

```sql
WITH vendas_por_cliente AS (
    SELECT
        cliente_id,
        SUM(valor) AS total_vendido
    FROM vendas
    GROUP BY cliente_id
)

SELECT
    cliente_id,
    total_vendido
FROM vendas_por_cliente
WHERE total_vendido > 1000;
```

---

## 7. Window Functions

Usadas para fazer cálculos analíticos sem perder o nível de detalhe das linhas.

|Função|Para que serve|Exemplo de uso|
|---|---|---|
|`ROW_NUMBER()`|Criar numeração sequencial|Ranking simples|
|`RANK()`|Criar ranking com empates|Ranking de vendas|
|`DENSE_RANK()`|Ranking sem pular posições|Ranking mais compacto|
|`LAG()`|Buscar valor da linha anterior|Comparar venda atual com venda anterior|
|`LEAD()`|Buscar valor da próxima linha|Comparar com próximo registro|
|`SUM() OVER`|Soma acumulada ou por janela|Receita acumulada|
|`AVG() OVER`|Média por janela|Média móvel|

Exemplo:

```sql
SELECT
    cliente_id,
    data_venda,
    valor,
    SUM(valor) OVER (
        PARTITION BY cliente_id
        ORDER BY data_venda
    ) AS valor_acumulado
FROM vendas;
```

---

## 8. Manipulação de dados

|Comando|Para que serve|Observação|
|---|---|---|
|`INSERT`|Inserir novos registros|Mais comum em processos de carga|
|`UPDATE`|Atualizar registros existentes|Deve ser usado com cuidado|
|`DELETE`|Excluir registros|Deve ser usado com muito cuidado|
|`MERGE`|Inserir ou atualizar registros conforme condição|Muito usado em pipelines e cargas incrementais|

Exemplo:

```sql
UPDATE clientes
SET cidade = 'São Paulo'
WHERE id = 10;
```

---

## 9. Criação e estrutura de tabelas

|Comando|Para que serve|
|---|---|
|`CREATE TABLE`|Criar uma tabela|
|`ALTER TABLE`|Alterar estrutura de uma tabela|
|`DROP TABLE`|Excluir uma tabela|
|`TRUNCATE TABLE`|Limpar todos os dados de uma tabela|
|`CREATE VIEW`|Criar uma visão reutilizável|

Exemplo:

```sql
CREATE TABLE clientes (
    id INT,
    nome VARCHAR(100),
    cidade VARCHAR(100)
);
```

---

## 10. Principais comandos para dashboards e análises

Esses são os mais importantes para um Analista de Dados:

|Prioridade|Comando|
|---|---|
|Muito alta|`SELECT`|
|Muito alta|`WHERE`|
|Muito alta|`JOIN`|
|Muito alta|`GROUP BY`|
|Muito alta|`COUNT`, `SUM`, `AVG`, `MIN`, `MAX`|
|Alta|`CASE WHEN`|
|Alta|`ORDER BY`|
|Alta|`HAVING`|
|Alta|`WITH` / `CTE`|
|Alta|`Window Functions`|
|Média|`CREATE VIEW`|
|Média|`INSERT`, `UPDATE`, `DELETE`|

---

## Resumo final

|Categoria|Comandos principais|
|---|---|
|Consultar dados|`SELECT`, `FROM`, `WHERE`, `DISTINCT`, `LIMIT`|
|Ordenar e filtrar|`ORDER BY`, `ASC`, `DESC`, `BETWEEN`, `IN`, `LIKE`, `IS NULL`|
|Agregar dados|`COUNT`, `SUM`, `AVG`, `MIN`, `MAX`, `GROUP BY`, `HAVING`|
|Combinar tabelas|`INNER JOIN`, `LEFT JOIN`, `RIGHT JOIN`, `FULL JOIN`|
|Criar regras|`CASE WHEN`|
|Organizar consultas|`WITH`, `CTE`, `VIEW`|
|Análise avançada|`ROW_NUMBER`, `RANK`, `LAG`, `LEAD`, `SUM OVER`, `AVG OVER`|
|Manipular dados|`INSERT`, `UPDATE`, `DELETE`, `MERGE`|
|Estrutura de banco|`CREATE`, `ALTER`, `DROP`, `TRUNCATE`|