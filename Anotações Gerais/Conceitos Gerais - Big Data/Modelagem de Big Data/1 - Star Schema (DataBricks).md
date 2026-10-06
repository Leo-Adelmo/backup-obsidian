artigo: https://www.databricks.com/br/glossary/star-schema
## O que é um esquema em estrela?

Um **esquema em estrela** é um modelo multidimensional que organiza os dados em um banco de dados para torná-los mais fáceis de entender e analisar. Pode ser aplicado a data warehouses, bancos de dados, data marts e outras ferramentas. O design do esquema em estrela é otimizado para consultar grandes conjuntos de dados.

Introduzido por Ralph Kimball na década de 90, os esquemas em estrela armazenam dados, gerenciam o histórico e atualizam os dados com eficiência, reduzindo a duplicação de definições de negócios repetitivas e agregando e filtrando dados em data warehouses em alta velocidade.

![[Pasted image 20250729204830.png]]

## Tabelas de fatos e tabelas de dimensão

Os esquemas em estrela são usados para desnormalizar dados de negócios em dimensões (como tempo e produtos) e fatos (como valores e quantidades de transações).

No centro do esquema em estrela, está uma única tabela de fatos que contém os fatos do seu negócio (como valores e quantidades de transações). A tabela de fatos se conecta a várias outras tabelas de dimensões como tempo ou produto. Um esquema em estrela une duas ou mais tabelas de fatos e dimensões, permitindo que você divida seus dados da maneira que achar melhor.

## Desnormalização de dados

Os esquemas em estrela **desnormalizam** os dados adicionando colunas redundantes a algumas tabelas de dimensão para acelerar e simplificar a query e o trabalho com dados. Sua finalidade é permitir redundância (duplicação de dados) no modelo de dados para evitar operações de join computacionalmente caras e melhorar a velocidade das queries.

Nesse modelo, as tabelas de fatos são normalizadas; mas as de dimensões, não. Ou seja, os dados da tabela de fatos existem apenas na tabela de fatos, mas as tabelas de dimensões podem conter dados redundantes.

## Benefícios dos esquemas em estrela

- Modelos de fato/dimensionais, como esquemas em estrela, são **simples** de entender e implementar, tornando mais fácil para que os usuários finais encontrem os dados de que precisam. Também podem ser aplicados a data marts e outros recursos de dados.
- Em comparação com modelos normalizados, como esquemas em floco de neve, os esquemas em estrela dependem menos de joins para acessar dados, tornando-os **ideais para queries simples**.
- **Adaptam-se bem aos modelos OLAP.**
- Os esquemas em estrela tentam evitar joins computacionalmente caras, resultando em **melhor desempenho de query** em comparação com dados normalizados.

## Diferenças entre esquemas em estrela e 3NF (terceira forma normal)

**3NF (terceira forma normal)** é uma técnica para reduzir a redundância de dados por meio da normalização e é um padrão para bancos de dados considerados perfeitamente normalizados. Devido à normalização dos dados, é comum haver mais tabelas em comparação aos esquemas em estrela. Por outro lado, as queries tendem a ser mais complexas devido a mais joins entre tabelas grandes.