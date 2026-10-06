# Visão Geral da Arquitetura

O Airflow é uma plataforma que permite construir e executar **workflows** (fluxos de trabalho). Um workflow é representado como um **Dag** (Grafo Acíclico Dirigido - Directed Acyclic Graph) e contém peças individuais de trabalho chamadas **Tasks** (Tarefas), organizadas levando em conta dependências e fluxos de dados.
![[Pasted image 20251222181746.png]]

Um Dag especifica as dependências entre tarefas, o que define a ordem na qual executar as tarefas. As tarefas descrevem o que fazer, seja buscar dados, executar análises, acionar outros sistemas ou mais.

O Airflow em si é agnóstico ao que você está executando - ele irá orquestrar e executar qualquer coisa com prazer, seja com suporte de alto nível de um dos nossos provedores, ou diretamente como um comando usando shell ou Operadores Python.
## Componentes do Airflow

A arquitetura do Airflow consiste em múltiplos componentes. As seções a seguir descrevem a função de cada componente e se eles são necessários para uma instalação mínima do Airflow, ou um componente opcional para alcançar melhor extensibilidade, desempenho e escalabilidade do Airflow.
### Componentes obrigatórios

Uma instalação mínima do Airflow consiste nos seguintes componentes:

- Um **scheduler** (agendador), que lida tanto com o acionamento de workflows agendados quanto com o envio de Tarefas ao executor para execução. O **executor** é uma propriedade de configuração do scheduler, não um componente separado, e é executado dentro do processo do scheduler. Existem vários executores disponíveis prontos para uso, e você também pode escrever o seu próprio.
    
- Um **Dag processor** (processador de Dag), que analisa arquivos Dag e os serializa no banco de dados de metadados. Mais informações sobre o processamento de arquivos Dag podem ser encontradas em Processamento de Arquivos Dag.
    
- Um **webserver** (servidor web), que apresenta uma interface de usuário prática para inspecionar, acionar e depurar o comportamento de Dags e tarefas.
    
- Uma pasta de **arquivos Dag**, que é lida pelo scheduler para descobrir quais tarefas executar e quando executá-las.
    
- Um **banco de dados de metadados**, geralmente PostgreSQL ou MySQL, que armazena o estado de tarefas, Dags e variáveis. A configuração de um banco de dados de metadados é descrita em Configurar um Backend de Banco de Dados e é obrigatória para o Airflow funcionar.
    

### Componentes opcionais

Alguns componentes do Airflow são opcionais e podem habilitar melhor extensibilidade, escalabilidade e desempenho no seu Airflow:

- **Worker** (trabalhador) opcional, que executa as tarefas dadas a ele pelo scheduler. Na instalação básica, o worker pode fazer parte do scheduler e não ser um componente separado. Ele pode ser executado como um processo de longa duração no CeleryExecutor, ou como um POD no KubernetesExecutor.
    
- **Triggerer** (gatilho) opcional, que executa tarefas diferidas em um loop de eventos asyncio. Na instalação básica onde tarefas diferidas não são usadas, um triggerer não é necessário. Mais informações sobre diferimento de tarefas podem ser encontradas em Operadores e Triggers Diferíveis.
    
- Pasta opcional de **plugins**. Plugins são uma forma de estender a funcionalidade do Airflow (similar a pacotes instalados). Os plugins são lidos pelo scheduler, Dag processor, triggerer e webserver. Mais informações sobre plugins podem ser encontradas em Plugins.
    

## Implantando componentes do Airflow

Todos os componentes são aplicações Python que podem ser implantadas usando vários mecanismos de implantação.

Eles podem ter **pacotes instalados** extras em seu ambiente Python. Isso é útil, por exemplo, para instalar operadores ou sensores personalizados ou estender a funcionalidade do Airflow com plugins personalizados.

Embora o Airflow possa ser executado em uma única máquina e com instalação simples onde apenas o scheduler e o webserver são implantados, o Airflow é projetado para ser escalável e seguro, e é capaz de executar em um ambiente distribuído - onde vários componentes podem rodar em diferentes máquinas, com diferentes perímetros de segurança e podem ser escalados executando múltiplas instâncias dos componentes acima.

A separação de componentes também permite maior segurança, isolando os componentes uns dos outros e permitindo a execução de diferentes tarefas. Por exemplo, separar o Dag processor do scheduler garante que o scheduler não tenha acesso aos arquivos Dag e não possa executar código fornecido pelo autor do Dag.

Além disso, embora uma única pessoa possa executar e gerenciar a instalação do Airflow, a implantação do Airflow em configurações mais complexas pode envolver vários papéis de usuários que podem interagir com diferentes partes do sistema, o que é um aspecto importante da implantação segura do Airflow. Os papéis são descritos em detalhes no Modelo de Segurança do Airflow e geralmente incluem:

- **Deployment Manager** (Gerente de Implantação) - uma pessoa que instala e configura o Airflow e gerencia a implantação
- **Dag author** (Autor de Dag) - uma pessoa que escreve Dags e os submete ao Airflow
- **Operations User** (Usuário de Operações) - uma pessoa que aciona Dags e tarefas e monitora sua execução

## Diagramas de Arquitetura

Os diagramas abaixo mostram diferentes formas de implantar o Airflow - gradualmente da implantação simples "uma máquina" e uma única pessoa, para uma implantação mais complexa com componentes separados, papéis de usuário separados e, finalmente, com perímetros de segurança mais isolados.

O significado dos diferentes tipos de conexão nos diagramas abaixo é o seguinte:

- **linhas sólidas marrons** representam submissão e sincronização de arquivos Dag
- **linhas sólidas azuis** representam implantação e acesso a pacotes instalados e plugins
- **linhas tracejadas pretas** representam fluxo de controle de workers pelo scheduler (via executor)
- **linhas sólidas pretas** representam acesso à UI para gerenciar a execução dos workflows
- **linhas tracejadas vermelhas** representam acesso ao banco de dados de metadados por todos os componentes

### Implantação básica do Airflow

Esta é a implantação mais simples do Airflow, geralmente operada e gerenciada em uma única máquina. Tal implantação geralmente usa o LocalExecutor, onde o scheduler e os workers estão no mesmo processo Python e os arquivos Dag são lidos diretamente do sistema de arquivos local pelo scheduler. O webserver roda na mesma máquina que o scheduler. Não há componente triggerer, o que significa que o diferimento de tarefas não é possível.

Tal instalação normalmente não separa papéis de usuário - implantação, configuração, operação, autoria e manutenção são todos feitos pela mesma pessoa e não há perímetros de segurança entre os componentes.
![[Pasted image 20251222190015.png]]

Se você deseja executar o Airflow em uma única máquina em uma configuração simples de máquina única, você pode pular os diagramas mais complexos abaixo e ir direto para a seção Cargas de trabalho.

### Arquitetura distribuída do Airflow

Esta é a arquitetura do Airflow onde os componentes do Airflow são distribuídos entre múltiplas máquinas e onde vários papéis de usuários são introduzidos - Gerente de Implantação, Autor de Dag, Usuário de Operações. Você pode ler mais sobre esses vários papéis no Modelo de Segurança do Airflow.

No caso de uma implantação distribuída, é importante considerar os aspectos de segurança dos componentes. O webserver não tem acesso aos arquivos Dag diretamente. O código na aba "Code" da UI é lido do banco de dados de metadados. O webserver não pode executar nenhum código enviado pelo Autor de Dag. Ele só pode executar código que está instalado como um pacote instalado ou plugin pelo Gerente de Implantação. O Usuário de Operações só tem acesso à UI e só pode acionar Dags e tarefas, mas não pode criar Dags.

Os arquivos Dag precisam ser sincronizados entre todos os componentes que os utilizam - scheduler, triggerer e workers. Os arquivos Dag podem ser sincronizados por vários mecanismos - as formas típicas de como os Dags podem ser sincronizados são descritas em Gerenciar arquivos Dag da nossa documentação do Helm Chart. O Helm chart é uma das formas de implantar o Airflow em cluster K8S.

![[Pasted image 20251222190204.png]]

### Arquitetura de processamento separado de Dag

Em uma instalação mais complexa onde segurança e isolamento são importantes, você também verá o componente Dag processor autônomo que permite separar o scheduler do acesso aos arquivos Dag. Isso é adequado se o foco da implantação for o isolamento entre tarefas analisadas. Embora o Airflow ainda não suporte recursos completos de multi-tenancy, ele pode ser usado para garantir que o código fornecido pelo Autor de Dag nunca seja executado no contexto do scheduler.
![[Pasted image 20251222190236.png]]
**Nota:** Quando um arquivo Dag é alterado, pode haver casos em que o scheduler e o worker verão versões diferentes do Dag até que ambos os componentes se atualizem. Você pode evitar o problema certificando-se de que o Dag esteja desativado durante a implantação e reativado quando finalizado. Se necessário, a cadência de sincronização e varredura da pasta de Dag pode ser configurada. Por favor, certifique-se de que você realmente sabe o que está fazendo se alterar as configurações.

## Cargas de trabalho

Um Dag passa por uma série de Tarefas, e há três tipos comuns de tarefa que você verá:

- **Operators** (Operadores): tarefas predefinidas que você pode encadear rapidamente para construir a maior parte dos seus Dags.
- **Sensors** (Sensores): uma subclasse especial de Operadores que são inteiramente sobre esperar que um evento externo aconteça.
- Um **@task** decorado com TaskFlow: uma função Python personalizada empacotada como uma Tarefa.

Internamente, todos esses são na verdade subclasses do `BaseOperator` do Airflow, e os conceitos de Tarefa e Operador são de certa forma intercambiáveis, mas é útil pensar neles como conceitos separados - essencialmente, Operadores e Sensores são modelos (templates), e quando você chama um em um arquivo Dag, você está criando uma Tarefa.

## Fluxo de Controle

Os Dags são projetados para serem executados várias vezes, e múltiplas execuções deles podem acontecer em paralelo. Os Dags são parametrizados, sempre incluindo um intervalo para o qual estão "executando" (o intervalo de dados), mas com outros parâmetros opcionais também.

As Tarefas têm dependências declaradas entre si. Você verá isso em um Dag usando os operadores `>>` e `<<`:

```python
first_task >> [second_task, third_task]
fourth_task << third_task
```

Ou, com os métodos `set_upstream` e `set_downstream`:

```python
first_task.set_downstream([second_task, third_task])
fourth_task.set_upstream(third_task)
```

Essas dependências são o que compõem as "arestas" do grafo, e como o Airflow descobre em qual ordem executar suas tarefas. Por padrão, uma tarefa aguardará todas as suas tarefas upstream terem sucesso antes de ser executada, mas isso pode ser customizado usando recursos como Branching (Ramificação), LatestOnly (Apenas Mais Recente) e Trigger Rules (Regras de Gatilho).

Para passar dados entre tarefas você tem três opções:

- **XComs** (Comunicações Cruzadas - "Cross-communications"): um sistema onde você pode fazer tarefas enviarem e receberem pequenos pedaços de metadados.
- Fazer upload e download de arquivos grandes de um serviço de armazenamento (seja um que você execute, ou parte de uma nuvem pública)
- A **API TaskFlow** passa dados automaticamente entre tarefas via XComs implícitos

O Airflow envia Tarefas para executar em Workers conforme o espaço fica disponível, então não há garantia de que todas as tarefas no seu Dag serão executadas no mesmo worker ou na mesma máquina.

Conforme você constrói seus Dags, eles provavelmente ficarão muito complexos, então o Airflow fornece vários mecanismos para tornar isso mais sustentável, por exemplo, TaskGroups permitem que você agrupe visualmente tarefas na UI.

Há também recursos para permitir que você pré-configure facilmente o acesso a um recurso central, como um datastore, na forma de Connections & Hooks (Conexões e Ganchos), e para limitar concorrência, via Pools (Pools).

## Interface do usuário

O Airflow vem com uma interface de usuário que permite ver o que os Dags e suas tarefas estão fazendo, acionar execuções de Dags, visualizar logs e fazer alguma depuração limitada e resolução de problemas com seus Dags.
![[Pasted image 20251222190322.png]]
É geralmente a melhor maneira de ver o status da sua instalação do Airflow como um todo, bem como mergulhar em Dags individuais para ver seu layout, o status de cada tarefa e os logs de cada tarefa.