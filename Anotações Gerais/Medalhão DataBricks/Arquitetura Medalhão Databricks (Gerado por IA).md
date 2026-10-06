
## O que é?

A arquitetura medalhão é um padrão de design de dados desenvolvido pelo Databricks que organiza dados em um data lakehouse através de três camadas progressivas: Bronze, Silver e Gold. Cada camada representa um nível crescente de qualidade e refinamento dos dados, seguindo uma progressão lógica desde dados brutos até informações prontas para consumo analítico.

**Camada Bronze**: Armazena dados brutos em seu formato original, preservando o estado nativo das fontes de dados sem transformações significativas.

**Camada Silver**: Contém dados validados, limpos e enriquecidos, onde inconsistências são corrigidas e estruturas são padronizadas.

**Camada Gold**: Apresenta dados com regras de negócio aplicadas, agregações e otimizações específicas para casos de uso analíticos e relatórios.

## Para que serve?

A arquitetura medalhão serve para estabelecer uma pipeline de dados estruturada que garante qualidade progressiva e governança eficaz. Seus principais objetivos incluem:

- **Rastreabilidade completa**: Permite acompanhar a origem e transformação dos dados desde a fonte até o consumo final
- **Qualidade incremental**: Implementa validações e limpezas em etapas, facilitando identificação e correção de problemas
- **Reutilização de dados**: Diferentes consumidores podem acessar dados no nível de refinamento adequado às suas necessidades
- **Governança simplificada**: Facilita implementação de controles de acesso, auditoria e compliance
- **Eficiência operacional**: Reduz redundância de processamento e otimiza recursos computacionais

## Em que casos devo usar?

A arquitetura medalhão é recomendada especialmente em cenários que apresentam as seguintes características:

**Múltiplas fontes de dados heterogêneas**: Quando você precisa integrar dados de diferentes sistemas com formatos, qualidades e frequências de atualização distintas.

**Diferentes perfis de consumidores**: Quando cientistas de dados precisam de dados brutos para exploração, enquanto analistas de negócio necessitam de dados estruturados e confiáveis.

**Requisitos rigorosos de governança**: Em organizações que demandam auditoria, compliance regulatório ou controles rigorosos de qualidade de dados.

**Pipelines de dados complexas**: Quando existe necessidade de aplicar múltiplas transformações, validações e regras de negócio em sequência.

**Ambiente de alta escala**: Para organizações que processam grandes volumes de dados e precisam de arquitetura que suporte crescimento e otimização de performance.

**Ecossistema analítico diversificado**: Quando a mesma base de dados precisa alimentar relatórios executivos, modelos de machine learning e análises exploratórias simultaneamente.

A arquitetura medalhão não é necessária para casos simples com poucas fontes de dados, transformações básicas ou quando existe apenas um tipo de consumidor dos dados.