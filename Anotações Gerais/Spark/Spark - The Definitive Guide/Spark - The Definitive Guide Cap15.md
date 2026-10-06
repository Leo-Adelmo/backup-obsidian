# Capítulo 15. Como o Spark Executa em um Cluster

Até agora no livro, focamos nas propriedades do Spark como uma interface de programação. Discutimos como as APIs estruturadas pegam uma operação lógica, dividem-na em um plano lógico e convertem isso em um plano físico que consiste em operações de Resilient Distributed Dataset (RDD) que são executadas no cluster de máquinas. Este capítulo foca no que acontece quando o Spark executa esse código. Discutiremos isso de uma maneira independente de implementação — isso não depende nem do gerenciador de cluster que você está usando nem do código que está executando. No final das contas, todo código Spark é executado da mesma maneira.

Este capítulo aborda vários tópicos importantes:

- A arquitetura e componentes de uma Aplicação Spark
- O ciclo de vida de uma Aplicação Spark dentro e fora do Spark
- Propriedades importantes de execução de baixo nível, como pipelining
- O que é necessário para executar uma Aplicação Spark, como introdução ao Capítulo 16.

Vamos começar com a arquitetura.

## A Arquitetura de uma Aplicação Spark

No Capítulo 2, discutimos alguns dos componentes de alto nível de uma Aplicação Spark. Vamos revisá-los novamente:

**O driver do Spark**  
O driver é o processo "no banco do motorista" da sua Aplicação Spark. É o controlador da execução de uma Aplicação Spark e mantém todo o estado do cluster Spark (o estado e tarefas dos executores). Ele deve se comunicar com o gerenciador de cluster para obter recursos físicos e iniciar executores. No final das contas, é apenas um processo em uma máquina física responsável por manter o estado da aplicação em execução no cluster.

**Os executores do Spark**  
Os executores Spark são os processos que realizam as tarefas atribuídas pelo driver do Spark. Os executores têm uma responsabilidade principal: pegar as tarefas atribuídas pelo driver, executá-las e relatar seu estado (sucesso ou falha) e resultados. Cada Aplicação Spark tem seus próprios processos executores separados.

**O gerenciador de cluster**  
O Driver do Spark e os Executores não existem no vácuo, e é aí que entra o gerenciador de cluster. O gerenciador de cluster é responsável por manter um cluster de máquinas que executará sua(s) Aplicação(ões) Spark. De maneira um pouco confusa, um gerenciador de cluster terá suas próprias abstrações de "driver" (às vezes chamado de mestre) e "worker". A diferença principal é que estes estão vinculados a máquinas físicas em vez de processos (como são no Spark). A Figura 15-1 mostra uma configuração básica de cluster. A máquina à esquerda da ilustração é o Nó Driver do Gerenciador de Cluster. Os círculos representam processos daemon em execução e gerenciando cada um dos nós worker individuais. Ainda não há nenhuma Aplicação Spark em execução — estes são apenas os processos do gerenciador de cluster.

**Figura 15-1. Um driver de cluster e worker (ainda sem Aplicação Spark)**

Quando chega a hora de realmente executar uma Aplicação Spark, solicitamos recursos do gerenciador de cluster para executá-la. Dependendo de como nossa aplicação está configurada, isso pode incluir um lugar para executar o driver do Spark ou podem ser apenas recursos para os executores da nossa Aplicação Spark. Durante a execução da Aplicação Spark, o gerenciador de cluster será responsável por gerenciar as máquinas subjacentes nas quais nossa aplicação está sendo executada.

O Spark atualmente suporta três gerenciadores de cluster: um gerenciador de cluster standalone simples e integrado, Apache Mesos e Hadoop YARN. No entanto, esta lista continuará a crescer, então certifique-se de verificar a documentação para o seu gerenciador de cluster favorito.

Agora que abordamos os componentes básicos de uma aplicação, vamos examinar uma das primeiras escolhas que você precisará fazer ao executar suas aplicações: escolher o modo de execução.

## Modos de Execução

Um modo de execução dá a você o poder de determinar onde os recursos mencionados anteriormente estão fisicamente localizados quando você vai executar sua aplicação. Você tem três modos para escolher:

- Modo cluster
- Modo cliente
- Modo local

Vamos percorrer cada um deles em detalhes usando a Figura 15-1 como modelo. Na seção a seguir, retângulos com bordas sólidas representam o processo driver do Spark, enquanto aqueles com bordas pontilhadas representam os processos executores.

**Modo cluster**

O modo cluster é provavelmente a maneira mais comum de executar Aplicações Spark. No modo cluster, um usuário envia um JAR pré-compilado, script Python ou script R para um gerenciador de cluster. O gerenciador de cluster então inicia o processo driver em um nó worker dentro do cluster, além dos processos executores. Isso significa que o gerenciador de cluster é responsável por manter todos os processos relacionados à Aplicação Spark. A Figura 15-2 mostra que o gerenciador de cluster colocou nosso driver em um nó worker e os executores em outros nós worker.

**Figura 15-2. Modo cluster do Spark**

**Modo cliente**

O modo cliente é quase o mesmo que o modo cluster, exceto que o driver do Spark permanece na máquina cliente que enviou a aplicação. Isso significa que a máquina cliente é responsável por manter o processo driver do Spark, e o gerenciador de cluster mantém os processos executores. Na Figura 15-3, estamos executando a Aplicação Spark a partir de uma máquina que não está localizada no cluster. Essas máquinas são comumente chamadas de máquinas gateway ou nós de borda. Na Figura 15-3, você pode ver que o driver está sendo executado em uma máquina fora do cluster, mas os workers estão localizados em máquinas no cluster.

**Figura 15-3. Modo cliente do Spark**

**Modo local**

O modo local é uma diferença significativa dos dois modos anteriores: ele executa toda a Aplicação Spark em uma única máquina. Ele alcança paralelismo através de threads nessa única máquina. Esta é uma maneira comum de aprender Spark, testar suas aplicações ou experimentar iterativamente com desenvolvimento local. No entanto, não recomendamos usar o modo local para executar aplicações de produção.

## O Ciclo de Vida de uma Aplicação Spark (Fora do Spark)

Este capítulo até agora abordou o vocabulário necessário para discutir Aplicações Spark. Agora é hora de falar sobre o ciclo de vida geral das Aplicações Spark de "fora" do código Spark real. Faremos isso com um exemplo ilustrado de uma aplicação executada com `spark-submit` (introduzido no Capítulo 3). Assumimos que um cluster já está em execução com quatro nós, um driver (não um driver Spark, mas driver do gerenciador de cluster) e três nós worker. O gerenciador de cluster real não importa neste ponto: esta seção usa o vocabulário da seção anterior para percorrer passo a passo o ciclo de vida de uma Aplicação Spark, desde a inicialização até a saída do programa.

**NOTA**  
Esta seção também faz uso de ilustrações e segue a mesma notação que introduzimos anteriormente. Além disso, agora introduzimos linhas que representam comunicação de rede. Setas mais escuras representam comunicação pelo Spark ou processos relacionados ao Spark, enquanto linhas tracejadas representam comunicação mais geral (como comunicação de gerenciamento de cluster).

**Solicitação do Cliente**

O primeiro passo é você enviar uma aplicação real. Isso será um JAR ou biblioteca pré-compilada. Neste ponto, você está executando código em sua máquina local e vai fazer uma solicitação ao nó driver do gerenciador de cluster (Figura 15-4). Aqui, estamos explicitamente pedindo recursos apenas para o processo driver do Spark. Assumimos que o gerenciador de cluster aceita esta oferta e coloca o driver em um nó no cluster. O processo cliente que enviou o trabalho original sai e a aplicação está em execução no cluster.

**Figura 15-4. Solicitando recursos para um driver**

Para fazer isso, você executará algo como o seguinte comando em seu terminal:

```
./bin/spark-submit \
--class <main-class> \
--master <master-url> \
--deploy-mode cluster \
--conf <key>=<value> \
... # outras opções
<application-jar> \
[argumentos-da-aplicação]
```

**Lançamento**

Agora que o processo driver foi colocado no cluster, ele começa a executar o código do usuário (Figura 15-5). Este código deve incluir um SparkSession que inicializa um cluster Spark (por exemplo, driver + executores). O SparkSession subsequentemente se comunicará com o gerenciador de cluster (a linha mais escura), pedindo para lançar processos executores do Spark em todo o cluster (as linhas mais claras). O número de executores e suas configurações relevantes são definidos pelo usuário através dos argumentos de linha de comando na chamada original do spark-submit.

**Figura 15-5. Lançando a Aplicação Spark**

O gerenciador de cluster responde lançando os processos executores (assumindo que tudo corra bem) e envia as informações relevantes sobre suas localizações para o processo driver. Depois que tudo está conectado corretamente, temos um "Cluster Spark" como você provavelmente pensa hoje.

**Execução**

Agora que temos um "Cluster Spark", o Spark segue seu caminho executando código, como mostrado na Figura 15-6. O driver e os workers se comunicam entre si, executando código e movendo dados. O driver agenda tarefas em cada worker, e cada worker responde com o status dessas tarefas e sucesso ou falha. (Abordaremos esses detalhes em breve.)

**Figura 15-6. Execução da aplicação**

**Conclusão**

Após a conclusão de uma Aplicação Spark, o processo driver sai com sucesso ou falha (Figura 15-7). O gerenciador de cluster então desliga os executores nesse cluster Spark para o driver. Neste ponto, você pode ver o sucesso ou falha da Aplicação Spark pedindo esta informação ao gerenciador de cluster.

**Figura 15-7. Encerrando a aplicação**

## O Ciclo de Vida de uma Aplicação Spark (Dentro do Spark)

Acabamos de examinar o ciclo de vida de uma Aplicação Spark fora do código do usuário (basicamente a infraestrutura que suporta o Spark), mas é provavelmente mais importante falar sobre o que acontece dentro do Spark quando você executa uma aplicação. Este é o "código do usuário" (o código real que você escreve que define sua Aplicação Spark). Cada aplicação é composta por um ou mais trabalhos Spark. Os trabalhos Spark dentro de uma aplicação são executados em série (a menos que você use threading para lançar múltiplas ações em paralelo).

**O SparkSession**

O primeiro passo de qualquer Aplicação Spark é criar um SparkSession. Em muitos modos interativos, isso é feito para você, mas em uma aplicação, você deve fazê-lo manualmente.

Algum de seu código legado pode usar o padrão novo SparkContext. Isso deve ser evitado em favor do método builder no SparkSession, que instancia de forma mais robusta os Contextos Spark e SQL e garante que não haja conflito de contexto, dado que pode haver várias bibliotecas tentando criar uma sessão na mesma Aplicação Spark:

```scala
// Criando um SparkSession em Scala
import org.apache.spark.sql.SparkSession
val spark = SparkSession.builder().appName("Databricks Spark Example")
  .config("spark.sql.warehouse.dir", "/user/hive/warehouse")
  .getOrCreate()
```

```python
# Criando um SparkSession em Python
from pyspark.sql import SparkSession
spark = SparkSession.builder.master("local").appName("Word Count")\
  .config("spark.some.config.option", "some-value")\
  .getOrCreate()
```

Depois de ter um SparkSession, você deve ser capaz de executar seu código Spark. A partir do SparkSession, você pode acessar todos os contextos e configurações de baixo nível e legados, também. Observe que a classe SparkSession foi adicionada apenas no Spark 2.X. Códigos mais antigos que você pode encontrar criariam diretamente um SparkContext e um SQLContext para as APIs estruturadas.

**O SparkContext**

Um objeto SparkContext dentro do SparkSession representa a conexão com o cluster Spark. Esta classe é como você se comunica com algumas das APIs de nível mais baixo do Spark, como RDDs. É comumente armazenado como a variável sc em exemplos e documentação mais antigos. Através de um SparkContext, você pode criar RDDs, acumuladores e variáveis de broadcast, e pode executar código no cluster.

Na maioria dos casos, você não deve precisar inicializar explicitamente um SparkContext; você deve simplesmente ser capaz de acessá-lo através do SparkSession. Se você realmente quiser, você deve criá-lo da maneira mais geral, através do método getOrCreate:

```scala
// em Scala
import org.apache.spark.SparkContext
val sc = SparkContext.getOrCreate()
```

**O SPARKSESSION, SQLCONTEXT E HIVECONTEXT**

Em versões anteriores do Spark, o SQLContext e o HiveContext forneciam a capacidade de trabalhar com DataFrames e Spark SQL e eram comumente armazenados como a variável sqlContext em exemplos, documentação e código legado. Como ponto histórico, o Spark 1.X tinha efetivamente dois contextos. O SparkContext e o SQLContext. Cada um deles realizava coisas diferentes. O primeiro focava em controle mais refinado das abstrações centrais do Spark, enquanto o último focava nas ferramentas de nível mais alto como o Spark SQL. No Spark 2.X, a comunidade combinou as duas APIs no SparkSession centralizado que temos hoje. No entanto, ambas essas APIs ainda existem e você pode acessá-las via SparkSession. É importante notar que você nunca deve precisar usar o SQLContext e raramente precisa usar o SparkContext.

Depois de inicializar seu SparkSession, é hora de executar algum código. Como sabemos dos capítulos anteriores, todo código Spark se compila em RDDs. Portanto, na próxima seção, pegaremos algumas instruções lógicas (um trabalho DataFrame) e passaremos passo a passo pelo que acontece ao longo do tempo.

**Instruções Lógicas**

Como você viu no início do livro, o código Spark essencialmente consiste em transformações e ações. Como você constrói isso depende de você — seja através de SQL, manipulação de RDD de baixo nível ou algoritmos de aprendizado de máquina. Compreender como pegamos instruções declarativas como DataFrames e as convertemos em planos de execução físicos é um passo importante para entender como o Spark é executado em um cluster. Nesta seção, certifique-se de executar isso em um ambiente novo (um novo shell Spark) para acompanhar o trabalho, estágio e números de tarefas.

**Instruções lógicas para execução física**

Mencionamos isso na Parte II, mas vale a pena reiterar para que você possa entender melhor como o Spark pega seu código e realmente executa os comandos no cluster. Vamos percorrer mais código, linha por linha, explicar o que está acontecendo nos bastidores para que você possa sair com uma melhor compreensão de suas Aplicações Spark. Em capítulos posteriores, quando discutirmos monitoramento, realizaremos um rastreamento mais detalhado de um trabalho Spark através da UI do Spark. Neste exemplo atual, adotaremos uma abordagem mais simples. Vamos fazer um trabalho de três etapas: usando um DataFrame simples, vamos reparti-lo, realizar uma manipulação valor por valor e, em seguida, agregar alguns valores e coletar o resultado final.

**NOTA**  
Este código foi escrito e executado com o Spark 2.2 em Python (você obterá o mesmo resultado em Scala, então omitimos isso). O número de trabalhos provavelmente não mudará drasticamente, mas pode haver melhorias nas otimizações subjacentes do Spark que mudam as estratégias de execução física.

```python
# em Python
df1 = spark.range(2, 10000000, 2)
df2 = spark.range(2, 10000000, 4)
step1 = df1.repartition(5)
step12 = df2.repartition(6)
step2 = step1.selectExpr("id * 5 as id")
step3 = step2.join(step12, ["id"])
step4 = step3.selectExpr("sum(id)")
step4.collect() # 2500000000000
```

Quando você executa este código, podemos ver que sua ação desencadeia um trabalho Spark completo. Vamos dar uma olhada no plano de explicação para fundamentar nossa compreensão do plano de execução físico. Podemos acessar essas informações na guia SQL (depois de realmente executar uma consulta) na UI do Spark, também:

```
step4.explain()

== Physical Plan ==
*HashAggregate(keys=[], functions=[sum(id#15L)])
+- Exchange SinglePartition
   +- *HashAggregate(keys=[], functions=[partial_sum(id#15L)])
      +- *Project [id#15L]
         +- *SortMergeJoin [id#15L], [id#10L], Inner
            :- *Sort [id#15L ASC NULLS FIRST], false, 0
            :  +- Exchange hashpartitioning(id#15L, 200)
            :     +- *Project [(id#7L * 5) AS id#15L]
            :        +- Exchange RoundRobinPartitioning(5)
            :           +- *Range (2, 10000000, step=2, splits=8)
            +- *Sort [id#10L ASC NULLS FIRST], false, 0
               +- Exchange hashpartitioning(id#10L, 200)
                  +- Exchange RoundRobinPartitioning(6)
                     +- *Range (2, 10000000, step=4, splits=8)
```

O que você tem quando chama collect (ou qualquer ação) é a execução de um trabalho Spark que individualmente consiste em estágios e tarefas. Vá para localhost:4040 se você estiver executando isso em sua máquina local para ver a UI do Spark. Vamos acompanhar na guia "jobs" eventualmente passando para estágios e tarefas conforme avançamos para níveis de detalhe adicionais.

**Um Trabalho Spark**

Em geral, deve haver um trabalho Spark para uma ação. Ações sempre retornam resultados. Cada trabalho se divide em uma série de estágios, cujo número depende de quantas operações de shuffle precisam ocorrer.

Este trabalho se divide nos seguintes estágios e tarefas:

- Estágio 1 com 8 Tarefas
- Estágio 2 com 8 Tarefas
- Estágio 3 com 6 Tarefas
- Estágio 4 com 5 Tarefas
- Estágio 5 com 200 Tarefas
- Estágio 6 com 1 Tarefa

Espero que você esteja pelo menos um pouco confuso sobre como chegamos a esses números para que possamos dedicar um tempo para entender melhor o que está acontecendo!

**Estágios**

Estágios no Spark representam grupos de tarefas que podem ser executadas juntas para calcular a mesma operação em várias máquinas. Em geral, o Spark tentará empacotar o máximo de trabalho possível (ou seja, o máximo de transformações possível dentro do seu trabalho) no mesmo estágio, mas o mecanismo inicia novos estágios após operações chamadas shuffles. Um shuffle representa um reparticionamento físico dos dados — por exemplo, ordenar um DataFrame, ou agrupar dados que foram carregados de um arquivo por chave (o que requer enviar registros com a mesma chave para o mesmo nó). Esse tipo de reparticionamento requer coordenação entre executores para mover dados. O Spark inicia um novo estágio após cada shuffle e mantém o controle da ordem em que os estágios devem ser executados para calcular o resultado final.

No trabalho que olhamos anteriormente, os dois primeiros estágios correspondem ao range que você executa para criar seus DataFrames. Por padrão, quando você cria um DataFrame com range, ele tem oito partições. O próximo passo é o reparticionamento. Isso muda o número de partições ao fazer shuffle dos dados. Esses DataFrames são embaralhados em seis partições e cinco partições, correspondendo ao número de tarefas nos estágios 3 e 4.

Os estágios 3 e 4 são executados em cada um desses DataFrames e o final do estágio representa o join (um shuffle). De repente, temos 200 tarefas. Isso ocorre por causa de uma configuração do Spark SQL. O valor padrão de spark.sql.shuffle.partitions é 200, o que significa que quando há um shuffle executado durante a execução, ele produz 200 partições de shuffle por padrão. Você pode alterar esse valor, e o número de partições de saída mudará.

**DICA**  
Abordamos o número de partições em um pouco mais de detalhe no Capítulo 19 porque é um parâmetro tão importante. Este valor deve ser definido de acordo com o número de núcleos em seu cluster para garantir uma execução eficiente. Veja como defini-lo:

```
spark.conf.set("spark.sql.shuffle.partitions", 50)
```

Uma boa regra é que o número de partições deve ser maior que o número de executores em seu cluster, potencialmente por múltiplos fatores dependendo da carga de trabalho. Se você estiver executando código em sua máquina local, seria benéfico definir esse valor mais baixo porque é improvável que sua máquina local seja capaz de executar esse número de tarefas em paralelo. Este é mais um padrão para um cluster no qual pode haver muito mais núcleos de executor para usar. Independentemente do número de partições, todo esse estágio é calculado em paralelo. O resultado final agrega essas partições individualmente, leva-as todas para uma única partição antes de finalmente enviar o resultado final para o driver. Veremos essa configuração várias vezes ao longo desta parte do livro.

**Tarefas**

Estágios no Spark consistem em tarefas. Cada tarefa corresponde a uma combinação de blocos de dados e um conjunto de transformações que serão executadas em um único executor. Se houver uma grande partição em nosso conjunto de dados, teremos uma tarefa. Se houver 1.000 pequenas partições, teremos 1.000 tarefas que podem ser executadas em paralelo. Uma tarefa é apenas uma unidade de computação aplicada a uma unidade de dados (a partição). Particionar seus dados em um número maior de partições significa que mais podem ser executadas em paralelo. Isso não é uma panaceia, mas é um lugar simples para começar com a otimização.

## Detalhes de Execução

Tarefas e estágios no Spark têm algumas propriedades importantes que vale a pena revisar antes de encerrarmos este capítulo. Primeiro, o Spark automaticamente cria pipeline de estágios e tarefas que podem ser feitas juntas, como uma operação de map seguida por outra operação de map. Segundo, para todas as operações de shuffle, o Spark escreve os dados em armazenamento estável (por exemplo, disco) e pode reutilizá-los em vários trabalhos. Discutiremos esses conceitos porque eles aparecerão quando você começar a inspecionar aplicações através da UI do Spark.

**Pipelining**

Uma parte importante do que faz do Spark uma "ferramenta de computação em memória" é que, diferentemente das ferramentas que vieram antes dele (por exemplo, MapReduce), o Spark executa o máximo de etapas possível em um ponto no tempo antes de escrever dados na memória ou disco. Uma das principais otimizações que o Spark realiza é o pipelining, que ocorre no nível do RDD e abaixo. Com o pipelining, qualquer sequência de operações que alimentam dados diretamente umas nas outras, sem precisar movê-los entre nós, é colapsada em um único estágio de tarefas que fazem todas as operações juntas. Por exemplo, se você escrever um programa baseado em RDD que faz um map, depois um filter, depois outro map, isso resultará em um único estágio de tarefas que imediatamente leem cada registro de entrada, passam-no pela primeira função map, passam-no pela função filter e passam-no pela última função map, se necessário. Esta versão em pipeline da computação é muito mais rápida do que escrever os resultados intermediários na memória ou disco após cada etapa. O mesmo tipo de pipelining acontece para uma computação DataFrame ou SQL que faz um select, filter e select.

Do ponto de vista prático, o pipelining será transparente para você enquanto escreve uma aplicação — o tempo de execução do Spark o fará automaticamente — mas você o verá se alguma vez inspecionar sua aplicação através da UI do Spark ou através de seus arquivos de log, onde verá que várias operações RDD ou DataFrame foram colocadas em pipeline em um único estágio.

**Persistência de Shuffle**

A segunda propriedade que você às vezes verá é a persistência de shuffle. Quando o Spark precisa executar uma operação que tem que mover dados entre nós, como uma operação reduce-by-key (onde dados de entrada para cada chave precisam primeiro ser reunidos de muitos nós), o mecanismo não pode mais realizar pipelining e, em vez disso, realiza um shuffle entre redes. O Spark sempre executa shuffles primeiro fazendo com que as tarefas "fonte" (aquelas que enviam dados) escrevam arquivos de shuffle em seus discos locais durante seu estágio de execução. Em seguida, o estágio que faz o agrupamento e redução inicia e executa tarefas que buscam seus registros correspondentes de cada arquivo de shuffle e realiza essa computação (por exemplo, busca e processa os dados para um intervalo específico de chaves). Salvar os arquivos de shuffle no disco permite que o Spark execute este estágio posteriormente no tempo do que o estágio fonte (por exemplo, se não houver executores suficientes para executar ambos ao mesmo tempo) e também permite que o mecanismo reinicie tarefas de redução em caso de falha sem reexecutar todas as tarefas de entrada.

Um efeito colateral que você verá para a persistência de shuffle é que executar um novo trabalho sobre dados que já foram embaralhados não reexecuta o lado "fonte" do shuffle. Como os arquivos de shuffle já foram escritos no disco anteriormente, o Spark sabe que pode usá-los para executar os estágios posteriores do trabalho e não precisa refazer os anteriores. Na UI do Spark e nos logs, você verá os estágios pré-shuffle marcados como "ignorados". Esta otimização automática pode economizar tempo em uma carga de trabalho que executa vários trabalhos sobre os mesmos dados, mas, é claro, para um desempenho ainda melhor, você pode realizar seu próprio cache com o método cache do DataFrame ou RDD, que permite controlar exatamente quais dados são salvos e onde. Você rapidamente se acostumará com esse comportamento depois de executar algumas ações Spark em dados agregados e inspecioná-los na UI.

## Conclusão

Neste capítulo, discutimos o que acontece com as Aplicações Spark quando vamos executá-las em um cluster. Isso significa como o cluster realmente executará esse código, bem como o que acontece dentro das Aplicações Spark durante o processo. Neste ponto, você deve se sentir bastante confortável entendendo o que acontece dentro e fora de uma Aplicação Spark. Isso lhe dará um ponto de partida para depurar suas aplicações. O Capítulo 16 discutirá a escrita de Aplicações Spark e as coisas que você deve considerar ao fazê-lo.