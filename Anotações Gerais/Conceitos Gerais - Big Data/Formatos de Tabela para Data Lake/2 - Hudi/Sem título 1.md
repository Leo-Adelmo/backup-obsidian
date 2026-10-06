
---
## Tipos de Tabelas no Apache Hudi

O Apache Hudi oferece dois tipos principais de tabelas, cada uma com características distintas para otimizar operações de leitura e gravação:

### 1. Copy-on-Write (CoW)

- **Armazenamento**: Utiliza arquivos base em formato colunar (como Parquet) para armazenar os dados.
    
- **Gravações**: Cada atualização ou exclusão de registro resulta na criação de novos arquivos base, sem a utilização de arquivos de log.
    
- **Leitura**: As consultas leem diretamente os arquivos base, garantindo alto desempenho de leitura.
    
- **Vantagens**: Ideal para cargas de trabalho analíticas (OLAP) com alta taxa de leitura.
    
- **Desvantagens**: Operações de gravação podem ser mais lentas devido à necessidade de reescrever arquivos base.
    

### 2. Merge-on-Read (MoR)

- **Armazenamento**: Combina arquivos base com arquivos de log que registram atualizações e exclusões.
    
- **Gravações**: As alterações são registradas em arquivos de log e, periodicamente, esses logs são mesclados com os arquivos base.
    
- **Leitura**: Durante a consulta, os arquivos base e os logs são combinados dinamicamente para fornecer os dados mais recentes.
    
- **Vantagens**: Reduz a latência de gravação e suporta dados quase em tempo real.
    
- **Desvantagens**: O desempenho da leitura pode variar dependendo da compactação dos arquivos de log.
    

---

## Tipos de Consultas no Apache Hudi

O Hudi suporta diferentes tipos de consultas para atender a diversas necessidades de leitura:

### 1. Consultas Snapshot

- **Descrição**: Retornam o estado mais recente da tabela em um determinado ponto no tempo.
    
- **Uso**: Adequadas para obter uma visão consistente e atualizada dos dados.
    

### 2. Consultas Incrementais

- **Descrição**: Retornam apenas os dados que foram modificados desde a última consulta ou commit.
    
- **Uso**: Útil para pipelines de dados que precisam processar apenas as alterações recentes.
    

### 3. Consultas Otimizadas para Leitura

- **Descrição**: Focadas em fornecer o melhor desempenho de leitura, aproveitando índices e outras otimizações.
    
- **Uso**: Ideal para cargas de trabalho que exigem alta performance em consultas analíticas.
    

---

## Comparativo entre CoW e MoR

|Característica|Copy-on-Write (CoW)|Merge-on-Read (MoR)|
|---|---|---|
|Latência de Dados|Maior|Menor|
|Custo de Atualização|Maior|Menor|
|Tamanho do Arquivo|Menor|Maior|
|Amplificação de Gravação|Maior|Menor|

---

Se precisar de mais detalhes ou exemplos práticos sobre como configurar ou utilizar esses tipos de tabelas e consultas no Apache Hudi, estou à disposição para ajudar!