artigo: https://www.keboola.com/blog/kimball-vs-inmon
# Kimball vs Inmon: Qual abordagem você deve escolher ao projetar sua arquitetura de data warehouse ?

Data warehouses são o repositório central de dados que permite às empresas consolidar dados, automatizar operações de dados e usar o repositório central para apoiar todos os relatórios, business intelligence (BI), análises e tomada de decisões em toda a empresa.

Mas projetar uma arquitetura de data warehouse pode ser bastante desafiador.

Desde questões de escalabilidade, custo total de propriedade, flexibilidade e acessibilidade, os padrões de design envolvidos na construção e manutenção do ambiente de data warehousing podem ser exigentes.

As duas principais abordagens para projetar a arquitetura do data warehouse são a abordagem Inmon vs a abordagem Kimball.

Neste artigo, vamos aprofundar o que cada autor propôs, mostrar as vantagens e desvantagens relativas de cada abordagem, e finalmente contrastar e comparar as duas abordagens para facilitar a vida do seu arquiteto de dados.

## Abordagem Kimball

### Vantagens da arquitetura Kimball

**Simplicidade e velocidade.** A arquitetura de Kimball é muito mais simples e rápida para projetar e configurar.

**Compreensível.** O modelo de dados dimensional é fácil de entender tanto para funcionários não-técnicos quanto técnicos.

**Relevância.** Ao contrário da arquitetura de Inmon, a abordagem bottom-up de Kimball torna todas as integrações de dados relevantes para as necessidades do negócio.

**Necessidades da equipe de engenharia.** A abordagem de Kimball precisa de menos engenheiros com habilidades técnicas menos especializadas tanto para configurar quanto para manter o data warehouse do que a abordagem de Inmon.

### Desvantagens da arquitetura Kimball

**Redundância de dados.** Como os dados são carregados em um modelo dimensional, há mais redundância de dados e, portanto, maior chance de erros.

**Nenhuma fonte única da verdade.** Os dados dentro do data warehouse são projetados e organizados em torno de data marts. Juntamente com a redundância de dados, a arquitetura de Kimball não mantém uma fonte única da verdade em toda a empresa.

**Menos flexível.** À medida que as necessidades de dados mudam, os requisitos de negócio se alteram e as fontes de dados de entrada modificam suas cargas úteis, a arquitetura de Kimball é menos flexível a mudanças e se adapta mais lentamente.

**Incompleta.** A abordagem de Kimball começa (e termina) nos processos críticos de negócio. Portanto, não oferece uma visão completa de 360 graus dos dados da empresa. Em vez disso, é útil para relatórios de negócio de áreas temáticas específicas.

## Abordagem Inmon

Bill Inmon, o pai do data warehouse, propôs uma abordagem diferente, top-down para arquitetar data warehouses:

1. **Comece com o modelo de dados corporativo.** Identifique todas as diferentes fontes de dados disponíveis para a empresa. (É por isso que esta abordagem também é chamada de Corporate Information Factory ou CIF - ela começa com os dados disponíveis para uma Empresa)
    
2. **A partir dos dados e compreensão das necessidades de negócio, identifique as entidades-chave** (cliente, produto, pedido, etc.) e suas relações mútuas.
    
3. **Use a estrutura de entidades para construir um modelo lógico detalhado.** O modelo lógico captura em grande detalhe todos os atributos de cada entidade, as relações mútuas e codependências entre entidades, etc. Em termos de modelagem de dados, o modelo lógico projeta esquemas lógicos de relacionamentos de entidades.
    
4. **A partir do modelo lógico, construa o modelo físico.** Use processos ETL para extrair dados de diferentes fontes, transformar dados e carregá-los em um modelo de dados normalizado. Cada modelo de dados normalizado mantém os dados na 3ª forma normal para evitar redundância. O modelo de dados normalizado é o núcleo do data warehouse.
    
5. **Construa data marts para departamentos específicos.** Os dados são acessados via data marts para todas as necessidades de relatórios, o data warehouse atua como uma fonte única da verdade.
    

### Vantagens da arquitetura Inmon

**Flexibilidade.** A abordagem de Inmon é muito mais flexível a mudanças nos requisitos de negócio e mais flexível a modificações nas fontes de dados. Por causa do design do processo ETL que leva a dados normalizados, a arquitetura de Inmon é mais adaptável. Os arquitetos mudam apenas algumas tabelas normalizadas, e a mudança é propagada downstream.

**Fonte única da verdade.** Por causa do modelo de dados normalizado, o data warehouse atua como uma fonte única da verdade em toda a empresa.

**Menos propenso a erros.** A normalização evita redundância de dados, então tanto os processos de engenharia quanto os processos analíticos são menos propensos a erros.

**Completude.** A abordagem de Inmon cobre todos os dados da Empresa, então todas as necessidades de relatórios são cobertas.

### Desvantagens da arquitetura Inmon

**Custo inicial de configuração e manutenção.** Configurar e manter a arquitetura de Inmon é muito mais caro do que o tempo e investimento necessários para a arquitetura de Kimball. Esquemas normalizados exibem maior complexidade tanto no design quanto na manutenção do que suas contrapartes desnormalizadas.

**Habilidades necessárias.** A abordagem de Inmon necessita engenheiros altamente qualificados, que são mais difíceis de encontrar e mais caros para manter na folha de pagamento.

**Mais ETL é necessário.** Ao separar data marts do data warehouse, mais processos ETL são necessários para construir os data marts, levando portanto a mais sobrecarga de engenharia.

## Comparação: Kimball vs Inmon

### Como decidir entre as arquiteturas Kimball e Inmon?

Tudo está nas trocas entre as vantagens e desvantagens comparativas.

**Kimball é a melhor escolha** se você quer ver resultados mais rápido, tem uma pequena equipe de engenheiros e prevê poucas mudanças nos requisitos de negócio. Caso contrário, a redundância de dados pode causar anomalias e custos de manutenção no futuro.

**Inmon é a escolha** para grandes empresas que desejam ver uma imagem completa dos dados da empresa, mesmo que a implantação do data warehouse vá custar mais e demorar mais do que a contraparte de Kimball.

## Ferramentas de Data Warehousing - Acelere suas implantações

Independentemente de qual metodologia está mais próxima do seu coração, utilize Keboola para acelerar sua engenharia de integração de dados.

Keboola é a plataforma de operações de dados end-to-end que permite automatizar seus pipelines de dados.

Oferece mais de 250 integrações entre fontes e bancos de dados, data warehouses e data marts. Use Keboola para acelerar sua implantação de data warehouse. Simplesmente automatize os trabalhos ETL que levam à sua arquitetura de escolha Kimball ou Inmon.