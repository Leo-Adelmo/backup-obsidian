# As Três Primeiras Formas Normais

As formas normais são regras sistemáticas criadas por Edgar F. Codd para organizar dados em bancos de dados relacionais, eliminando problemas estruturais que causam inconsistências e desperdício de espaço. Cada forma normal resolve problemas específicos e deve ser aplicada sequencialmente.

## Primeira Forma Normal (1FN)

### Definição Completa

Uma tabela está na 1FN quando satisfaz todas estas condições:

- **Atomicidade**: Cada célula contém apenas um valor indivisível
- **Ausência de grupos repetitivos**: Não há colunas que representem conjuntos de valores similares
- **Unicidade de linhas**: Cada linha deve ser única
- **Ordem irrelevante**: A ordem das linhas não deve importar para o significado dos dados

### Conceitos-Chave

**Valor Atômico**: É um valor que não pode ser dividido em partes menores dentro do contexto do banco de dados. Por exemplo, um nome completo "João Silva" pode ser considerado atômico se você sempre trata o nome como uma unidade, mas deve ser dividido em "João" e "Silva" se você precisa trabalhar com nome e sobrenome separadamente.

**Grupos Repetitivos**: São colunas múltiplas que armazenam valores do mesmo tipo. Por exemplo: `Telefone1`, `Telefone2`, `Telefone3`.

### Exemplos Detalhados

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

### Definição Completa

Uma tabela está na 2FN quando:

- **Está na 1FN**
- **Não possui dependências funcionais parciais** da chave primária

### Conceitos-Chave

**Chave Primária Composta**: É uma chave primária formada por duas ou mais colunas. Por exemplo: `(PedidoID, ProdutoID)` em uma tabela de itens de pedido.

**Dependência Funcional**: Existe dependência funcional de A para B (A → B) quando, para cada valor de A, existe exatamente um valor correspondente de B. Por exemplo: `CPF → Nome` (um CPF determina unicamente um nome).

**Dependência Funcional Parcial**: Ocorre quando um atributo não-chave depende apenas de **parte** da chave primária composta, não da chave inteira.

### Exemplo Detalhado

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

### Definição Completa

Uma tabela está na 3FN quando:

- **Está na 2FN**
- **Não possui dependências funcionais transitivas**

### Conceitos-Chave

**Dependência Funcional Transitiva**: Ocorre quando um atributo não-chave A depende de outro atributo não-chave B, que por sua vez depende da chave primária. Matematicamente: se `Chave → B` e `B → A`, então existe dependência transitiva `Chave → B → A`.

### Exemplo Detalhado

**❌ Violação da 3FN:**

```
ClienteID | NomeCliente | CidadeCliente | EstadoCliente | RegiaoEstado | Populacao
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

## Tipos de Anomalias Eliminadas

### Anomalia de Inserção

**Problema**: Não conseguir inserir dados sem ter outros dados relacionados. **Exemplo**: Na tabela não normalizada, não posso cadastrar informações sobre um novo estado sem ter pelo menos um cliente desse estado.

### Anomalia de Atualização

**Problema**: Precisar alterar a mesma informação em múltiplos lugares. **Exemplo**: Se a região de um estado mudar, preciso atualizar todas as linhas de clientes desse estado.

### Anomalia de Exclusão

**Problema**: Perder informações importantes ao excluir dados. **Exemplo**: Se excluir o último cliente de um estado, perco as informações sobre esse estado.

## Processo de Normalização Passo a Passo

### Passo 1: Identifique as Dependências Funcionais

Liste todas as relações de dependência entre os atributos:

- `ClienteID → NomeCliente, CidadeCliente, EstadoCliente`
- `EstadoCliente → RegiaoEstado, PopulacaoEstado`

### Passo 2: Determine a Chave Primária

Identifique o conjunto mínimo de atributos que determina todos os outros.

### Passo 3: Aplique as Formas Normais Sequencialmente

- Primeiro 1FN (atomicidade)
- Depois 2FN (eliminar dependências parciais)
- Por último 3FN (eliminar dependências transitivas)

## Benefícios Detalhados da Normalização

### Benefícios de Armazenamento

- **Redução de redundância**: Cada dado é armazenado apenas uma vez
- **Economia de espaço**: Menos duplicação significa menos espaço em disco
- **Consistência referencial**: Relacionamentos entre tabelas garantem integridade

### Benefícios de Manutenção

- **Facilidade de atualização**: Mudanças em um local afetam todo o sistema
- **Redução de erros**: Menos lugares para atualizar significa menos chance de inconsistência
- **Flexibilidade**: Fácil adicionar novos tipos de dados sem reestruturar tudo

### Benefícios de Integridade

- **Prevenção de inconsistências**: Dados contraditórios são estruturalmente impossíveis
- **Aplicação de regras de negócio**: Constraints podem ser aplicadas de forma centralizada
- **Auditoria facilitada**: Mudanças são rastreáveis e controláveis

## Considerações Práticas e Limitações

### Quando Considerar Desnormalização

Em alguns cenários específicos, pode ser apropriado sacrificar normalização por performance:

1. **Consultas muito frequentes**: Se uma consulta que junta múltiplas tabelas é executada milhares de vezes por segundo
2. **Dados históricos**: Informações que nunca mudam podem ser armazenadas de forma redundante
3. **Relatórios complexos**: Tabelas de resumo podem ser mantidas para acelerar relatórios

### Ferramentas para Análise

- **Diagramas ER**: Visualizam relacionamentos entre entidades
- **Análise de dependências funcionais**: Identificam todas as relações de dependência
- **Ferramentas CASE**: Automatizam parte do processo de normalização

A normalização é uma ferramenta poderosa para criar estruturas de dados robustas e eficientes, mas deve ser aplicada com compreensão dos requisitos específicos de cada sistema.