
As chaves são conceitos fundamentais em bancos de dados relacionais que servem para **identificar registros únicos** e **estabelecer relacionamentos** entre tabelas. Vou explicar detalhadamente cada tipo de chave:

## Chave Primária (Primary Key - PK)

### Definição

A chave primária é um atributo ou conjunto de atributos que **identifica unicamente** cada registro (linha) em uma tabela. É a principal forma de distinguir uma linha de todas as outras.

### Características Obrigatórias

- **Unicidade**: Cada valor deve ser único na tabela
- **Não-nulidade**: Não pode conter valores NULL (vazios)
- **Imutabilidade**: Idealmente não deve mudar após ser definida
- **Minimalidade**: Deve usar o menor número possível de atributos

### Exemplos Práticos

**Tabela Clientes:**

```
Cliente_ID (PK) | Nome      | Email
001             | João      | joao@email.com
002             | Maria     | maria@email.com
003             | Pedro     | pedro@email.com
```

**Tabela Produtos:**

```
Produto_ID (PK) | Nome_Produto | Preço
A001            | Notebook     | 2500.00
A002            | Mouse        | 50.00
B001            | Teclado      | 150.00
```

### Chave Primária Composta

Quando uma única coluna não é suficiente para garantir unicidade, usamos múltiplas colunas:

**Tabela Itens_Pedido:**

```
Pedido_ID (PK) | Produto_ID (PK) | Quantidade | Preço_Unitário
001            | A001            | 2          | 2500.00
001            | A002            | 1          | 50.00
002            | A001            | 1          | 2500.00
```

Aqui, nem `Pedido_ID` nem `Produto_ID` sozinhos garantem unicidade, mas juntos formam uma chave primária composta `(Pedido_ID, Produto_ID)`.

## Chave Estrangeira (Foreign Key - FK)

### Definição

A chave estrangeira é um atributo ou conjunto de atributos em uma tabela que **referencia a chave primária** de outra tabela (ou da mesma tabela). Ela estabelece e mantém a **integridade referencial** entre tabelas.

### Características

- **Deve corresponder a uma chave primária existente** na tabela referenciada
- **Pode conter valores NULL** (a menos que seja especificado o contrário)
- **Estabelece relacionamentos** entre tabelas
- **Mantém consistência** dos dados

### Exemplo Prático

**Tabela Pedidos:**

```
Pedido_ID (PK) | Data_Pedido | Cliente_ID (FK) | Total
001            | 2024-01-15  | 001             | 5050.00
002            | 2024-01-16  | 002             | 2500.00
003            | 2024-01-17  | 001             | 200.00
```

Aqui, `Cliente_ID` é uma chave estrangeira que referencia `Cliente_ID` na tabela Clientes. Isso significa:

- O pedido 001 foi feito pelo cliente João (ID 001)
- O pedido 002 foi feito pela cliente Maria (ID 002)
- O pedido 003 foi feito pelo cliente João (ID 001) novamente

### Integridade Referencial

A chave estrangeira garante que:

- **Não posso inserir** um pedido com `Cliente_ID = 999` se não existe cliente com ID 999
- **Não posso excluir** um cliente que possui pedidos associados (a menos que configure cascade)
- **Não posso alterar** um `Cliente_ID` para um valor inexistente

## Chave Candidata (Candidate Key)

### Definição

São **todos os atributos ou conjuntos de atributos** que poderiam servir como chave primária. Em outras palavras, são combinações que identificam unicamente cada registro.

### Exemplo

**Tabela Funcionários:**

```
Funcionario_ID | CPF         | Email               | Nome
001            | 12345678901 | joao@empresa.com   | João Silva
002            | 98765432100 | maria@empresa.com  | Maria Santos
```

**Chaves Candidatas:**

- `Funcionario_ID` (única por definição)
- `CPF` (única por natureza legal)
- `Email` (única por regra de negócio)

**Escolha da Chave Primária:** Geralmente escolhemos `Funcionario_ID` como chave primária por ser:

- Mais simples (um só atributo)
- Mais estável (não muda como email)
- Mais eficiente (numérica)

## Chave Alternativa (Alternate Key)

### Definição

São as **chaves candidatas que não foram escolhidas** como chave primária. Ainda mantêm a propriedade de unicidade.

No exemplo anterior:

- **Chave Primária:** `Funcionario_ID`
- **Chaves Alternativas:** `CPF`, `Email`

## Chave Surrogate vs Chave Natural

### Chave Natural

É derivada dos **dados reais do negócio**.

**Exemplos:**

- CPF em tabela de pessoas
- Placa em tabela de veículos
- ISBN em tabela de livros

### Chave Surrogate (Artificial)

É **criada artificialmente** pelo sistema, sem significado no mundo real.

**Exemplos:**

- IDs auto-incrementais (1, 2, 3, ...)
- UUIDs (identificadores únicos universais)
- Sequências numéricas

**Vantagens da Chave Surrogate:**

- **Estabilidade**: Nunca precisa mudar
- **Performance**: Geralmente numérica e pequena
- **Simplicidade**: Fácil de gerenciar
- **Independência**: Não depende de regras de negócio

## Super Chave (Super Key)

### Definição

É **qualquer conjunto de atributos** que identifica unicamente um registro. Inclui chaves candidatas mais atributos extras (desnecessários).

### Exemplo

**Tabela Produtos:**

```
Produto_ID | Nome_Produto | Código_Barras | Categoria
A001       | Notebook     | 123456789     | Eletrônicos
```

**Super Chaves:**

- `{Produto_ID}` (chave candidata)
- `{Código_Barras}` (chave candidata)
- `{Produto_ID, Nome_Produto}` (super chave - contém atributo extra)
- `{Produto_ID, Nome_Produto, Categoria}` (super chave - contém mais atributos extras)

## Aplicação Prática no Exemplo da Galeria de Arte

### Antes da Normalização

**Tabela Artist (Original):**

```
Artist_ID (PK) | FirstName | LastName | City      | State | Zip    | Country
001            | Pablo     | Picasso  | São Paulo | SP    | 01234  | Brasil
002            | Van       | Gogh     | São Paulo | SP    | 01234  | Brasil
```

**Problemas:**

- Dados de endereço duplicados
- `Zip → City, State, Country` (dependência transitiva)

### Após a Normalização

**Tabela Artist_Info:**

```
Artist_ID (PK) | FirstName | LastName | Zip (FK)
001            | Pablo     | Picasso  | 01234
002            | Van       | Gogh     | 01234
```

**Tabela Artist_Address:**

```
Zip (PK) | City      | State | Country
01234    | São Paulo | SP    | Brasil
87654    | Rio Janeiro| RJ   | Brasil
```

**Relacionamento:**

- `Artist_Info.Zip` é **chave estrangeira** que referencia `Artist_Address.Zip`
- `Artist_Address.Zip` é **chave primária** na tabela de endereços
- Elimina duplicação e mantém integridade

## Benefícios das Chaves

### Integridade dos Dados

- **Chaves primárias** garantem que não há registros duplicados
- **Chaves estrangeiras** garantem que relacionamentos são válidos

### Performance

- **Índices automáticos** são criados para chaves primárias
- **Consultas mais rápidas** usando relacionamentos

### Consistência

- **Regras de negócio** são aplicadas automaticamente
- **Atualizações em cascata** podem ser configuradas

### Exemplo de Integridade Referencial

```sql
-- Isto FUNCIONARÁ:
INSERT INTO Pedidos (Pedido_ID, Cliente_ID, Total) 
VALUES (004, 001, 1500.00);  -- Cliente 001 existe

-- Isto FALHARÁ:
INSERT INTO Pedidos (Pedido_ID, Cliente_ID, Total) 
VALUES (005, 999, 1500.00);  -- Cliente 999 NÃO existe

-- Isto FALHARÁ:
DELETE FROM Clientes WHERE Cliente_ID = 001;  -- Tem pedidos associados
```

As chaves são, portanto, os mecanismos fundamentais que garantem a **integridade**, **consistência** e **eficiência** dos bancos de dados relacionais, permitindo que as informações sejam organizadas de forma lógica e confiável.