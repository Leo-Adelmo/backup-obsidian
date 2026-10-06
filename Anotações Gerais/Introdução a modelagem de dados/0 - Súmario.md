# Introdução à Modelagem de Dados - Estrutura Completa

## **Artigo 1: Fundamentos da Modelagem de Dados**

### 1.1 O que é Modelagem de Dados

- Definição e importância
- Benefícios da modelagem adequada
- Consequências de má modelagem

### 1.2 Tipos de Modelos

- **Modelo Conceitual**
    - Visão de alto nível
    - Independente de tecnologia
    - Foco no negócio
- **Modelo Lógico**
    - Estrutura de dados detalhada
    - Independente de SGBD específico
    - Definição de relacionamentos
- **Modelo Físico**
    - Implementação específica do SGBD
    - Otimizações de performance
    - Índices, partições, etc.

### 1.3 Ciclo de Vida da Modelagem

- Levantamento de requisitos
- Modelagem conceitual
- Modelagem lógica
- Modelagem física
- Implementação e manutenção

### 1.4 Propriedades ACID

- **Atomicidade (Atomicity)**
    - Transações indivisíveis
    - Tudo ou nada
    - Exemplos práticos
- **Consistência (Consistency)**
    - Regras de integridade
    - Estados válidos do banco
    - Constraints e validações
- **Isolamento (Isolation)**
    - Transações concorrentes
    - Níveis de isolamento
    - Problemas de concorrência
- **Durabilidade (Durability)**
    - Persistência das transações
    - Recovery e backup
    - Garantias de armazenamento

### 1.5 Ferramentas de Modelagem

- ERwin, PowerDesigner, MySQL Workbench
- Lucidchart, Draw.io, dbdiagram.io
- Critérios de escolha

---

## **Artigo 2: Modelo Entidade-Relacionamento**

### 2.1 Conceitos Fundamentais

- **Entidades**
    - Definição e identificação
    - Entidades fortes vs. fracas
    - Conjunto de entidades
- **Atributos**
    - Simples vs. compostos
    - Univalorados vs. multivalorados
    - Armazenados vs. derivados
    - Atributos opcionais vs. obrigatórios

### 2.2 Chaves

- **Chave Primária**
    - Definição e propriedades
    - Chaves naturais vs. artificiais
    - Boas práticas
- **Chaves Candidatas**
    - Identificação e seleção
    - Múltiplas opções de chave
- **Chave Estrangeira**
    - Conceito e função
    - Integridade referencial

### 2.3 Relacionamentos

- **Definição e Tipos**
    - Binários, ternários, n-ários
    - Relacionamentos recursivos
    - Relacionamentos identificadores
- **Atributos de Relacionamento**
    - Quando usar
    - Exemplos práticos
    - Diferença de atributos de entidade

### 2.4 Cardinalidades

- **Tipos de Cardinalidade**
    - 1:1 (um para um)
    - 1:N (um para muitos)
    - N:N (muitos para muitos)
- **Participação**
    - Total (obrigatória)
    - Parcial (opcional)
- **Interpretação no Mundo Real**
    - Como determinar cardinalidades
    - Exemplos de diferentes domínios

### 2.5 Conceitos Avançados

- **Especialização/Generalização**
    - Herança em bancos de dados
    - Disjunção vs. sobreposição
    - Implementação física
- **Agregação**
    - Relacionamentos de relacionamentos
    - Casos de uso específicos

### 2.6 Diagramas ER

- **Notações**
    - Chen (clássica)
    - Crow's Foot (pé de galinha)
    - UML
- **Boas Práticas de Diagramação**
    - Layout e organização
    - Nomenclatura consistente
    - Documentação adequada

---

## **Artigo 3: Do Modelo ER ao Modelo Relacional**

### 3.1 Fundamentos do Modelo Relacional

- Conceitos de Codd
- Relações, tuplas, atributos
- Domínios e valores nulos

### 3.2 Regras de Mapeamento

- **Entidades → Tabelas**
    - Mapeamento direto
    - Definição de colunas
    - Chaves primárias
- **Relacionamentos 1:1**
    - Estratégias de implementação
    - Fusão de tabelas vs. tabelas separadas
- **Relacionamentos 1:N**
    - Chaves estrangeiras
    - Integridade referencial
- **Relacionamentos N:N**
    - Tabelas associativas
    - Chaves compostas
    - Atributos do relacionamento

### 3.3 Mapeamento de Conceitos Avançados

- **Entidades Fracas**
    - Chaves primárias compostas
    - Dependência existencial
- **Especialização/Generalização**
    - Estratégia de tabela única
    - Estratégia de tabelas separadas
    - Estratégia híbrida
- **Relacionamentos Ternários**
    - Decomposição em binários
    - Quando manter ternário

### 3.4 Constraints e Integridade

- **Integridade de Entidade**
    - Chaves primárias não nulas
    - Unicidade
- **Integridade Referencial**
    - Chaves estrangeiras
    - Ações em cascata
- **Integridade de Domínio**
    - Tipos de dados
    - Check constraints
- **Integridade Semântica**
    - Regras de negócio
    - Triggers e procedures

### 3.5 Índices Básicos

- Conceito e importância
- Índices primários vs. secundários
- Quando criar índices
- Impacto na performance

---

## **Artigo 4: Normalização**

### 4.1 Fundamentos da Normalização

- **Por que Normalizar?**
    - Problemas de redundância
    - Inconsistências de dados
    - Desperdício de espaço
- **Anomalias de Dados**
    - Anomalia de inserção
    - Anomalia de atualização
    - Anomalia de exclusão
    - Exemplos práticos

### 4.2 Dependências Funcionais

- **Conceitos Básicos**
    - Definição formal
    - Notação
    - Identificação de dependências
- **Tipos de Dependências**
    - Dependência total
    - Dependência parcial
    - Dependência transitiva
- **Algoritmos de Decomposição**
    - Preservação de dependências
    - Junção sem perdas

### 4.3 Formas Normais

- **Primeira Forma Normal (1FN)**
    - Atomicidade dos dados
    - Eliminação de grupos repetitivos
    - Chave primária obrigatória
    - Exemplos de desnormalização
- **Segunda Forma Normal (2FN)**
    - Dependência funcional total
    - Eliminação de dependências parciais
    - Aplicação apenas a chaves compostas
- **Terceira Forma Normal (3FN)**
    - Eliminação de dependências transitivas
    - Atributos não-chave dependem apenas da chave
    - Casos práticos
- **Forma Normal Boyce-Codd (BCNF)**
    - Refinamento da 3FN
    - Quando 3FN não é suficiente
    - Exemplos de aplicação

### 4.4 Formas Normais Avançadas

- **Quarta Forma Normal (4FN)**
    - Dependências multivaloradas
    - Casos específicos de aplicação
- **Quinta Forma Normal (5FN)**
    - Dependências de junção
    - Situações raras de aplicação

### 4.5 Desnormalização Controlada

- **Quando Desnormalizar**
    - Requisitos de performance
    - Padrões de acesso aos dados
    - Volume de dados
- **Estratégias de Desnormalização**
    - Colunas derivadas
    - Tabelas de resumo
    - Duplicação controlada
- **Gerenciamento de Redundância**
    - Triggers para sincronização
    - Batch jobs de atualização
    - Monitoramento de consistência

### 4.6 Ferramentas e Metodologias

- Algoritmos automáticos de normalização
- Análise de dependências funcionais
- Validação de formas normais

---

## **Artigo 5: Modelagem Dimensional - Star e Snowflake Schema**

### 5.1 Fundamentos da Modelagem Dimensional

- **OLTP vs. Modelagem Dimensional**
    - Diferentes objetivos
    - Padrões de acesso
    - Otimizações específicas
- **Conceitos Básicos**
    - Fatos vs. dimensões
    - Grão (granularidade)
    - Contexto de negócio

### 5.2 Tabelas Fato

- **Características**
    - Dados quantitativos (medidas)
    - Grande volume de registros
    - Chaves estrangeiras para dimensões
- **Tipos de Medidas**
    - Aditivas (soma, subtração)
    - Semi-aditivas (saldo, estoque)
    - Não-aditivas (percentuais, ratios)
- **Tipos de Fatos**
    - Fatos transacionais
    - Fatos de snapshot
    - Fatos acumulativos

### 5.3 Tabelas Dimensão

- **Características**
    - Dados descritivos
    - Relativamente pequenas
    - Desnormalizadas
- **Atributos Dimensionais**
    - Chave primária (surrogate key)
    - Atributos descritivos
    - Hierarquias naturais
- **Dimensões Especiais**
    - Dimensão tempo (obrigatória)
    - Dimensão degenerada
    - Dimensões conformes

### 5.4 Star Schema

- **Estrutura e Características**
    - Uma tabela fato central
    - Dimensões diretamente ligadas
    - Desnormalização das dimensões
- **Vantagens**
    - Consultas mais rápidas
    - Estrutura simples
    - Menos JOINs
- **Desvantagens**
    - Maior redundância
    - Manutenção mais complexa
    - Maior uso de espaço

### 5.5 Snowflake Schema

- **Estrutura e Características**
    - Dimensões normalizadas
    - Múltiplos níveis de hierarquia
    - Redução de redundância
- **Quando Usar**
    - Economia de espaço crítica
    - Dimensões muito grandes
    - Atualizações frequentes nas dimensões
- **Comparação com Star Schema**
    - Trade-offs performance vs. espaço
    - Complexidade de manutenção

### 5.6 Técnicas Avançadas

- **Slowly Changing Dimensions (SCD)**
    - Type 1: Sobrescrever
    - Type 2: Adicionar registro
    - Type 3: Adicionar atributo
    - Type 4: Tabela de histórico
- **Factless Fact Tables**
    - Eventos sem medidas
    - Relacionamentos N:N
    - Cobertura e elegibilidade
- **Bridge Tables**
    - Relacionamentos N:N entre fato e dimensão
    - Hierarquias irregulares
    - Agrupamentos **variáveis**

### 5.7 Galaxy Schema

- Múltiplas tabelas fato
- Dimensões compartilhadas
- Complexidade vs. flexibilidade

---

## **Artigo 6: Data Warehousing - Arquitetura e Processos**

### 6.1 Conceitos de Data Warehousing

- **Definições Fundamentais**
    - Data Warehouse vs. Database
    - Características: integrado, orientado por assunto, variante no tempo, não-volátil
- **Arquiteturas**
    - Top-down (Inmon)
    - Bottom-up (Kimball)
    - Híbrida
    - Data Vault

### 6.2 Componentes da Arquitetura

- **Staging Area**
    - Área de preparação
    - Limpeza e transformação inicial
    - Armazenamento temporário
- **Data Warehouse Core**
    - Integrated Data Store
    - Modelo dimensional ou normalizado
    - Metadados
- **Data Marts**
    - Subconjuntos especializados
    - Departamentais vs. funcionais
    - Dependentes vs. independentes

### 6.3 Processos ETL/ELT

- **Extract (Extração)**
    - Fontes heterogêneas
    - Full vs. incremental
    - CDC (Change Data Capture)
- **Transform (Transformação)**
    - Limpeza de dados
    - Padronização
    - Enriquecimento
    - Agregações
- **Load (Carga)**
    - Estratégias de carga
    - Janelas de manutenção
    - Validação e auditoria

### 6.4 Modelagem para DW

- **Estratégias de Modelagem**
    - Dimensional (Kimball)
    - Normalizada (Inmon)
    - Data Vault (Linstedt)
- **Data Vault Básico**
    - Hubs, Links, Satellites
    - Vantagens para ambientes ágeis
    - Auditoria completa

### 6.5 Gestão e Governança

- **Qualidade de Dados**
    - Profiling
    - Monitoring
    - Cleansing
- **Metadados**
    - Técnicos vs. de negócio
    - Linhagem de dados
    - Documentação
- **Segurança e Compliance**
    - Controle de acesso
    - Auditoria
    - LGPD/GDPR

---

## **Artigo 7: OLTP e OLAP**

### 7.1 Sistemas OLTP (Online Transaction Processing)

- **Características Fundamentais**
    - Transações em tempo real
    - Alta concorrência
    - Dados operacionais atuais
    - Propriedades ACID rigorosas
- **Padrões de Acesso**
    - Operações pontuais (CRUD)
    - Consultas simples e rápidas
    - Atualizações frequentes
- **Otimizações OLTP**
    - Índices para operações transacionais
    - Particionamento por operação
    - Connection pooling
    - Caching estratégico

### 7.2 Sistemas OLAP (Online Analytical Processing)

- **Características Fundamentais**
    - Consultas analíticas complexas
    - Dados históricos agregados
    - Poucos usuários simultâneos
    - Foco em leitura
- **Operações OLAP**
    - Drill-down (detalhar)
    - Roll-up (agregar)
    - Slice (fatiar)
    - Dice (segmentar)
    - Pivot (rotacionar)

### 7.3 Tipos de OLAP

- **MOLAP (Multidimensional)**
    - Cubos pré-calculados
    - Performance superior
    - Limitações de escalabilidade
- **ROLAP (Relational)**
    - Base em SGBD relacional
    - Maior flexibilidade
    - Dependente de otimizações SQL
- **HOLAP (Hybrid)**
    - Combina MOLAP e ROLAP
    - Agregações em MOLAP, detalhes em ROLAP
    - Balança performance e flexibilidade

### 7.4 Cubos OLAP

- **Estrutura Multidimensional**
    - Dimensões, hierarquias, níveis
    - Medidas e cálculos
    - Células e valores
- **Implementação Prática**
    - Microsoft SSAS
    - Oracle OLAP
    - Mondrian (open source)
    - Implementações em PostgreSQL

### 7.5 Comparação Detalhada

- **Arquitetura de Dados**
    - OLTP: normalizado, atual
    - OLAP: dimensional, histórico
- **Performance**
    - OLTP: transações rápidas
    - OLAP: consultas complexas otimizadas
- **Escalabilidade**
    - Diferentes estratégias
    - Diferentes gargalos
- **Casos de Uso**
    - Quando usar cada abordagem
    - Arquiteturas híbridas

### 7.6 Implementações Práticas

- **OLAP em SGBDs Relacionais**
    - PostgreSQL com extensões
    - SQL Server com funcões OLAP
    - Oracle Analytics
- **Ferramentas de BI**
    - Tableau, Power BI, Qlik
    - Integração com cubos
    - DirectQuery vs. Import

---

## **Artigo 8: Casos Práticos e Integração**

### 8.1 Projeto Completo: E-commerce

- **Fase 1: Modelo ER**
    - Entidades principais
    - Relacionamentos
    - Cardinalidades
- **Fase 2: Modelo Relacional**
    - Normalização 3FN
    - Implementação OLTP
    - Performance tuning
- **Fase 3: Data Warehouse**
    - Modelagem dimensional
    - Star schema
    - Processos ETL
- **Fase 4: Análises OLAP**
    - Cubos multidimensionais
    - Dashboards
    - Self-service BI

### 8.2 Migração e Evolução

- **De OLTP para OLAP**
    - Estratégias de migração
    - Mantendo sistemas legados
    - Sincronização de dados
- **Modernização de Arquitetura**
    - Cloud vs. on-premise
    - Real-time vs. batch
    - Microserviços e eventos

### 8.3 Escolha da Abordagem

- **Matriz de Decisão**
    - Volume de dados
    - Padrões de acesso
    - Recursos técnicos
    - Orçamento disponível
- **Anti-patterns Comuns**
    - OLAP over OLTP
    - Over-normalization
    - Under-normalization
    - Ignorar governança

### 8.4 Tendências Modernas

- **Data Lakes**
    - ELT vs. ETL
    - Schema-on-read
    - Big Data integration
- **Real-time Analytics**
    - Streaming data
    - Lambda architecture
    - Kappa architecture
- **Cloud Data Warehousing**
    - Snowflake, BigQuery, Redshift
    - Serverless analytics
    - Auto-scaling

### 8.5 Exercícios Práticos

- **Projeto Bancário**
    - Contas, transações, clientes
    - Compliance e auditoria
- **Sistema de RH**
    - Funcionários, departamentos, folha
    - Análises de performance
- **Projeto de Logística**
    - Produtos, estoques, movimentações
    - Otimização de supply chain