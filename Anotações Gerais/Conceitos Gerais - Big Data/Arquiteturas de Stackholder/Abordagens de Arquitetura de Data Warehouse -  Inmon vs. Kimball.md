artigo: https://medium.com/@goyalarchana17/data-warehouse-architecture-approaches-inmon-vs-kimball-0bd8f04bb5cf

O data warehousing é um componente crítico da engenharia de dados moderna, permitindo que organizações armazenem, gerenciem e analisem grandes volumes de dados. Duas abordagens proeminentes para arquitetura de data warehouse são aquelas propostas por Bill Inmon e Ralph Kimball.

Neste artigo, exploraremos as características principais, prós e contras, e aplicações do mundo real de ambas as arquiteturas de data warehouse de Inmon e Kimball.

## Data Warehouse de Inmon: Abordagem EDW (top-down)

Bill Inmon é frequentemente referido como o "pai do data warehousing". Sua abordagem é conhecida como Corporate Information Factory (CIF) ou Abordagem Top-Down.

Imagine-o dizendo: "Por que começar pequeno quando você pode começar com um plano mestre para todo o universo?" Sua arquitetura é toda sobre criar um repositório central de dados que pode fazer tudo.

A técnica de design de Bill Inmon constrói estrutura de entidade usando o modelo normalizado (até 3NF), reduzindo a redundância de dados tanto quanto possível. Como resultado, os requisitos de negócio são claramente identificados, e qualquer anomalia de atualização de dados é evitada.

Além disso, essa abordagem top-down para design de banco de dados tem a vantagem de ser resistente a mudanças de negócio e conter uma perspectiva dimensional de dados através dos data marts.

### Características Principais:

1. **Data Warehouse Corporativo:**
    
    - Inmon defende a construção de um data warehouse abrangente e centralizado que integra dados de toda a organização.
    - O foco está em criar uma única versão da verdade.
2. **Modelo de Dados Normalizado:**
    
    - O data warehouse é projetado usando um modelo normalizado (tipicamente 3NF), que reduz a redundância de dados.
    - Os dados são altamente estruturados e detalhados.
3. **Data Marts:**
    
    - Data marts são criados como subconjuntos do data warehouse. Eles são frequentemente desnormalizados e adaptados para linhas de negócio ou departamentos específicos.
    - Esses data marts obtêm dados exclusivamente do data warehouse central.
4. **Processo ETL:**
    
    - Um processo robusto de ETL (Extract, Transform, Load) é essencial para limpar, integrar e carregar dados no data warehouse centralizado antes de serem distribuídos para os data marts.
5. **Variante no Tempo e Não-Volátil:**
    
    - O data warehouse armazena dados históricos e é projetado para reter dados para análise de longo prazo.

### Prós:

- **Integridade dos Dados**: Dados normalizados reduzem redundância e garantem consistência dos dados.
- **Modelo de Dados Abrangente**: Fornece uma visão holística dos dados da organização.
- **Escalabilidade**: Pode lidar com grandes volumes de dados e consultas complexas.

### Contras:

- **Implementação Demorada**: Requer tempo e recursos significativos para projetar e implementar.
- **Complexidade**: Gerenciar e manter um data warehouse normalizado pode ser complexo.
- **Desempenho de Consulta Mais Lento**: Dados normalizados podem levar a desempenho de consulta mais lento comparado a estruturas desnormalizadas.

### Exemplo de Cenário:

Um banco multinacional segue a arquitetura de Inmon para criar um data warehouse centralizado que integra dados de suas várias filiais mundiais. Este data warehouse fornece uma fonte única da verdade para relatórios financeiros globais, enquanto data marts individuais são usados por escritórios regionais para análise e relatórios localizados.

## Data Warehouse de Kimball: Abordagem de data mart (bottom-up)

Iniciado por Ralph Kimball, o modelo de dados Kimball segue uma abordagem bottom-up para design de arquitetura de data warehouse na qual data marts são primeiro formados baseados em requisitos de negócio.

Ele é o tipo de cara que acredita em começar com vitórias rápidas e construir sobre elas. "Por que esperar pela enchilada inteira quando você pode ter um taco delicioso agora mesmo?" ele poderia dizer.

A abordagem "bottom-up" refere-se a um data warehouse como "uma cópia de dados transacionais especificamente arquitetada para consulta e análise", também conhecida como star schema. Neste método, um data mart é primeiro desenvolvido para fornecer as capacidades necessárias de relatório e análise para processos de negócio específicos (ou assuntos). Isso significa que, em contraste com a estratégia orientada por dados de Inmon, deve ser orientado por negócio.

### Características Principais:

1. **Modelo Dimensional:**
    
    - Kimball promove o uso de modelos dimensionais (star schemas e snowflake schemas), que são projetados para facilidade de uso e desempenho de consulta.
    - Os dados são organizados em tabelas de fatos e tabelas de dimensão.
2. **Data Marts:**
    
    - Kimball começa criando data marts que abordam processos de negócio específicos (por exemplo, vendas, inventário).
    - Esses data marts são posteriormente integrados em um data warehouse abrangente.
3. **Dimensões Conformadas:**
    
    - Dimensões são compartilhadas através dos data marts, garantindo consistência e integração.
4. **Processo ETL:**
    
    - Um processo ETL simplificado carrega dados nos modelos dimensionais, tornando o data warehouse mais acessível para usuários de negócio.
5. **Focado no Usuário:**
    
    - A abordagem de Kimball é projetada para ser amigável ao usuário, permitindo que usuários de negócio naveguem e consultem os dados facilmente.

### Prós:

- **Implementação Mais Rápida**: Data marts podem ser implementados rapidamente para atender necessidades imediatas de negócio.
- **Amigável ao Usuário**: O design de star schema é intuitivo e fácil para usuários finais entenderem e consultarem.
- **Flexibilidade**: Mais fácil de adaptar e estender o data warehouse conforme novos requisitos emergem.

### Contras:

- **Redundância de Dados**: Dados desnormalizados podem levar à redundância e aumento dos requisitos de armazenamento.
- **Inconsistências**: Potencial para inconsistências entre data marts se não gerenciados adequadamente.
- **Desafios de Integração**: Integrar data marts para formar um data warehouse coeso pode ser desafiador.

### Exemplo de Cenário:

Uma empresa de e-commerce usa a arquitetura de Kimball para construir data marts para diferentes processos de negócio como pedidos de clientes, inventário de produtos e tráfego do website. Esses data marts são integrados em um data warehouse maior com dimensões conformadas, permitindo análise abrangente através de diferentes áreas de negócio.

## Resumo e Conclusão

Se você é do Time Inmon ou Time Kimball, ambas as arquiteturas têm seus superpoderes. A abordagem de Inmon é como construir uma catedral — grandiosa, intrincada e atemporal. A abordagem de Kimball, por outro lado, é como montar uma obra-prima de Lego — flexível, divertida e amigável ao usuário. A escolha entre elas depende das necessidades e restrições da sua organização. Então, escolha seu herói sabiamente e embarque na sua aventura de data warehousing!

Entender essas duas arquiteturas icônicas de data warehouse pode ajudá-lo a escolher a abordagem certa para sua organização. Aproveitando os pontos fortes da metodologia de Inmon ou Kimball, você pode construir um data warehouse robusto que atende às suas necessidades de negócio e suporta tomada de decisão baseada em dados.

### Comparação Resumida:

|Aspecto|Inmon (Top-Down)|Kimball (Bottom-Up)|
|---|---|---|
|**Abordagem**|Corporativa, centralizada|Departamental, incremental|
|**Modelo de Dados**|Normalizado (3NF)|Dimensional (Star/Snowflake)|
|**Implementação**|Longa, complexa|Rápida, iterativa|
|**Foco**|Integridade e consistência|Usabilidade e performance|
|**Melhor para**|Grandes corporações|Projetos com necessidades imediatas|
|**Custo inicial**|Alto|Baixo|
|**Flexibilidade**|Menor|Maior|

A escolha entre essas abordagens deve considerar fatores como:

- Tamanho e complexidade da organização
- Urgência das necessidades de relatório
- Recursos disponíveis (tempo, dinheiro, expertise)
- Cultura organizacional
- Requisitos de governança de dados

Ambas as abordagens têm mérito e podem ser bem-sucedidas quando aplicadas no contexto apropriado.