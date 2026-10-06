# Resumo: Modelagem de Dados e Arquiteturas

## Modelagem

### O que é uma Entidade

Uma **entidade** é um objeto ou conceito do mundo real que pode ser identificado de forma única e sobre o qual desejamos armazenar informações em um banco de dados. Características principais:

- Representa algo concreto (pessoa, produto, pedido) ou abstrato (conceito, evento)
- Possui atributos que descrevem suas propriedades
- Tem uma chave primária que a identifica unicamente
- Exemplos: Cliente, Produto, Venda, Funcionário

### Entidade Relacionamento (ER)

O **Modelo Entidade-Relacionamento** é uma técnica de modelagem conceitual que representa a estrutura lógica de um banco de dados através de:

- **Entidades**: objetos principais do sistema
- **Atributos**: características das entidades
- **Relacionamentos**: associações entre entidades
- **Cardinalidade**: define quantas instâncias de uma entidade podem se relacionar com outra (1:1, 1:N, N:M)

Utiliza diagramas ER para visualizar graficamente a estrutura dos dados antes da implementação física.

## Modelagem de Big Data

### Modelagem Star Schema

Estrutura dimensional onde uma **tabela fato** central se conecta diretamente a múltiplas **tabelas dimensão**:

- **Tabela Fato**: contém métricas quantitativas e chaves estrangeiras
- **Tabelas Dimensão**: contêm atributos descritivos desnormalizados
- **Vantagens**: consultas simples, alta performance, fácil compreensão
- **Desvantagem**: maior redundância de dados
- **Uso ideal**: data warehouses com necessidade de consultas rápidas

### Modelagem Snowflake

Extensão do Star Schema onde as **tabelas dimensão são normalizadas** em múltiplos níveis:

- Dimensões são divididas em sub-tabelas relacionadas
- **Vantagens**: menor redundância, economia de espaço de armazenamento
- **Desvantagens**: consultas mais complexas, possível degradação de performance
- **Uso ideal**: quando o espaço de armazenamento é crítico e a complexidade é aceitável

## Arquitetura de Stakeholder

### Data Mart

**Subconjunto especializado** de um data warehouse focado em uma área específica do negócio:

- Atende necessidades de um departamento ou função específica
- Menor complexidade e custo comparado ao data warehouse completo
- Implementação mais rápida
- Pode ser dependente (derivado do DW) ou independente
- Exemplos: Data Mart de Vendas, Marketing, Finanças

### Data Mesh

**Arquitetura descentralizada** que trata dados como produtos, com propriedade distribuída:

- **Propriedade por domínio**: cada área de negócio possui seus dados
- **Dados como produto**: foco na qualidade e usabilidade
- **Infraestrutura self-service**: ferramentas padronizadas para todos os domínios
- **Governança federada**: políticas globais com autonomia local
- **Vantagem**: escalabilidade e agilidade organizacional
- **Desafio**: coordenação entre domínios

### Kimball (Bottom-Up)

**Metodologia dimensional** que constrói o data warehouse a partir de data marts:

- Inicia com data marts específicos por área de negócio
- Integra gradualmente os data marts em um data warehouse
- Foca na arquitetura dimensional (star/snowflake schema)
- **Vantagens**: entrega rápida de valor, menor risco, implementação incremental
- **Processo**: definir processos de negócio → identificar granularidade → escolher dimensões → identificar fatos

### Inmon (Top-Down)

**Metodologia normalizada** que constrói primeiro o data warehouse corporativo:

- Cria um data warehouse integrado e normalizado como fonte única da verdade
- Data marts são derivados posteriormente do DW central
- Enfatiza a integração e consistência dos dados corporativos
- **Vantagens**: visão única e consistente, melhor para relatórios corporativos
- **Desvantagens**: maior tempo de implementação, maior complexidade inicial
- **Processo**: análise corporativa → modelagem normalizada → implementação do DW → criação de data marts

---

## Comparação das Abordagens

|Aspecto|Kimball|Inmon|
|---|---|---|
|Direção|Bottom-up|Top-down|
|Modelagem|Dimensional|Normalizada|
|Implementação|Incremental|Monolítica|
|Tempo para valor|Rápido|Longo|
|Complexidade inicial|Baixa|Alta|
|Consistência|Gradual|Imediata|