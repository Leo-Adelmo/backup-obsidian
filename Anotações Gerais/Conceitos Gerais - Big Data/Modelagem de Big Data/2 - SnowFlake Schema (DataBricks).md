artigo: https://www.databricks.com/br/glossary/snowflake-schema
## O que é um esquema snowflake?

Um **esquema em floco de neve** é um modelo de dados multidimensional e uma extensão do esquema em estrela, subdividindo as tabelas de dimensão em tabelas de subdimensão. Os esquemas em floco de neve são comumente usados para business intelligence e relatórios usando análise multidimensional em data warehouses, data marts e bancos de dados relacionais.

Em um esquema em floco de neve, os engenheiros subdividem cada tabela de dimensão em subdimensões lógicas. Isso deixa o modelo de dados mais complexo, mas torna mais fácil para os analistas trabalharem com determinados formatos de dados.

É chamado de esquema em floco de neve porque o Diagrama Entidade-Relacionamento (ERD) se parece com um floco de neve, conforme mostrado abaixo.

![[Pasted image 20250729205335.png]]

## Esquemas em floco de neve x esquemas em estrela

Semelhantes aos esquemas em estrela, os esquemas em floco de neve também possuem uma tabela de fatos central, conectada a várias tabelas de dimensão por meio de chaves estrangeiras. No entanto, ao contrário dos esquemas em estrela, os esquemas em floco de neve são mais normalizados.

Um esquema em floco de neve adere estritamente a um alto padrão de normalização e é eficiente em termos de armazenamento, mas tem um desempenho de tempo de query inferior ao de um modelo de dados desnormalizado. Por outro lado, modelos de dados desnormalizados, como esquemas em estrela, têm alta redundância de dados (duplicação de dados) e podem acelerar o desempenho da query em detrimento da duplicação de dados.

## Benefícios dos esquemas em floco de neve

- Recuperação rápida de dados
- Garantia da qualidade dos dados
- Modelo de dados simples e comum para armazenamento de dados

## Desvantagens dos esquemas em floco de neve

- Muita sobrecarga na configuração inicial
- Modelo de dados rígido
- Altos custos de manutenção