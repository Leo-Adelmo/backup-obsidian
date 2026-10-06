## O Que É Normalização?

A normalização é o ramo da teoria relacional que fornece insights de design. É o processo de determinar quanta redundância existe em uma tabela. Os objetivos da normalização são:

- Ser capaz de caracterizar o nível de redundância em um esquema relacional
- Fornecer mecanismos para transformar esquemas a fim de remover redundância

A teoria de normalização baseia-se fortemente na teoria de dependências funcionais. A teoria de normalização define seis formas normais (FN). Cada forma normal envolve um conjunto de propriedades de dependência que um esquema deve satisfazer e cada forma normal fornece garantias sobre a presença e/ou ausência de anomalias de atualização. Isso significa que formas normais mais altas têm menos redundância e, como resultado, menos problemas de atualização.

## Formas Normais

Todas as tabelas em qualquer banco de dados podem estar em uma das formas normais que discutiremos a seguir. Idealmente, queremos apenas redundância mínima para CP (Chave Primária) para CE (Chave Estrangeira). Tudo o mais deve ser derivado de outras tabelas. Existem seis formas normais, mas vamos examinar apenas as primeiras quatro, que são:

- Primeira forma normal (1FN)
- Segunda forma normal (2FN)
- Terceira forma normal (3FN)
- Forma normal de Boyce-Codd (FNBC)

FNBC é raramente usada.

## Primeira Forma Normal (1FN)

Na primeira forma normal, apenas valores únicos são permitidos na interseção de cada linha e coluna; portanto, não há grupos repetitivos.

Para normalizar uma relação que contém um grupo repetitivo, remova o grupo repetitivo e forme duas novas relações.

A CP da nova relação é uma combinação da CP da relação original mais um atributo da relação recém-criada para identificação única.

Ou seja, uma tabela está na 1FN quando satisfaz todas estas condições:

- **Atomicidade**: Cada célula contém apenas um valor indivisível
- **Ausência de grupos repetitivos**: Não há colunas que representem conjuntos de valores similares
- **Unicidade de linhas**: Cada linha deve ser única
- **Ordem irrelevante**: A ordem das linhas não deve importar para o significado dos dados

### Conceitos-Chave

**Valor Atômico**: É um valor que não pode ser dividido em partes menores dentro do contexto do banco de dados. Por exemplo, um nome completo "João Silva" pode ser considerado atômico se você sempre trata o nome como uma unidade, mas deve ser dividido em "João" e "Silva" se você precisa trabalhar com nome e sobrenome separadamente.

### Exemplos (Gerados por IA)

**❌ Violação da 1FN - Valores Não-Atômicos:**

```
ID | Cliente      | Endereço
1  | João Silva   | Rua A, 123, São Paulo, SP, 01234-567
2  | Maria Santos | Av. B, 456, Rio de Janeiro, RJ, 12345-678
```

**✅ Corrigido para 1FN:**

```
ID | Nome  | Sobrenome | Rua    | Numero | Cidade         | Estado | CEP
1  | João  | Silva     | Rua A  | 123    | São Paulo      | SP     | 01234-567
2  | Maria | Santos    | Av. B  | 456    | Rio de Janeiro | RJ     | 12345-678
```

**❌ Violação da 1FN - Grupos Repetitivos:**

```
ID | Cliente | Telefone1     | Telefone2     | Telefone3
1  | João    | (11)1234-5678 | (11)9876-5432 | NULL
2  | Maria   | (21)5555-1234 | NULL          | NULL
```

**✅ Corrigido para 1FN:**

```
ClienteID | Cliente | Telefone      | TipoTelefone
1         | João    | (11)1234-5678 | Residencial
1         | João    | (11)9876-5432 | Celular
2         | Maria   | (21)5555-1234 | Residencial
```

## Segunda Forma Normal (2FN)

Para a segunda forma normal, a relação deve primeiro estar em 1FN. A relação está automaticamente em 2FN se, e somente se, a CP compreende um único atributo.

Se a relação tem uma CP composta, então cada atributo não-chave deve ser totalmente dependente de toda a CP e não de um subconjunto da CP (ou seja, não deve haver dependência parcial ou aumentativa).

Ou seja, uma tabela está na 2FN quando:

- **Está na 1FN**
- **Não possui dependências funcionais parciais** da chave primária

### Conceitos-Chave

**Chave Primária Composta**: É uma chave primária formada por duas ou mais colunas. Por exemplo: `(PedidoID, ProdutoID)` em uma tabela de itens de pedido.

**Dependência Funcional**: Existe dependência funcional de A para B (A → B) quando, para cada valor de A, existe exatamente um valor correspondente de B. Por exemplo: `CPF → Nome` (um CPF determina unicamente um nome).

**Dependência Funcional Parcial**: Ocorre quando um atributo não-chave depende apenas de **parte** da chave primária composta, não da chave inteira.

### Exemplo (Gerado por IA)

**❌ Violação da 2FN:**

```
PedidoID | ProdutoID | NomeProduto | Categoria | Quantidade | DataPedido
001      | A123      | Notebook    | Eletrônico| 2          | 2024-01-15
001      | B456      | Mouse       | Eletrônico| 1          | 2024-01-15
002      | A123      | Notebook    | Eletrônico| 1          | 2024-01-16
```

**Análise dos problemas:**

- Chave primária: `(PedidoID, ProdutoID)`
- `NomeProduto` depende apenas de `ProdutoID` (dependência parcial)
- `Categoria` depende apenas de `ProdutoID` (dependência parcial)
- `DataPedido` depende apenas de `PedidoID` (dependência parcial)
- `Quantidade` depende da chave completa (OK)

**✅ Corrigido para 2FN:**

Tabela Pedidos:

```
PedidoID | DataPedido
001      | 2024-01-15
002      | 2024-01-16
```

Tabela Produtos:

```
ProdutoID | NomeProduto | Categoria
A123      | Notebook    | Eletrônico
B456      | Mouse       | Eletrônico
```

Tabela ItensPedido:

```
PedidoID | ProdutoID | Quantidade
001      | A123      | 2
001      | B456      | 1
002      | A123      | 1
```
## Terceira Forma Normal (3FN)

Para estar na terceira forma normal, a relação deve estar na segunda forma normal. Também todas as dependências transitivas devem ser removidas; um atributo não-chave não pode ser funcionalmente dependente de outro atributo não-chave.

Ou seja, uma tabela está na 3FN quando:

- **Estar na 2FN**
- **Não possuir dependências funcionais transitivas**

### Conceitos-Chave

**Dependência Funcional Transitiva**: Ocorre quando um atributo não-chave A depende de outro atributo não-chave B, que por sua vez depende da chave primária. Matematicamente: se `Chave → B` e `B → A`, então existe dependência transitiva `Chave → B → A`.

### Exemplo (Gerado por IA)

**❌ Violação da 3FN:**

```
ClienteID | NomeCliente | CidadeCliente | EstadoCliente | RegiaoEstado | PopulacaoEstado
001       | João Silva  | São Paulo     | SP            | Sudeste      | 46000000
002       | Ana Costa   | Recife        | PE            | Nordeste     | 9500000
003       | Carlos Lima | Campinas      | SP            | Sudeste      | 46000000
```

**Análise da dependência transitiva:**

- `ClienteID → EstadoCliente` (normal)
- `EstadoCliente → RegiaoEstado` (dependência entre não-chaves)
- `EstadoCliente → PopulacaoEstado` (dependência entre não-chaves)
- Logo: `ClienteID → EstadoCliente → RegiaoEstado` (transitiva)

**Problemas identificados:**

1. **Redundância**: Informações sobre SP aparecem em múltiplas linhas
2. **Anomalia de atualização**: Se a população de SP mudar, precisa atualizar múltiplas linhas
3. **Anomalia de inserção**: Não posso inserir dados sobre um estado sem ter um cliente desse estado
4. **Anomalia de exclusão**: Se excluir todos os clientes de PE, perco dados sobre o estado

**✅ Corrigido para 3FN:**

Tabela Clientes:

```
ClienteID | NomeCliente | CidadeCliente | EstadoCliente
001       | João Silva  | São Paulo     | SP
002       | Ana Costa   | Recife        | PE
003       | Carlos Lima | Campinas      | SP
```

Tabela Estados:

```
EstadoCliente | RegiaoEstado | PopulacaoEstado
SP            | Sudeste      | 46000000
PE            | Nordeste     | 9500000
RJ            | Sudeste      | 17000000
```
## 4ª Forma Normal (4FN)

A 4FN trata de **dependências multivaloradas** (DMV). Uma tabela está na 4FN quando:

- Já está na 3FN (ou FNBC)
- Não possui dependências multivaloradas não triviais

**O que é uma dependência multivalorada?** Ocorre quando um atributo determina um conjunto de valores de outro atributo, independentemente de outros atributos.

**Exemplo de violação da 4FN:**

```
PROFESSOR
+-------------+-------------+---------------+
| Professor   | Disciplina  | Hobby         |
+-------------+-------------+---------------+
| João        | Matemática  | Futebol       |
| João        | Matemática  | Leitura       |
| João        | Física      | Futebol       |
| João        | Física      | Leitura       |
+-------------+-------------+---------------+
```

Aqui temos duas dependências multivaloradas independentes:

- Um professor → várias disciplinas
- Um professor → vários hobbies

O problema: disciplinas e hobbies não têm relação entre si, mas são forçados a formar combinações cartesianas, gerando redundância.

**Solução - normalizar para 4FN:**

```
PROFESSOR_DISCIPLINA
+-------------+-------------+
| Professor   | Disciplina  |
+-------------+-------------+
| João        | Matemática  |
| João        | Física      |
+-------------+-------------+

PROFESSOR_HOBBY
+-------------+-------------+
| Professor   | Hobby       |
+-------------+-------------+
| João        | Futebol     |
| João        | Leitura     |
+-------------+-------------+
```

## 5ª Forma Normal (5FN)

A 5FN, também chamada de **Forma Normal de Projeção-Junção (PJ/NF)**, trata de dependências de junção. Uma tabela está na 5FN quando:

- Já está na 4FN
- Não pode ser decomposta em tabelas menores sem perda de informação

**Exemplo de violação da 5FN:**

```
FORNECEDOR_PRODUTO_PROJETO
+-------------+----------+-----------+
| Fornecedor  | Produto  | Projeto   |
+-------------+----------+-----------+
| F1          | P1       | Proj1     |
| F1          | P2       | Proj1     |
| F2          | P1       | Proj2     |
+-------------+----------+-----------+
```

Se houver uma regra de negócio onde:

- Um fornecedor pode fornecer determinados produtos
- Um projeto pode usar determinados produtos
- Um fornecedor pode participar de determinados projetos
- **E essas relações são independentes**

Então podemos decompor em:

```
FORNECEDOR_PRODUTO        PRODUTO_PROJETO         FORNECEDOR_PROJETO
+-------------+--------+  +----------+---------+  +-------------+---------+
| Fornecedor  | Produto|  | Produto  | Projeto |  | Fornecedor  | Projeto |
+-------------+--------+  +----------+---------+  +-------------+---------+
| F1          | P1     |  | P1       | Proj1   |  | F1          | Proj1   |
| F1          | P2     |  | P2       | Proj1   |  | F2          | Proj2   |
| F2          | P1     |  | P1       | Proj2   |  +-------------+---------+
+-------------+--------+  +----------+---------+
```

## Quando usar?

Na prática, 4FN e 5FN são raramente aplicadas porque:

- São complexas de identificar
- Podem gerar muitas junções (impactando performance)
- A maioria dos problemas é resolvida até a 3FN/FNBC
- São mais relevantes em sistemas com regras de negócio muito específicas

A normalização até 3FN geralmente oferece o melhor equilíbrio entre integridade de dados e performance.

## Normalização e Design de Banco de Dados

Durante o processo de normalização do design de banco de dados, certifique-se de que as entidades propostas atendam à forma normal exigida antes que as estruturas de tabela sejam criadas. Muitos bancos de dados do mundo real foram inadequadamente projetados ou sobrecarregados com anomalias se inadequadamente modificados ao longo do tempo. Você pode ser solicitado a reprojetar e modificar bancos de dados existentes. Isso pode ser um grande empreendimento se as tabelas não estiverem adequadamente normalizadas.

