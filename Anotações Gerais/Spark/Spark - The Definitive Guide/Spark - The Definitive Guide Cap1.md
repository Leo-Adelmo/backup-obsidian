# Capítulo 1. O que é Apache Spark?

Apache Spark é um mecanismo de computação unificado e um conjunto de bibliotecas para processamento paralelo de dados em clusters de computadores. No momento desta escrita, Spark é o mecanismo de código aberto mais ativamente desenvolvido para essa tarefa, tornando-o uma ferramenta padrão para qualquer desenvolvedor ou cientista de dados interessado em big data. Spark suporta múltiplas linguagens de programação amplamente utilizadas (Python, Java, Scala e R), inclui bibliotecas para diversas tarefas, desde SQL até streaming e aprendizado de máquina, e executa em qualquer lugar, desde um laptop até um cluster de milhares de servidores. Isso o torna um sistema fácil de começar e escalar para processamento de big data ou em escala incrivelmente grande.

A Figura 1-1 ilustra todos os componentes e bibliotecas que o Spark oferece aos usuários finais.

Figura 1-1. O conjunto de ferramentas do Spark

Você notará que as categorias correspondem aproximadamente às diferentes partes deste livro. Isso não deveria ser surpresa; nosso objetivo aqui é educá-lo em todos os aspectos do Spark, e o Spark é composto por vários componentes diferentes.

Dado que você está lendo este livro, talvez já saiba um pouco sobre o Apache Spark e o que ele pode fazer. No entanto, neste capítulo, queremos abordar brevemente a filosofia predominante por trás do Spark, bem como o contexto em que foi desenvolvido (por que todos estão subitamente empolgados com o processamento paralelo de dados?) e sua história. Também delinearemos os primeiros passos para executar o Spark.

## A Filosofia do Apache Spark

Vamos decompor nossa descrição do Apache Spark — um mecanismo de computação unificado e conjunto de bibliotecas para big data — em seus componentes-chave:

### Unificado

O principal objetivo do Spark é oferecer uma plataforma unificada para escrever aplicações de big data. O que queremos dizer com unificado? O Spark é projetado para suportar uma ampla gama de tarefas de análise de dados, desde simples carregamento de dados e consultas SQL até aprendizado de máquina e computação em streaming, sobre o mesmo mecanismo de computação e com um conjunto consistente de APIs. O principal insight por trás desse objetivo é que tarefas de análise de dados do mundo real — sejam análises interativas em uma ferramenta como um notebook Jupyter, ou desenvolvimento tradicional de software para aplicações de produção — tendem a combinar muitos tipos diferentes de processamento e bibliotecas.

A natureza unificada do Spark torna essas tarefas mais fáceis e eficientes de escrever. Primeiro, o Spark fornece APIs consistentes e combináveis que você pode usar para construir uma aplicação a partir de peças menores ou a partir de bibliotecas existentes. Também facilita para você escrever suas próprias bibliotecas de análise sobre ele. No entanto, APIs combináveis não são suficientes: as APIs do Spark também são projetadas para permitir alto desempenho, otimizando entre as diferentes bibliotecas e funções combinadas em um programa de usuário. Por exemplo, se você carregar dados usando uma consulta SQL e depois avaliar um modelo de aprendizado de máquina sobre eles usando a biblioteca ML do Spark, o mecanismo pode combinar essas etapas em uma única varredura sobre os dados. A combinação de APIs gerais e execução de alto desempenho, não importa como você as combine, faz do Spark uma plataforma poderosa para aplicações interativas e de produção.

O foco do Spark em definir uma plataforma unificada é a mesma ideia por trás de plataformas unificadas em outras áreas de software. Por exemplo, os cientistas de dados se beneficiam de um conjunto unificado de bibliotecas (por exemplo, Python ou R) ao fazer modelagem, e os desenvolvedores web se beneficiam de frameworks unificados como Node.js ou Django. Antes do Spark, nenhum sistema de código aberto tentou fornecer esse tipo de mecanismo unificado para processamento paralelo de dados, o que significa que os usuários tinham que montar uma aplicação a partir de múltiplas APIs e sistemas. Assim, o Spark rapidamente se tornou o padrão para esse tipo de desenvolvimento. Ao longo do tempo, o Spark continuou a expandir suas APIs integradas para cobrir mais cargas de trabalho. Ao mesmo tempo, os desenvolvedores do projeto continuaram a refinar seu tema de um mecanismo unificado. Em particular, um dos principais focos deste livro serão as "APIs estruturadas" (DataFrames, Datasets e SQL) que foram finalizadas no Spark 2.0 para permitir uma otimização mais poderosa sob aplicações de usuário.

### Mecanismo de computação

Ao mesmo tempo em que o Spark busca a unificação, ele limita cuidadosamente seu escopo a um mecanismo de computação. Com isso, queremos dizer que o Spark lida com o carregamento de dados de sistemas de armazenamento e a realização de cálculos sobre eles, não o armazenamento permanente como um fim em si mesmo. Você pode usar o Spark com uma ampla variedade de sistemas de armazenamento persistente, incluindo sistemas de armazenamento em nuvem como Azure Storage e Amazon S3, sistemas de arquivos distribuídos como Apache Hadoop, armazenamentos de chave-valor como Apache Cassandra e barramentos de mensagens como Apache Kafka. No entanto, o Spark não armazena dados a longo prazo por si mesmo, nem favorece um sistema sobre outro. A motivação principal aqui é que a maioria dos dados já reside em uma mistura de sistemas de armazenamento. Os dados são caros para mover, então o Spark se concentra em realizar cálculos sobre os dados, não importa onde eles residam. Nas APIs voltadas para o usuário, o Spark trabalha arduamente para fazer esses sistemas de armazenamento parecerem amplamente semelhantes para que as aplicações não precisem se preocupar com o local onde seus dados estão.

O foco do Spark na computação o torna diferente das plataformas de software de big data anteriores, como o Apache Hadoop. O Hadoop incluía tanto um sistema de armazenamento (o sistema de arquivos Hadoop, projetado para armazenamento de baixo custo em clusters de servidores commodity) quanto um sistema de computação (MapReduce), que eram estreitamente integrados. No entanto, essa escolha torna difícil executar um dos sistemas sem o outro. Mais importante, essa escolha também torna desafiador escrever aplicações que acessem dados armazenados em qualquer outro lugar. Embora o Spark funcione bem no armazenamento Hadoop, hoje ele também é amplamente usado em ambientes para os quais a arquitetura Hadoop não faz sentido, como a nuvem pública (onde o armazenamento pode ser adquirido separadamente da computação) ou aplicações de streaming.

### Bibliotecas

O componente final do Spark são suas bibliotecas, que se baseiam em seu design como um mecanismo unificado para fornecer uma API unificada para tarefas comuns de análise de dados. O Spark suporta tanto bibliotecas padrão que acompanham o mecanismo quanto uma ampla variedade de bibliotecas externas publicadas como pacotes de terceiros pelas comunidades de código aberto. Hoje, as bibliotecas padrão do Spark são, na verdade, a maior parte do projeto de código aberto: o próprio mecanismo central do Spark mudou pouco desde que foi lançado pela primeira vez, mas as bibliotecas cresceram para fornecer cada vez mais tipos de funcionalidade. O Spark inclui bibliotecas para SQL e dados estruturados (Spark SQL), aprendizado de máquina (MLlib), processamento de stream (Spark Streaming e o mais recente Structured Streaming) e análise de grafos (GraphX). Além dessas bibliotecas, existem centenas de bibliotecas externas de código aberto, desde conectores para vários sistemas de armazenamento até algoritmos de aprendizado de máquina. Um índice de bibliotecas externas está disponível em spark-packages.org.

## Contexto: O Problema do Big Data

Por que precisamos de um novo mecanismo e modelo de programação para análise de dados em primeiro lugar? Como acontece com muitas tendências em computação, isso se deve a mudanças nos fatores econômicos subjacentes às aplicações e hardware de computador.

Durante a maior parte de sua história, os computadores ficavam mais rápidos a cada ano através de aumentos na velocidade do processador: os novos processadores a cada ano podiam executar mais instruções por segundo do que os do ano anterior. Como resultado, as aplicações também se tornavam automaticamente mais rápidas a cada ano, sem necessidade de alterações em seu código. Essa tendência levou a um ecossistema grande e estabelecido de aplicações se acumulando ao longo do tempo, a maioria das quais foi projetada para executar apenas em um único processador. Essas aplicações acompanharam a tendência de melhoria das velocidades do processador para escalar para cálculos maiores e volumes maiores de dados ao longo do tempo.

Infelizmente, essa tendência no hardware parou por volta de 2005: devido a limites rígidos na dissipação de calor, os desenvolvedores de hardware pararam de fazer processadores individuais mais rápidos e mudaram para adicionar mais núcleos de CPU paralelos, todos funcionando na mesma velocidade. Essa mudança significou que, de repente, as aplicações precisavam ser modificadas para adicionar paralelismo a fim de funcionar mais rapidamente, o que preparou o terreno para novos modelos de programação como o Apache Spark.

Além disso, as tecnologias para armazenar e coletar dados não diminuíram significativamente em 2005, quando as velocidades do processador diminuíram. O custo para armazenar 1 TB de dados continua a cair aproximadamente duas vezes a cada 14 meses, o que significa que é muito barato para organizações de todos os tamanhos armazenar grandes quantidades de dados. Além disso, muitas das tecnologias para coletar dados (sensores, câmeras, conjuntos de dados públicos, etc.) continuam a cair de preço e melhorar em resolução. Por exemplo, a tecnologia de câmeras continua a melhorar em resolução e a cair no custo por pixel a cada ano, a ponto de uma webcam de 12 megapixels custar apenas $3 a $4; isso tornou barato coletar uma ampla gama de dados visuais, seja de pessoas filmando vídeos ou sensores automatizados em um ambiente industrial. Além disso, as próprias câmeras são os principais sensores em outros dispositivos de coleta de dados, como telescópios e até máquinas de sequenciamento genético, reduzindo também o custo dessas tecnologias.

O resultado final é um mundo em que coletar dados é extremamente barato — muitas organizações hoje até consideram negligência não registrar dados de possível relevância para o negócio — mas processá-los requer cálculos grandes e paralelos, muitas vezes em clusters de máquinas. Além disso, neste novo mundo, o software desenvolvido nos últimos 50 anos não pode escalar automaticamente, nem os modelos de programação tradicionais para aplicações de processamento de dados, criando a necessidade de novos modelos de programação. É para este mundo que o Apache Spark foi construído.

## História do Spark

O Apache Spark começou na UC Berkeley em 2009 como o projeto de pesquisa Spark, que foi publicado pela primeira vez no ano seguinte em um artigo intitulado "Spark: Cluster Computing with Working Sets" por Matei Zaharia, Mosharaf Chowdhury, Michael Franklin, Scott Shenker e Ion Stoica do AMPlab da UC Berkeley. Na época, o Hadoop MapReduce era o mecanismo de programação paralela dominante para clusters, sendo o primeiro sistema de código aberto a lidar com processamento de dados paralelos em clusters de milhares de nós. O AMPlab trabalhou com vários usuários iniciais do MapReduce para entender os benefícios e desvantagens deste novo modelo de programação, e foi, portanto, capaz de sintetizar uma lista de problemas em vários casos de uso e começar a projetar plataformas de computação mais gerais. Além disso, Zaharia também trabalhou com usuários do Hadoop na UC Berkeley para entender suas necessidades para a plataforma — especificamente, equipes que estavam fazendo aprendizado de máquina em larga escala usando algoritmos iterativos que precisavam fazer várias passagens pelos dados.

Através dessas conversas, duas coisas ficaram claras. Primeiro, a computação em cluster tinha um tremendo potencial: em cada organização que usava MapReduce, aplicações totalmente novas podiam ser construídas usando os dados existentes, e muitos novos grupos começaram a usar o sistema após seus casos de uso iniciais. Segundo, no entanto, o mecanismo MapReduce tornava tanto desafiador quanto ineficiente construir grandes aplicações. Por exemplo, o algoritmo típico de aprendizado de máquina pode precisar fazer 10 ou 20 passagens pelos dados, e no MapReduce, cada passagem tinha que ser escrita como um trabalho MapReduce separado, que tinha que ser lançado separadamente no cluster e carregar os dados do zero.

Para resolver esse problema, a equipe do Spark primeiro projetou uma API baseada em programação funcional que poderia expressar sucintamente aplicações de várias etapas. A equipe então implementou essa API sobre um novo mecanismo que poderia realizar compartilhamento de dados em memória eficiente entre as etapas de computação. A equipe também começou a testar esse sistema com usuários tanto de Berkeley quanto externos.

A primeira versão do Spark suportava apenas aplicações em lote, mas logo outro caso de uso convincente ficou claro: ciência de dados interativa e consultas ad hoc. Simplesmente conectando o interpretador Scala ao Spark, o projeto poderia fornecer um sistema interativo altamente utilizável para executar consultas em centenas de máquinas. O AMPlab também rapidamente construiu sobre essa ideia para desenvolver o Shark, um mecanismo que poderia executar consultas SQL sobre o Spark e permitir o uso interativo por analistas, bem como cientistas de dados. O Shark foi lançado pela primeira vez em 2011.

Após esses lançamentos iniciais, rapidamente ficou claro que as adições mais poderosas ao Spark seriam novas bibliotecas, e assim o projeto começou a seguir a abordagem de "biblioteca padrão" que tem hoje. Em particular, diferentes grupos do AMPlab iniciaram o MLlib, o Spark Streaming e o GraphX. Eles também garantiram que essas APIs fossem altamente interoperáveis, permitindo a escrita de aplicações de big data completas no mesmo mecanismo pela primeira vez.

Em 2013, o projeto havia crescido para uso generalizado, com mais de 100 colaboradores de mais de 30 organizações fora da UC Berkeley. O AMPlab contribuiu com o Spark para a Apache Software Foundation como um lar de longo prazo e independente de fornecedores para o projeto. A equipe inicial do AMPlab também lançou uma empresa, a Databricks, para fortalecer o projeto, juntando-se à comunidade de outras empresas e organizações que contribuem para o Spark. Desde então, a comunidade Apache Spark lançou o Spark 1.0 em 2014 e o Spark 2.0 em 2016, e continua a fazer lançamentos regulares, trazendo novos recursos para o projeto.

Finalmente, a ideia central do Spark de APIs combináveis também foi refinada ao longo do tempo. As primeiras versões do Spark (antes da 1.0) definiram amplamente essa API em termos de operações funcionais — operações paralelas como mapas e reduções sobre coleções de objetos Java. A partir da versão 1.0, o projeto adicionou o Spark SQL, uma nova API para trabalhar com dados estruturados — tabelas com um formato de dados fixo que não está vinculado à representação em memória do Java. O Spark SQL permitiu novas otimizações poderosas entre bibliotecas e APIs, entendendo tanto o formato dos dados quanto o código do usuário que é executado nele com mais detalhes. Ao longo do tempo, o projeto adicionou uma infinidade de novas APIs que se baseiam nessa fundação estruturada mais poderosa, incluindo DataFrames, pipelines de aprendizado de máquina e Structured Streaming, uma API de streaming de alto nível, automaticamente otimizada. Neste livro, passaremos uma quantidade significativa de tempo explicando essas APIs de próxima geração, a maioria das quais é marcada como pronta para produção.

## O Presente e Futuro do Spark

O Spark existe há vários anos, mas continua a ganhar em popularidade e casos de uso. Muitos novos projetos dentro do ecossistema Spark continuam a expandir os limites do que é possível com o sistema. Por exemplo, um novo mecanismo de streaming de alto nível, o Structured Streaming, foi introduzido em 2016. Essa tecnologia é uma parte enorme de empresas que resolvem desafios de dados em escala massiva, desde empresas de tecnologia como Uber e Netflix usando ferramentas de streaming e aprendizado de máquina do Spark, até instituições como NASA, CERN e o Broad Institute do MIT e Harvard aplicando o Spark à análise de dados científicos.

O Spark continuará a ser uma pedra angular das empresas que fazem análise de big data no futuro próximo, especialmente considerando que o projeto ainda está se desenvolvendo rapidamente. Qualquer cientista de dados ou engenheiro que precise resolver problemas de big data provavelmente precisa de uma cópia do Spark em sua máquina — e, esperamos, uma cópia deste livro em sua estante!

## Executando o Spark

Este livro contém uma abundância de código relacionado ao Spark, e é essencial que você esteja preparado para executá-lo enquanto aprende. Na maior parte, você vai querer executar o código interativamente para que possa experimentar com ele. Vamos revisar algumas de suas opções antes de começarmos a trabalhar com as partes de codificação do livro.

Você pode usar o Spark a partir de Python, Java, Scala, R ou SQL. O próprio Spark é escrito em Scala e executa na Máquina Virtual Java (JVM), portanto, para executar o Spark em seu laptop ou em um cluster, tudo o que você precisa é de uma instalação do Java. Se você quiser usar a API Python, também precisará de um interpretador Python (versão 2.7 ou posterior). Se você quiser usar R, precisará de uma versão do R em sua máquina.

Existem duas opções que recomendamos para começar com o Spark: baixar e instalar o Apache Spark em seu laptop, ou executar uma versão baseada na web no Databricks Community Edition, um ambiente em nuvem gratuito para aprender Spark que inclui o código deste livro. Explicamos ambas as opções a seguir.

### Baixando o Spark Localmente

Se você quiser baixar e executar o Spark localmente, o primeiro passo é garantir que você tenha o Java instalado em sua máquina (disponível como java), bem como uma versão Python se você gostaria de usar Python. Em seguida, visite a página oficial de download do projeto, selecione o tipo de pacote "Pre-built for Hadoop 2.7 and later" e clique em "Direct Download". Isso baixa um arquivo TAR comprimido, ou tarball, que você precisará extrair. A maior parte deste livro foi escrita usando o Spark 2.2, então baixar a versão 2.2 ou posterior deve ser um bom ponto de partida.

#### Baixando o Spark para um cluster Hadoop

O Spark pode ser executado localmente sem qualquer sistema de armazenamento distribuído, como o Apache Hadoop. No entanto, se você gostaria de conectar a versão do Spark em seu laptop a um cluster Hadoop, certifique-se de baixar a versão correta do Spark para essa versão do Hadoop, que pode ser escolhida em http://spark.apache.org/downloads.html selecionando um tipo de pacote diferente. Discutiremos como o Spark é executado em clusters e no sistema de arquivos Hadoop em capítulos posteriores, mas neste ponto recomendamos apenas executar o Spark em seu laptop para começar.

> NOTA No Spark 2.2, os desenvolvedores também adicionaram a capacidade de instalar o Spark para Python via pip install pyspark. Essa funcionalidade surgiu quando este livro estava sendo escrito, então não pudemos incluir todas as instruções relevantes.

#### Construindo o Spark a partir do código-fonte

Não vamos cobrir isso no livro, mas você também pode construir e configurar o Spark a partir do código-fonte. Você pode selecionar um pacote de origem na página de download do Apache para obter apenas o código-fonte e seguir as instruções no arquivo README para construção.

Depois de baixar o Spark, você vai querer abrir um prompt de linha de comando e extrair o pacote. No nosso caso, estamos instalando o Spark 2.2. A seguir está um trecho de código que você pode executar em qualquer linha de comando estilo Unix para descompactar o arquivo que você baixou do Spark e mover para o diretório:

```
cd ~/Downloads
tar -xf spark-2.2.0-bin-hadoop2.7.tgz
cd spark-2.2.0-bin-hadoop2.7.tgz
```

Note que o Spark tem um grande número de diretórios e arquivos dentro do projeto. Não se intimide! A maioria desses diretórios é relevante apenas se você estiver lendo o código-fonte. A próxima seção cobrirá os diretórios mais importantes — aqueles que nos permitem iniciar os diferentes consoles do Spark para uso interativo.

### Iniciando os Consoles Interativos do Spark

Você pode iniciar um shell interativo no Spark para várias linguagens de programação diferentes. A maior parte deste livro é escrita com Python, Scala e SQL em mente; portanto, esses são nossos pontos de partida recomendados.

#### Iniciando o console Python

Você precisará do Python 2 ou 3 instalado para iniciar o console Python. No diretório inicial do Spark, execute o seguinte código:

```
./bin/pyspark
```

Depois de fazer isso, digite "spark" e pressione Enter. Você verá o objeto SparkSession impresso, que cobrimos no Capítulo 2.

#### Iniciando o console Scala

Para iniciar o console Scala, você precisará executar o seguinte comando:

```
./bin/spark-shell
```

Depois de fazer isso, digite "spark" e pressione Enter. Assim como em Python, você verá o objeto SparkSession, que cobrimos no Capítulo 2.

#### Iniciando o console SQL

Partes deste livro cobrirão uma grande quantidade de Spark SQL. Para essas partes, você pode querer iniciar o console SQL. Revisitaremos alguns dos detalhes mais relevantes depois de realmente cobrir esses tópicos no livro.

```
./bin/spark-sql
```

### Executando o Spark na Nuvem

Se você gostaria de ter uma experiência de notebook simples e interativa para aprender Spark, talvez prefira usar o Databricks Community Edition. A Databricks, como mencionamos anteriormente, é uma empresa fundada pela equipe de Berkeley que iniciou o Spark, e oferece uma edição comunitária gratuita de seu serviço de nuvem como um ambiente de aprendizado. O Databricks Community Edition inclui uma cópia de todos os dados e exemplos de código para este livro, tornando fácil executar qualquer um deles rapidamente.

Para usar o Databricks Community Edition, siga as instruções em https://github.com/databricks/Spark-The-Definitive-Guide. Você poderá usar Scala, Python, SQL ou R a partir de uma interface baseada em navegador web para executar e visualizar resultados.

### Dados Usados Neste Livro

Usaremos várias fontes de dados neste livro para nossos exemplos. Se você quiser executar o código localmente, pode baixá-los do repositório oficial de código deste livro conforme descrito em https://github.com/databricks/Spark-The-Definitive-Guide. Em resumo, você baixará os dados, colocará em uma pasta e depois executará os trechos de código neste livro.