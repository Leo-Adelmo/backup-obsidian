Fonte: https://airflow.apache.org/docs/apache-airflow/stable/core-concepts/executor/index.html

---
# Executor

Executors são o mecanismo pelo qual as instâncias de tarefas são executadas. Eles possuem uma API comum e são "plugáveis", o que significa que você pode trocar executores com base nas necessidades da sua instalação.

Os executors são definidos pela opção `executor` na seção `[core]` do arquivo de configuração.

Executors integrados são referenciados pelo nome, por exemplo:

```
[core]
executor = KubernetesExecutor
```

Executors personalizados ou de terceiros podem ser configurados fornecendo o caminho do módulo da classe python do executor, por exemplo:

```
[core]
executor = my.custom.executor.module.ExecutorClass
```

Se você quiser verificar qual executor está configurado atualmente, pode usar o comando `airflow config get-value core executor`:

```
$ airflow config get-value core executor
LocalExecutor
```

## Tipos de Executors

Existe apenas um tipo de executor que executa tarefas localmente (dentro do processo `scheduler`) na árvore do repositório, mas outros personalizados podem ser escritos para alcançar resultados similares, e há aqueles que executam suas tarefas remotamente (geralmente através de um pool de workers). O Airflow vem configurado com o `LocalExecutor` por padrão, que é um executor local e a opção mais simples para execução. No entanto, como o `LocalExecutor` executa processos no processo do scheduler, isso pode ter um impacto no desempenho do scheduler. Você pode usar o `LocalExecutor` para instalações de produção pequenas e de máquina única, ou um dos executores remotos para instalações multi-máquinas/nuvem.

### Executors Locais

As tarefas do Airflow são executadas localmente dentro do processo do scheduler.

**Prós**: Muito fácil de usar, rápido, latência muito baixa e poucos requisitos para configuração.

**Contras**: Capacidades limitadas e compartilha recursos com o scheduler do Airflow.

**Exemplos**:

- Local Executor

### Executors Remotos

Executores remotos podem ser divididos em duas categorias:

**Executors em Fila/Batch**

As tarefas do Airflow são enviadas para uma fila central onde workers remotos puxam tarefas para executar. Geralmente, os workers são persistentes e executam múltiplas tarefas de uma vez.

**Prós**: Mais robustos pois você está desacoplando os workers do processo do scheduler. Os workers podem ser hosts grandes que processam muitas tarefas (geralmente em paralelo), o que é econômico. A latência pode ser relativamente baixa, pois os workers podem ser provisionados para estarem em execução o tempo todo para pegar tarefas imediatamente da fila.

**Contras**: Workers compartilhados têm o problema do "vizinho barulhento" com tarefas competindo por recursos nos hosts compartilhados ou competindo pela forma como o ambiente/sistema está configurado. Eles também podem ser caros se sua carga de trabalho não for constante - você pode ter workers ociosos, recursos excessivamente escalados, ou precisa gerenciar o escalonamento para cima e para baixo.

**Exemplos**:

- CeleryExecutor
- BatchExecutor
- EdgeExecutor (Experimental Pré-Lançamento)

**Executors Containerizados**

As tarefas do Airflow são executadas ad hoc dentro de containers/pods. Cada tarefa é isolada em seu próprio ambiente containerizado que é implantado quando a tarefa do Airflow é enfileirada.

**Prós**: Cada tarefa do Airflow é isolada em um container, então não há problema de vizinho barulhento. O ambiente de execução pode ser customizado para tarefas específicas (bibliotecas do sistema, binários, dependências, quantidade de recursos, etc). Econômico, pois os workers estão vivos apenas pela duração da tarefa.

**Contras**: Há latência na inicialização, pois o container ou pod precisa ser implantado antes que a tarefa possa começar. Pode ser caro se você estiver executando muitas tarefas curtas/pequenas. Não há workers para gerenciar, mas você deve gerenciar algo como um cluster Kubernetes.

**Exemplos**:

- KubernetesExecutor
- EcsExecutor

**Nota**: Novos usuários do Airflow podem presumir que precisam executar um processo executor separado usando um dos Executors Locais ou Remotos. Isso não está correto. A lógica do executor é executada dentro do processo do scheduler e executará as tarefas localmente ou não dependendo do executor selecionado.

## Usando Múltiplos Executors Simultaneamente

A partir da versão 2.10.0, o Airflow pode operar com uma configuração multi-executor. Cada executor tem seu próprio conjunto de prós e contras, frequentemente sendo trade-offs entre latência, isolamento e eficiência computacional, entre outras propriedades. Executar múltiplos executores permite que você faça melhor uso dos pontos fortes de todos os executores disponíveis e evite suas fraquezas. Em outras palavras, você pode usar um executor específico para um conjunto específico de tarefas onde seus méritos e benefícios particulares fazem mais sentido para esse caso de uso.

### Configuração

Configurar múltiplos executores usa a mesma opção de configuração, aproveitando uma notação de lista separada por vírgulas para especificar múltiplos executores.

**Nota**: O primeiro executor na lista será o executor padrão para o ambiente. Qualquer Tarefa ou Dag do Airflow que não especificar um executor específico usará este executor no nível do ambiente. Todos os outros executores na lista serão inicializados e prontos para executar tarefas se especificados em uma Tarefa ou Dag do Airflow.

Alguns exemplos de configuração de múltiplos executores válida:

```
[core]
executor = LocalExecutor
```

```
[core]
executor = LocalExecutor,CeleryExecutor
```

```
[core]
executor = KubernetesExecutor,my.custom.module.ExecutorClass
```

Para facilitar a especificação de executores em tarefas e Dags, a configuração de executor agora suporta aliases. Você pode então usar este alias para se referir ao executor em seus Dags:

```
[core]
executor = LocalExecutor,ShortName:my.custom.module.ExecutorClass
```

### Escrevendo Dags e Tarefas

Para especificar um executor para uma tarefa, use o parâmetro executor nos Operadores do Airflow:

```python
BashOperator(
    task_id="hello_world",
    executor="LocalExecutor",
    bash_command="echo 'hello world!'",
)
```

```python
@task(executor="LocalExecutor")
def hello_world():
    print("hello world!")
```

Para especificar um executor para um Dag inteiro, use o mecanismo existente do Airflow de argumentos padrão:

```python
with DAG(
    dag_id="hello_worlds",
    default_args={"executor": "LocalExecutor"},  # Aplica a todas as tarefas no Dag
) as dag:
    hw = hello_world()
    hw_again = hello_world_again()
```

### Monitoramento

Quando usar um único executor, as métricas do Airflow se comportarão como eram antes da versão 2.9. Mas se múltiplos executores estiverem configurados, as métricas do executor (`executor.open_slots`, `executor.queued_slots`, e `executor.running_tasks`) serão publicadas para cada executor configurado, com o nome do executor anexado ao nome da métrica (ex: `executor.open_slots.<nome da classe do executor>`).

## Escrevendo Seu Próprio Executor

Todos os executores do Airflow implementam uma interface comum para que sejam plugáveis. Principalmente, o scheduler do Airflow usa essa interface para interagir com o executor. A interface pública é o `BaseExecutor`.

Algumas razões pelas quais você pode querer escrever um executor personalizado incluem:

- Não existe um executor que se ajuste ao seu caso de uso específico
- Você gostaria de usar um executor que aproveite um serviço de computação do seu provedor de nuvem preferido
- Você tem uma ferramenta/serviço privado para execução de tarefas disponível apenas para você ou sua organização

### Métodos Obrigatórios para Implementar

Os seguintes métodos devem ser sobrescritos no mínimo para ter seu executor suportado pelo Airflow:

- **sync**: Será chamado periodicamente durante os heartbeats do executor. Implemente este método para atualizar o estado das tarefas que o executor conhece.
- **execute_async**: Executa uma carga de trabalho de forma assíncrona. Este método é chamado durante o heartbeat do executor que é executado periodicamente pelo scheduler.

### Métodos Opcionais da Interface para Implementar

- **start**: O job scheduler do Airflow chamará este método após inicializar o objeto executor.
- **end**: O job scheduler do Airflow chamará este método ao encerrar.
- **terminate**: Parar o executor de forma mais forçada, até mesmo matando/parando tarefas em execução.
- **try_adopt_task_instances**: Tarefas que foram abandonadas são fornecidas ao executor para adotar ou tratá-las através deste método.
- **get_cli_commands**: Executores podem fornecer comandos CLI aos usuários implementando este método.
- **get_task_log**: Executores podem fornecer mensagens de log aos logs de tarefas do Airflow implementando este método.

---

Esta é uma tradução completa da página sobre Executors do Apache Airflow, mantendo a estrutura e o conteúdo técnico original.