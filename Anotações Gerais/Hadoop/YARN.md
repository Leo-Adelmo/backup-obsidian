
O Apache YARN (Yet Another Resource Negotiator) é o sistema de gerenciamento de recursos de cluster do Hadoop. O YARN foi introduzido no Hadoop 2 para melhorar a implementação do MapReduce, mas é suficientemente genérico para suportar também outros paradigmas de computação distribuída.

O YARN fornece APIs para solicitar e trabalhar com recursos de cluster, mas essas APIs normalmente não são usadas diretamente pelo código do usuário. Em vez disso, os usuários escrevem para APIs de nível superior fornecidas por frameworks de computação distribuída, que são construídos sobre o YARN e ocultam os detalhes de gerenciamento de recursos do usuário. A situação é ilustrada na Figura 4-1, que mostra alguns frameworks de computação distribuída (MapReduce, Spark e assim por diante) executando como aplicações YARN na camada de computação do cluster (YARN) e na camada de armazenamento do cluster (HDFS e HBase).
![[Pasted image 20250415232729.png]]
Figura 4-1. Aplicações YARN

Também existe uma camada de aplicações que se baseiam nos frameworks mostrados na Figura 4-1. Pig, Hive e Crunch são exemplos de frameworks de processamento que rodam sobre MapReduce, Spark ou Tez (ou nos três), e não interagem diretamente com o YARN.

Este capítulo percorre as funcionalidades do YARN e fornece uma base para entender capítulos posteriores na Parte IV que cobrem os frameworks de processamento distribuído do Hadoop.

## Anatomia da Execução de uma Aplicação YARN

O YARN fornece seus serviços principais através de dois tipos de daemons de longa duração: um gerenciador de recursos (um por cluster) para gerenciar o uso de recursos em todo o cluster, e gerenciadores de nós executando em todos os nós do cluster para iniciar e monitorar contêineres. Um contêiner executa um processo específico da aplicação com um conjunto restrito de recursos (memória, CPU e assim por diante). Dependendo de como o YARN está configurado (veja "YARN" na página 300), um contêiner pode ser um processo Unix ou um cgroup Linux. A Figura 4-2 ilustra como o YARN executa uma aplicação.

Figura 4-2. Como o YARN executa uma aplicação

Para executar uma aplicação no YARN, um cliente contacta o gerenciador de recursos e solicita a execução de um processo mestre de aplicação (etapa 1 na Figura 4-2). O gerenciador de recursos então encontra um gerenciador de nó que pode lançar o mestre da aplicação em um contêiner (etapas 2a e 2b). O que exatamente o mestre da aplicação faz depois de estar em execução depende da aplicação. Ele poderia simplesmente executar um cálculo no contêiner em que está executando e retornar o resultado ao cliente. Ou poderia solicitar mais contêineres do gerenciador de recursos (etapa 3) e usá-los para executar uma computação distribuída (etapas 4a e 4b). Este último é o que a aplicação YARN MapReduce faz, que veremos com mais detalhes em "Anatomia da Execução de um Job MapReduce" na página 185.

Observe na Figura 4-2 que o próprio YARN não fornece nenhuma maneira para as partes da aplicação (cliente, mestre, processo) se comunicarem entre si. A maioria das aplicações YARN não triviais usa alguma forma de comunicação remota (como a camada RPC do Hadoop) para passar atualizações de status e resultados de volta ao cliente, mas estes são específicos da aplicação.

## Requisições de Recursos

O YARN tem um modelo flexível para fazer solicitações de recursos. Uma solicitação para um conjunto de contêineres pode expressar a quantidade de recursos computacionais necessários para cada contêiner (memória e CPU), bem como restrições de localidade para os contêineres nessa solicitação.

A localidade é crítica para garantir que algoritmos de processamento de dados distribuídos usem a largura de banda do cluster de forma eficiente, por isso o YARN permite que uma aplicação especifique restrições de localidade para os contêineres que está solicitando. Restrições de localidade podem ser usadas para solicitar um contêiner em um nó específico ou rack, ou em qualquer lugar do cluster (fora do rack).

Às vezes, a restrição de localidade não pode ser atendida, caso em que ou nenhuma alocação é feita ou, opcionalmente, a restrição pode ser flexibilizada. Por exemplo, se um nó específico foi solicitado, mas não é possível iniciar um contêiner nele (porque outros contêineres estão sendo executados nele), então o YARN tentará iniciar um contêiner em um nó no mesmo rack ou, se isso não for possível, em qualquer nó do cluster.

No caso comum de lançar um contêiner para processar um bloco HDFS (para executar uma tarefa de mapeamento no MapReduce, por exemplo), a aplicação solicitará um contêiner em um dos nós que hospedam as três réplicas do bloco, ou em um nó em um dos racks que hospedam as réplicas, ou, na falta disso, em qualquer nó do cluster.

Uma aplicação YARN pode fazer solicitações de recursos a qualquer momento enquanto estiver em execução. Por exemplo, uma aplicação pode fazer todas as suas solicitações de uma vez, ou pode adotar uma abordagem mais dinâmica, solicitando mais recursos dinamicamente para atender às necessidades em constante mudança da aplicação.

O Spark adota a primeira abordagem, iniciando um número fixo de executores no cluster (veja "Spark no YARN" na página 571). O MapReduce, por outro lado, tem duas fases: os contêineres de tarefas de mapeamento são solicitados antecipadamente, mas os contêineres de tarefas de redução não são iniciados até mais tarde. Além disso, se alguma tarefa falhar, contêineres adicionais serão solicitados para que as tarefas com falha possam ser reexecutadas.

## Tempo de Vida da Aplicação

O tempo de vida de uma aplicação YARN pode variar drasticamente: de uma aplicação de curta duração de alguns segundos a uma aplicação de longa duração que executa por dias ou mesmo meses. Em vez de olhar para quanto tempo a aplicação executa, é útil categorizar aplicações em termos de como elas se mapeiam para os trabalhos que os usuários executam. O caso mais simples é uma aplicação por trabalho do usuário, que é a abordagem que o MapReduce adota.

O segundo modelo é executar uma aplicação por fluxo de trabalho ou sessão de usuário de trabalhos (possivelmente não relacionados). Essa abordagem pode ser mais eficiente que a primeira, pois os contêineres podem ser reutilizados entre trabalhos, e também há o potencial para armazenar em cache dados intermediários entre trabalhos. O Spark é um exemplo que usa esse modelo.

O terceiro modelo é uma aplicação de longa duração que é compartilhada por diferentes usuários. Tal aplicação frequentemente atua em algum tipo de função de coordenação. Por exemplo, o Apache Slider tem um mestre de aplicação de longa duração para lançar outras aplicações no cluster. Essa abordagem também é usada pelo Impala (veja "Alternativas SQL-on-Hadoop" na página 484) para fornecer uma aplicação proxy com a qual os daemons do Impala se comunicam para solicitar recursos do cluster. O mestre da aplicação "sempre ativo" significa que os usuários têm respostas de latência muito baixa às suas consultas, pois o overhead de iniciar um novo mestre de aplicação é evitado.

## Construindo Aplicações YARN

Escrever uma aplicação YARN do zero é bastante envolvente, mas em muitos casos não é necessário, pois muitas vezes é possível usar uma aplicação existente que se encaixa no necessário. Por exemplo, se você está interessado em executar um grafo acíclico dirigido (DAG) de trabalhos, então Spark ou Tez é apropriado; ou para processamento de fluxo, Spark, Samza ou Storm funcionam.

Existem alguns projetos que simplificam o processo de construção de uma aplicação YARN. O Apache Slider, mencionado anteriormente, torna possível executar aplicações distribuídas existentes no YARN. Os usuários podem executar suas próprias instâncias de uma aplicação (como HBase) em um cluster, independentemente de outros usuários, o que significa que diferentes usuários podem executar diferentes versões da mesma aplicação. O Slider fornece controles para alterar o número de nós em que uma aplicação está sendo executada e para suspender e depois retomar uma aplicação em execução.

O Apache Twill é semelhante ao Slider, mas, além disso, fornece um modelo de programação simples para o desenvolvimento de aplicações distribuídas no YARN. O Twill permite que você defina processos de cluster como uma extensão de um Java Runnable, e depois os execute em contêineres YARN no cluster. O Twill também fornece suporte para, entre outras coisas, registro em tempo real (eventos de log dos runnables são transmitidos de volta ao cliente) e mensagens de comando (enviadas do cliente para os runnables).

Nos casos em que nenhuma dessas opções é suficiente - como uma aplicação que tem requisitos de agendamento complexos - então o aplicativo distributed shell, que faz parte do próprio projeto YARN, serve como um exemplo de como escrever uma aplicação YARN. Ele demonstra como usar as APIs de cliente do YARN para lidar com a comunicação entre o cliente ou mestre da aplicação e os daemons do YARN.

## YARN Comparado ao MapReduce 1

A implementação distribuída do MapReduce na versão original do Hadoop (versão 1 e anteriores) às vezes é chamada de "MapReduce 1" para distingui-la do MapReduce 2, a implementação que usa YARN (no Hadoop 2 e posteriores).

É importante perceber que as APIs antigas e novas do MapReduce não são a mesma coisa que as implementações MapReduce 1 e MapReduce 2. As APIs são recursos voltados para o usuário do lado do cliente e determinam como você escreve programas MapReduce (veja Apêndice D), enquanto as implementações são apenas maneiras diferentes de executar programas MapReduce. Todas as quatro combinações são suportadas: tanto as APIs antigas quanto as novas do MapReduce rodam tanto no MapReduce 1 quanto no 2.

No MapReduce 1, existem dois tipos de daemon que controlam o processo de execução do trabalho: um jobtracker e um ou mais tasktrackers. O jobtracker coordena todos os trabalhos executados no sistema, agendando tarefas para serem executadas nos tasktrackers. Os tasktrackers executam tarefas e enviam relatórios de progresso para o jobtracker, que mantém um registro do progresso geral de cada trabalho. Se uma tarefa falhar, o jobtracker pode reagendá-la em um tasktracker diferente.

No MapReduce 1, o jobtracker cuida tanto do agendamento de trabalhos (combinando tarefas com tasktrackers) quanto do monitoramento do progresso das tarefas (acompanhando as tarefas, reiniciando tarefas com falha ou lentas e realizando a contabilidade das tarefas, como manter os totais de contadores). Em contraste, no YARN essas responsabilidades são tratadas por entidades separadas: o gerenciador de recursos e um mestre de aplicação (um para cada trabalho MapReduce). O jobtracker também é responsável por armazenar o histórico de trabalhos concluídos, embora seja possível executar um servidor de histórico de trabalhos como um daemon separado para tirar a carga do jobtracker. No YARN, a função equivalente é o servidor de linha do tempo, que armazena o histórico da aplicação.

O equivalente YARN de um tasktracker é um gerenciador de nó. O mapeamento é resumido na Tabela 4-1.

Tabela 4-1. Uma comparação dos componentes do MapReduce 1 e YARN

|MapReduce 1|YARN|
|---|---|
|Jobtracker|Gerenciador de recursos, mestre de aplicação, servidor de linha do tempo|
|Tasktracker|Gerenciador de nó|
|Slot|Contêiner|

O YARN foi projetado para abordar muitas das limitações do MapReduce 1. Os benefícios de usar o YARN incluem:

### Escalabilidade

O YARN pode ser executado em clusters maiores que o MapReduce 1. O MapReduce 1 atinge gargalos de escalabilidade na região de 4.000 nós e 40.000 tarefas, decorrentes do fato de que o jobtracker precisa gerenciar tanto os trabalhos quanto as tarefas. O YARN supera essas limitações em virtude de sua arquitetura dividida de gerenciador de recursos/mestre de aplicação: ele é projetado para escalar até 10.000 nós e 100.000 tarefas.

Em contraste com o jobtracker, cada instância de uma aplicação - aqui, um trabalho MapReduce - tem um mestre de aplicação dedicado, que é executado durante a duração da aplicação. Este modelo é na verdade mais próximo do artigo original do Google MapReduce, que descreve como um processo mestre é iniciado para coordenar tarefas de mapeamento e redução rodando em um conjunto de workers.

### Disponibilidade

A alta disponibilidade (HA) é geralmente alcançada replicando o estado necessário para que outro daemon assuma o trabalho necessário para fornecer o serviço, no caso de falha do daemon de serviço. No entanto, a grande quantidade de estado complexo e rapidamente mutável na memória do jobtracker (o status de cada tarefa é atualizado a cada poucos segundos, por exemplo) torna muito difícil adaptar o HA ao serviço jobtracker.

Com as responsabilidades do jobtracker divididas entre o gerenciador de recursos e o mestre de aplicação no YARN, tornar o serviço altamente disponível tornou-se um problema de dividir e conquistar: fornecer HA para o gerenciador de recursos, depois para aplicações YARN (em base por aplicação). E de fato, o Hadoop 2 suporta HA tanto para o gerenciador de recursos quanto para o mestre de aplicação para trabalhos MapReduce. A recuperação de falhas no YARN é discutida com mais detalhes em "Falhas" na página 193.

### Utilização

No MapReduce 1, cada tasktracker é configurado com uma alocação estática de "slots" de tamanho fixo, que são divididos em slots de mapeamento e slots de redução no momento da configuração. Um slot de mapeamento só pode ser usado para executar uma tarefa de mapeamento, e um slot de redução só pode ser usado para uma tarefa de redução.

No YARN, um gerenciador de nó gerencia um pool de recursos, em vez de um número fixo de slots designados. O MapReduce rodando no YARN não enfrentará a situação em que uma tarefa de redução precisa esperar porque apenas slots de mapeamento estão disponíveis no cluster, o que pode acontecer no MapReduce 1. Se os recursos para executar a tarefa estiverem disponíveis, então a aplicação será elegível para eles.

Além disso, os recursos no YARN são de granularidade fina, então uma aplicação pode fazer uma solicitação para o que precisa, em vez de para um slot indivisível, que pode ser muito grande (o que é um desperdício de recursos) ou muito pequeno (o que pode causar uma falha) para a tarefa específica.

### Multilocação

De certa forma, o maior benefício do YARN é que ele abre o Hadoop para outros tipos de aplicações distribuídas além do MapReduce. O MapReduce é apenas uma aplicação YARN entre muitas.

É até possível para os usuários executarem diferentes versões do MapReduce no mesmo cluster YARN, o que torna o processo de atualização do MapReduce mais gerenciável. (Observe, no entanto, que algumas partes do MapReduce, como o servidor de histórico de trabalhos e o manipulador de shuffle, bem como o próprio YARN, ainda precisam ser atualizados em todo o cluster.)

Como o Hadoop 2 é amplamente usado e é a versão estável mais recente, no resto deste livro o termo "MapReduce" refere-se ao MapReduce 2, a menos que indicado de outra forma. O Capítulo 7 analisa em detalhes como o MapReduce rodando no YARN funciona.

## Agendamento no YARN

Em um mundo ideal, as solicitações que uma aplicação YARN faz seriam atendidas imediatamente. No mundo real, no entanto, os recursos são limitados e, em um cluster ocupado, uma aplicação frequentemente precisará esperar para ter algumas de suas solicitações atendidas. É trabalho do agendador YARN alocar recursos para aplicações de acordo com alguma política definida. O agendamento em geral é um problema difícil e não há uma política "melhor", por isso o YARN fornece uma escolha de agendadores e políticas configuráveis. Vamos examinar estes a seguir.

### Opções de Agendador

Três agendadores estão disponíveis no YARN: os Agendadores FIFO, de Capacidade e Justo. O Agendador FIFO coloca aplicações em uma fila e as executa na ordem de submissão (primeiro a entrar, primeiro a sair). As solicitações para a primeira aplicação na fila são alocadas primeiro; uma vez que suas solicitações tenham sido satisfeitas, a próxima aplicação na fila é atendida, e assim por diante.

O Agendador FIFO tem o mérito de ser simples de entender e não necessitar de nenhuma configuração, mas não é adequado para clusters compartilhados. Aplicações grandes usarão todos os recursos em um cluster, então cada aplicação tem que esperar sua vez. Em um cluster compartilhado, é melhor usar o Agendador de Capacidade ou o Agendador Justo. Ambos permitem que trabalhos de longa duração sejam concluídos em tempo hábil, enquanto ainda permitem que usuários que estão executando pequenas consultas ad hoc concorrentes obtenham resultados de volta em um tempo razoável.

A diferença entre os agendadores é ilustrada na Figura 4-3, que mostra que sob o Agendador FIFO (i) o pequeno trabalho é bloqueado até que o grande trabalho seja concluído.

Com o Agendador de Capacidade (ii na Figura 4-3), uma fila dedicada separada permite que o pequeno trabalho comece assim que é submetido, embora isso seja ao custo da utilização geral do cluster, uma vez que a capacidade da fila é reservada para trabalhos nessa fila. Isso significa que o grande trabalho termina mais tarde do que quando usa o Agendador FIFO.

Com o Agendador Justo (iii na Figura 4-3), não há necessidade de reservar uma quantidade definida de capacidade, uma vez que ele equilibrará dinamicamente os recursos entre todos os trabalhos em execução. Logo após o primeiro (grande) trabalho começar, ele é o único trabalho em execução, então obtém todos os recursos no cluster. Quando o segundo (pequeno) trabalho começa, ele é alocado com metade dos recursos do cluster, para que cada trabalho esteja usando sua parte justa dos recursos.

Note que há um atraso entre o momento em que o segundo trabalho começa e quando recebe sua parte justa, uma vez que tem que esperar que os recursos se liberem conforme os contêineres usados pelo primeiro trabalho são concluídos. Depois que o pequeno trabalho é concluído e não requer mais recursos, o grande trabalho volta a usar toda a capacidade do cluster novamente. O efeito geral é tanto alta utilização do cluster quanto conclusão oportuna de pequenos trabalhos.

A Figura 4-3 contrasta a operação básica dos três agendadores. Nas próximas duas seções, examinamos algumas das opções de configuração mais avançadas para os Agendadores de Capacidade e Justo.