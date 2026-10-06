# Documentação: Apache Hadoop YARN e HDFS

## Sumário

1. [Introdução ao Apache Hadoop](https://claude.ai/chat/4685b0e2-295b-465f-aaf9-5ceb1b9f2310#introdu%C3%A7%C3%A3o-ao-apache-hadoop)
2. [HDFS (Hadoop Distributed File System)](https://claude.ai/chat/4685b0e2-295b-465f-aaf9-5ceb1b9f2310#hdfs-hadoop-distributed-file-system)
    - [Arquitetura do HDFS](https://claude.ai/chat/4685b0e2-295b-465f-aaf9-5ceb1b9f2310#arquitetura-do-hdfs)
    - [Componentes Principais](https://claude.ai/chat/4685b0e2-295b-465f-aaf9-5ceb1b9f2310#componentes-principais-do-hdfs)
    - [Funcionamento e Replicação de Dados](https://claude.ai/chat/4685b0e2-295b-465f-aaf9-5ceb1b9f2310#funcionamento-e-replica%C3%A7%C3%A3o-de-dados)
    - [Vantagens e Limitações](https://claude.ai/chat/4685b0e2-295b-465f-aaf9-5ceb1b9f2310#vantagens-e-limita%C3%A7%C3%B5es-do-hdfs)
3. [YARN (Yet Another Resource Negotiator)](https://claude.ai/chat/4685b0e2-295b-465f-aaf9-5ceb1b9f2310#yarn-yet-another-resource-negotiator)
    - [Arquitetura do YARN](https://claude.ai/chat/4685b0e2-295b-465f-aaf9-5ceb1b9f2310#arquitetura-do-yarn)
    - [Componentes Principais](https://claude.ai/chat/4685b0e2-295b-465f-aaf9-5ceb1b9f2310#componentes-principais-do-yarn)
    - [Gerenciamento de Recursos](https://claude.ai/chat/4685b0e2-295b-465f-aaf9-5ceb1b9f2310#gerenciamento-de-recursos)
    - [Escalonamento de Aplicações](https://claude.ai/chat/4685b0e2-295b-465f-aaf9-5ceb1b9f2310#escalonamento-de-aplica%C3%A7%C3%B5es)
4. [Integração entre HDFS e YARN](https://claude.ai/chat/4685b0e2-295b-465f-aaf9-5ceb1b9f2310#integra%C3%A7%C3%A3o-entre-hdfs-e-yarn)
5. [Casos de Uso](https://claude.ai/chat/4685b0e2-295b-465f-aaf9-5ceb1b9f2310#casos-de-uso)
6. [Referências Bibliográficas](https://claude.ai/chat/4685b0e2-295b-465f-aaf9-5ceb1b9f2310#refer%C3%AAncias-bibliogr%C3%A1ficas)

## Introdução ao Apache Hadoop

O Apache Hadoop é um framework de software de código aberto projetado para armazenamento e processamento distribuído de grandes conjuntos de dados usando modelos de programação simples. O Hadoop foi desenvolvido para escalar de servidores individuais para milhares de máquinas, cada uma oferecendo computação e armazenamento local (White, 2015).

De acordo com Shvachko et al. (2010), o Hadoop surgiu como uma resposta às limitações dos bancos de dados relacionais tradicionais quando se trata de processar conjuntos de dados massivos e não estruturados. O ecossistema Hadoop evoluiu significativamente desde sua criação, tornando-se uma solução fundamental para implementações de Big Data.

O Apache Hadoop é composto por vários módulos complementares, sendo os dois principais o HDFS (Hadoop Distributed File System) para armazenamento e o YARN (Yet Another Resource Negotiator) para gerenciamento de recursos e escalonamento de tarefas (Vavilapalli et al., 2013).

## HDFS (Hadoop Distributed File System)

### Arquitetura do HDFS

O HDFS é um sistema de arquivos distribuído projetado para ser executado em hardware commodity, proporcionando alta tolerância a falhas e alta taxa de transferência para aplicações com grandes conjuntos de dados. Segundo Shvachko et al. (2010), a arquitetura do HDFS segue o modelo mestre-escravo, consistindo em um único NameNode (mestre) e múltiplos DataNodes (escravos).

Conforme explicado por White (2015), o HDFS foi projetado especificamente para:

- Armazenar arquivos muito grandes (gigabytes a terabytes)
- Seguir o padrão "write-once, read-many-times" (escrever uma vez, ler muitas vezes)
- Executar em clusters de hardware commodity propensos a falhas
- Priorizar alto throughput em vez de baixa latência
- Mover a computação para os dados em vez do contrário

### Componentes Principais do HDFS

De acordo com Shvachko et al. (2010), os componentes fundamentais do HDFS são:

1. **NameNode**: Atua como o mestre no modelo mestre-escravo e gerencia o namespace do sistema de arquivos. Mantém a metainformação de todo o sistema de arquivos, incluindo a árvore de diretórios e metadados para todos os arquivos. O NameNode também determina o mapeamento de blocos para DataNodes.
    
2. **DataNodes**: São os nós escravos responsáveis pelo armazenamento e recuperação dos blocos de dados. Os DataNodes relatam periodicamente ao NameNode a lista de blocos que estão armazenando.
    
3. **Secondary NameNode**: Não é um backup do NameNode, mas sim um nó auxiliar que executa tarefas periódicas para manter o NameNode mais eficiente, principalmente combinando o arquivo de imagem com o arquivo de log de edições.
    

Como observado por Hadoop et al. (2014), o HDFS divide os arquivos em blocos grandes (tipicamente 128MB ou 256MB) e armazena cada bloco de forma redundante em vários DataNodes para garantir confiabilidade e disponibilidade.

### Funcionamento e Replicação de Dados

Segundo Shafer et al. (2010), o HDFS implementa um mecanismo de replicação de dados para garantir a confiabilidade e disponibilidade dos dados. Tipicamente, cada bloco de dados é replicado em múltiplos DataNodes (o fator de replicação padrão é 3) para proteger contra falhas de hardware e perda de dados.

A replicação segue uma política específica, como descrita por White (2015):

- A primeira réplica é colocada no mesmo nó que o cliente (se o cliente estiver em um DataNode no cluster)
- A segunda réplica é posicionada em um rack diferente do primeiro
- A terceira réplica é colocada no mesmo rack do segundo, mas em um nó diferente

Este modelo de replicação "rack-aware" equilibra confiabilidade, utilização da largura de banda da rede e distribuição de leituras entre os racks.

### Vantagens e Limitações do HDFS

**Vantagens** (conforme Shvachko et al., 2010):

- Alta tolerância a falhas através da replicação de dados
- Adequado para processamento em lote de grandes conjuntos de dados
- Escalonabilidade linear para armazenamento e computação
- Economia por utilizar hardware commodity
- Alta taxa de transferência de dados

**Limitações** (de acordo com Vavilapalli et al., 2013):

- Não adequado para acesso de baixa latência aos dados
- Ineficiente para muitos arquivos pequenos devido ao overhead de metadados
- Suporte limitado para modificações concorrentes (não é um sistema de arquivos POSIX completo)
- Ponto único de falha no NameNode (nas versões anteriores ao Hadoop 2.x)

## YARN (Yet Another Resource Negotiator)

### Arquitetura do YARN

YARN foi introduzido no Apache Hadoop 2.0 como um sistema de gerenciamento de recursos e escalonamento que separa as funcionalidades de gerenciamento de recursos e escalonamento/monitoramento de tarefas em daemons separados. Como destacado por Vavilapalli et al. (2013), essa separação permitiu que o Hadoop evoluísse de uma plataforma dedicada ao MapReduce para uma plataforma mais genérica de processamento distribuído.

Murthy et al. (2014) explicam que o YARN foi projetado para atender cinco requisitos principais:

1. Escalabilidade: suportar clusters com milhares de nós
2. Multi-tenancy: permitir múltiplos frameworks de processamento
3. Utilização do cluster: gerenciar recursos de forma eficiente
4. Segurança e controle de acesso: proteger dados e recursos
5. Compatibilidade: suportar aplicações MapReduce existentes

### Componentes Principais do YARN

De acordo com Vavilapalli et al. (2013), a arquitetura do YARN é composta pelos seguintes componentes:

1. **ResourceManager (RM)**: É o componente mestre que gerencia todos os recursos do cluster. O RM aloca recursos para aplicações de acordo com restrições como capacidade da fila, controle de acesso de usuários e políticas de escalonamento.
    
2. **NodeManager (NM)**: É o agente em cada nó responsável por monitorar o uso de recursos (CPU, memória, disco, rede) e reportá-los ao ResourceManager. Também gerencia os containers de aplicação no nó.
    
3. **ApplicationMaster (AM)**: É específico para cada aplicação e responsável por negociar recursos com o ResourceManager e trabalhar com os NodeManagers para executar e monitorar as tarefas. Cada aplicação tem seu próprio ApplicationMaster.
    
4. **Container**: Representa uma coleção de recursos físicos (CPU, memória, etc.) em um único nó. Todos os recursos no YARN são alocados como containers.
    

Como observado por Murthy et al. (2014), essa arquitetura permite que o YARN suporte diversos frameworks de processamento (como MapReduce, Spark, Tez, etc.) simultaneamente no mesmo cluster.

### Gerenciamento de Recursos

De acordo com Vavilapalli et al. (2013), o YARN implementa um modelo de gerenciamento de recursos de duas camadas:

1. **Gerenciamento Global**: Realizado pelo ResourceManager, que mantém uma visão global dos recursos disponíveis no cluster e decide como alocá-los entre as aplicações.
    
2. **Gerenciamento por Aplicação**: Conduzido pelos ApplicationMasters, que são responsáveis por solicitar recursos específicos para suas aplicações e monitorar a execução das tarefas.
    

Sharma et al. (2016) explicam que o YARN define recursos usando um modelo abstrato que permite que administradores configurem diferentes tipos de recursos a serem considerados no escalonamento, incluindo:

- Memória (RAM)
- vCores (unidades de CPU virtual)
- Recursos personalizados (como GPUs, em versões mais recentes)

### Escalonamento de Aplicações

O YARN oferece vários escalonadores para atender diferentes necessidades de ambiente, conforme documentado por Murthy et al. (2014):

1. **Capacity Scheduler**: Desenvolvido para permitir que múltiplas organizações compartilhem um grande cluster Hadoop. Fornece um mínimo de capacidade para cada organização e permite utilização eficiente dos recursos não utilizados.
    
2. **Fair Scheduler**: Aloca recursos de forma que todas as aplicações obtenham, em média, uma parcela igual dos recursos ao longo do tempo. Também suporta agendamento hierárquico através de filas aninhadas.
    
3. **FIFO Scheduler**: Um escalonador simples que coloca as aplicações em uma fila e executa-as na ordem recebida.
    

Vavilapalli et al. (2013) observam que o escalonador padrão (Capacity Scheduler) suporta recursos como preempção, reservas, priorização, e limites de capacidade, permitindo um controle refinado sobre a utilização dos recursos do cluster.

## Integração entre HDFS e YARN

Segundo White (2015), o HDFS e o YARN são componentes complementares no ecossistema Hadoop, com o HDFS fornecendo armazenamento distribuído e o YARN gerenciando recursos e escalonando tarefas.

Murthy et al. (2014) destacam várias formas como o HDFS e o YARN trabalham juntos:

1. **Locality Awareness**: O YARN tenta escalonar tarefas em nós onde os dados estão armazenados no HDFS para minimizar o tráfego de rede.
    
2. **Resource Allocation**: O YARN aloca recursos considerando tanto os requisitos das aplicações quanto a disponibilidade do HDFS.
    
3. **Fault Tolerance**: Ambos os sistemas são projetados para detectar e se recuperar de falhas, garantindo a robustez do cluster.
    
4. **Security**: Compartilham um modelo de segurança comum, incluindo autenticação Kerberos e controle de acesso.
    

De acordo com Sharma et al. (2016), essa integração permite que o Hadoop ofereça uma plataforma completa para processamento distribuído, combinando armazenamento escalável e computação flexível.

## Casos de Uso

Conforme documentado por White (2015) e Vavilapalli et al. (2013), o Hadoop YARN e HDFS são amplamente utilizados em diversos cenários:

1. **Análise de Big Data**: Para processamento de grandes volumes de dados estruturados e não estruturados.
    
2. **Data Warehousing**: Como base para data lakes e armazenamento de longo prazo.
    
3. **Processamento de Logs**: Para armazenar e analisar logs de servidores, aplicações e dispositivos.
    
4. **Machine Learning**: Como infraestrutura para treinar e avaliar modelos em grandes conjuntos de dados.
    
5. **ETL (Extract, Transform, Load)**: Para processar dados brutos e prepará-los para análise.
    

Sharma et al. (2016) observam que muitas empresas Fortune 500 implementaram clusters Hadoop para casos de uso críticos, incluindo análise de comportamento do usuário, detecção de fraudes, recomendações personalizadas e processamento de dados de sensores.

## Referências Bibliográficas

Hadoop, A., White, T., Cutting, D., Baldeschwieler, E., & others. (2014). Apache Hadoop HDFS Architecture Guide. _Apache Software Foundation_. https://hadoop.apache.org/docs/r2.7.1/hadoop-project-dist/hadoop-hdfs/HdfsDesign.html

Murthy, A. C., Vavilapalli, V. K., Eadline, D., Niemiec, J., & Markham, J. (2014). _Apache Hadoop YARN: Moving beyond MapReduce and Batch Processing with Apache Hadoop 2_. Addison-Wesley Professional.

Shafer, J., Rixner, S., & Cox, A. L. (2010). The Hadoop distributed filesystem: Balancing portability and performance. _IEEE International Symposium on Performance Analysis of Systems & Software_, 122-133.

Sharma, M., Hasteer, N., Tuli, A., & Bansal, A. (2016). Investigating the inclinations of research and practices in Hadoop: A systematic review. _Journal of Big Data_, 3(1), 1-33.

Shvachko, K., Kuang, H., Radia, S., & Chansler, R. (2010). The Hadoop Distributed File System. _IEEE 26th Symposium on Mass Storage Systems and Technologies (MSST)_, 1-10. https://doi.org/10.1109/MSST.2010.5496972

Vavilapalli, V. K., Murthy, A. C., Douglas, C., Agarwal, S., Konar, M., Evans, R., Graves, T., Lowe, J., Shah, H., Seth, S., Saha, B., Curino, C., O'Malley, O., Radia, S., Reed, B., & Baldeschwieler, E. (2013). Apache Hadoop YARN: Yet Another Resource Negotiator. _Proceedings of the 4th Annual Symposium on Cloud Computing (SOCC)_, 5:1-5:16. https://doi.org/10.1145/2523616.2523633

White, T. (2015). _Hadoop: The Definitive Guide (4th ed.)_. O'Reilly Media.