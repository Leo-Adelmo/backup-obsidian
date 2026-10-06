## 1. Definição formal

> “The grain of a fact table defines exactly what a single fact record represents. It must be declared before choosing dimensions or measures.”  
> — _Kimball & Ross, The Data Warehouse Toolkit, 3rd Edition, Wiley, 2013_

**Tradução livre:** O grão de uma tabela fato define exatamente o que cada registro representa. Ele deve ser declarado antes de escolher dimensões ou medidas.

Explicação: o grão é a **unidade mínima de observação** de uma tabela fato. Se o grão for “uma venda individual”, então cada linha do fato representa uma transação única e irrepetível.

---

## 2. Tipos comuns de grão

> “Fact tables can capture data at different levels of detail, such as transaction, periodic snapshot, or accumulating snapshot.”  
> — _Kimball Group, Dimensional Modeling Techniques, kimballgroup.com_

**Tradução livre:** Tabelas fato podem capturar dados em diferentes níveis de detalhe, como transações, snapshots periódicos ou snapshots acumulativos.

|Tipo de grão|Exemplo|Descrição|
|---|---|---|
|**Transacional (atomic)**|Cada venda, clique, pagamento|Registro por evento individual. Maior detalhamento possível.|
|**Snapshot periódico**|Vendas diárias, estoque mensal|Registro de resumo periódico (ex.: por dia).|
|**Snapshot acumulativo**|Pipeline de pedidos, processos de entrega|Linha representa progresso de um processo, sendo atualizada ao longo do tempo.|

---

## 3. Importância do grão

> “A clearly stated grain is the foundation of every fact table design. Without it, the design becomes ambiguous and measures lose meaning.”  
> — _Kimball & Ross, The Data Warehouse Toolkit, 2013_

**Tradução livre:** Uma definição clara do grão é a base de qualquer design de tabela fato. Sem ela, o design se torna ambíguo e as medidas perdem o sentido.

Impactos diretos:

- **Modelagem:** define quais dimensões e chaves devem existir.
    
- **Consistência:** garante que todas as medidas sejam comparáveis.
    
- **Performance:** quanto mais fino o grão, maior o volume de dados e custo de processamento.
    

---

## 4. Exemplos práticos

**Fato de Vendas (grão transacional)**  
Cada linha = uma venda individual.

|DataHora|Loja|Produto|Quantidade|Valor|
|---|---|---|---|---|
|2025-10-09 14:32|SP001|12345|2|50.00|

**Fato de Estoque (snapshot diário)**  
Cada linha = saldo do produto por dia.

|Data|Loja|Produto|Estoque_Qtd|
|---|---|---|---|
|2025-10-09|SP001|12345|340|

---

## 5. Grão e versionamento (SCD em fatos)

> “When facts must reflect changes over time, such as re-stated balances or evolving process milestones, the fact table may adopt a timespan grain with start and end dates.”  
> — _Joy Mundy, Design Tip #145, Kimball Group, 2012_

**Tradução livre:** Quando os fatos precisam refletir mudanças ao longo do tempo — como saldos reprocessados ou etapas evolutivas de um processo — a tabela fato pode adotar um grão de período de validade, com datas de início e fim.

Aplicação: mantém-se o mesmo grão lógico (ex.: “saldo diário por produto”), mas permite-se registrar múltiplas versões históricas da mesma observação.

---

## 6. Referências

1. Kimball, R. & Ross, M. (2013). _The Data Warehouse Toolkit: The Definitive Guide to Dimensional Modeling._ Wiley.
    
2. Kimball Group (2012). _Design Tip #145: Time-Stamping Accumulating Snapshot Fact Tables._ [kimballgroup.com](https://www.kimballgroup.com/2012/05/design-tip-145-time-stamping-accumulating-snapshot-fact-tables/?utm_source=chatgpt.com)
    
3. Kimball Group. _Dimensional Modeling Techniques._ kimballgroup.com
    
4. Golfarelli, M. & Rizzi, S. (2009). _Data Warehouse Design: Modern Principles and Methodologies._ McGraw-Hill.
    
5. Vaisman, A. & Zimányi, E. (2022). _Data Warehouse Systems: Design and Implementation._ Springer.



update dps insert dps se não bater nada, delete

1 - Atribuir is_current = True para todos os dados da batch

2 - Função merge into
when match target.chave_natural = batch.chave_natural
	when match target.hash !=  batch.hash
		target.is_current = false
		batch.is_current = true
when not
	delete target_row
	
3 - Adicionar todas as linhas do batch cuja o is_current = true

when match target.hash != batch.hash
	when match batch.chave_natural = trigger.chave_natural


se target.chave_natural = batch.chave_natural, então 
	se target.hash != batch.hash, então
		is_current = false
se não, então:
	 insert nova 


when batch.




MERGE INTO target 
USING batch 
ON target.chave_natural = batch.chave_natural
WHEN MATCHED AND target.hash != batch.hash THEN
  UPDATE SET
    target.is_current = false
     INSERT *
WHEN NOT MATCHED THEN
  UPDATE SET
     target.deleted = True
      INSERT *


MERGE INTO target
USING batch
ON target.hash = batch.hash 

WHEN MATCHED
  UPDATE SET 
	  target_is_current = True
	  
WHEN NOT MATCHED AND target.chave_natural = batch.chave_natural  THEN
   UPDATE SET
     target_is_current = False

WHEN NOT MATCHED AND target.chave_natural != batch.chave_natural THEN
    INSERT *


MERGE INTO target
USING batch
ON target.hash = batch.hash 

WHEN NOT MATCHED THEN
    INSERT *



MERGE INTO target USING batch ON target.chave_natural = batch.chave_natural WHEN MATCHED AND target.hash != batch.hash AND target.is_current = true THEN UPDATE SET target.is_current = false WHEN NOT MATCHED THEN INSERT *