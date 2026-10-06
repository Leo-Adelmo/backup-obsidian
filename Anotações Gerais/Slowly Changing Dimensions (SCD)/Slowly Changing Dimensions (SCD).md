## Introdução

**Slowly Changing Dimension** (Dimensões de mudança lenta) são estruturas para atualizar e manter dados em tabelas de dimensões quando as dimensões mudam. Existem diferentes métodos para lidar com dimensões em mudança, chamados de "tipos" de dimensões de mudança lenta.
[Microsoft Fabric  – Implement slowly changing dimension](https://learn.microsoft.com/en-us/fabric/data-factory/slowly-changing-dimension-type-one)

## Tipos principais de SCD

![[Pasted image 20250923205413.png]]

### Tipo 0 – Imutável

No **SCD Tipo 0**, determinados atributos são considerados imutáveis. Uma vez carregados, permanecem fixos, independentemente de alterações futuras na fonte operacional. Esse modelo é simples, exige pouco esforço de implementação e é adequado para atributos que, de fato, não devem ser alterados (ex.: data de nascimento).  
[Airbyte – SCD Types](https://airbyte.com/data-engineering-resources/scd-types-in-data-warehouse)

### Tipo 1 – Sobrescrita

O **SCD Tipo 1** trata alterações sobrescrevendo o valor antigo pelo novo. Nesse caso, não há manutenção de histórico: apenas o estado atual é preservado. Esse modelo é útil quando as mudanças não precisam ser auditadas ou quando apenas o valor vigente interessa para análises. Sua principal limitação é a perda de informações históricas.  
[GeeksforGeeks – Slowly Changing Dimensions](https://www.geeksforgeeks.org/slowly-changing-dimensions/?utm_source=chatgpt.com)

### Tipo 2 – Histórico completo

O **SCD Tipo 2** preserva o histórico criando um novo registro a cada alteração. Para distinguir versões, utilizam-se chaves substitutas, colunas de vigência (data de início e fim) e indicadores de ativo. Esse tipo é amplamente adotado em cenários que exigem análises temporais detalhadas, pois permite reconstruir o estado de uma dimensão em qualquer ponto do tempo. A desvantagem é o maior consumo de espaço e a complexidade de consultas.  
[Oracle – Using Slowly Changing Dimensions](https://docs.oracle.com/cd/E10926_01/doc/owb.101/b12151/apascd.htm?utm_source=chatgpt.com)

### Tipo 3 – Versão limitada

O **SCD Tipo 3** mantém apenas um histórico limitado, adicionando uma coluna extra para armazenar o valor anterior de um atributo, enquanto a coluna principal guarda o valor atual. É adequado quando apenas uma comparação “antes e depois” é necessária, mas não atende cenários que exigem múltiplas versões históricas.  
[Airbyte – SCD Types](https://airbyte.com/data-engineering-resources/scd-types-in-data-warehouse?utm_source=chatgpt.com)

## Tipos híbridos

Os 3 tipos restantes de dimensões de mudança lenta são considerados híbridos porque combinam as dimensões mencionadas anteriormente. Embora pareçam oferecer o melhor dos mundos, o aumento da flexibilidade analítica costuma ser sobrecarregado com problemas de complexidade e manutenibilidade.

- Tipo 5 → Tipo 1 + Tipo 4
- Tipo 6 → Tipo 1 + Tipo 2 + Tipo 3
- Tipo 7 → Tipo 2 + visão atual
[Medium - Slowly changing dimensions in data warehousing](https://medium.com/%40zivilevalutytesilveira/slowly-changing-dimensions-in-data-warehousing-9479699bf40f)

## Elementos de implementação

A correta implementação de SCD depende de alguns elementos fundamentais:

- **Chaves substitutas** (surrogate keys) para diferenciar versões.
    
- **Datas de vigência e flags** para controle temporal.
    
- **Processos ETL/ELT** capazes de detectar alterações e aplicar a lógica correta de inserção ou atualização.
    

Essas práticas garantem integridade histórica e evitam inconsistências, como sobreposição de períodos de validade.  
[Medium – Slowly Changing Dimensions in Data Warehousing](https://medium.com/%40zivilevalutytesilveira/slowly-changing-dimensions-in-data-warehousing-9479699bf40f?utm_source=chatgpt.com)

## Cenários de uso

- Quando apenas o valor atual interessa, aplica-se o **Tipo 1**.
    
- Para análises temporais e auditorias, utiliza-se o **Tipo 2**.
    
- Se for suficiente manter apenas o valor anterior de um atributo, adota-se o **Tipo 3**.
    
- Em casos de maior complexidade, recorre-se a **modelos híbridos** (Tipos 4–7).
    

A escolha do modelo deve sempre refletir os requisitos de negócio e a estratégia analítica da organização.  
[Kimball Group – Slowly Changing Dimensions](https://www.kimballgroup.com/2008/08/slowly-changing-dimensions/?utm_source=chatgpt.com)

## Desafios

O uso de SCD apresenta desafios como:

- Crescimento do volume de dados em dimensões com muitas alterações.
    
- Consultas SQL mais complexas em cenários de histórico longo.
    
- Necessidade de estreita colaboração entre equipes de engenharia e negócio para definição do modelo adequado.
    

Esses pontos devem ser considerados no desenho arquitetural, evitando problemas de desempenho e governança de dados.  
[Oracle – Using Slowly Changing Dimensions](https://docs.oracle.com/cd/E10926_01/doc/owb.101/b12151/apascd.htm?utm_source=chatgpt.com)

## Conclusão

As Slowly Changing Dimensions constituem um dos conceitos centrais do design de Data Warehouses. Sua correta aplicação permite análises que conciliam estado atual e histórico de entidades de negócio, suportando desde relatórios operacionais até auditorias regulatórias. A escolha do tipo adequado não é universal, mas deve ser guiada por requisitos de negócio, tolerância a complexidade de consultas e disponibilidade de recursos de armazenamento e processamento.

---
