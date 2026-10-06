

## SOLID

SOLID é um acrônimo que representa cinco princípios de design orientado a objetos que ajudam a tornar o software mais compreensível, flexível e manutenível.

Segundo a Microsoft: "Os princípios SOLID são diretrizes que podem ajudá-lo a criar software que é fácil de manter e estender ao longo do tempo." ([Microsoft Learn](https://learn.microsoft.com/pt-br/dotnet/architecture/modern-web-apps-azure/architectural-principles#solid))

Os cinco princípios são:

1. **S - Princípio da Responsabilidade Única (Single Responsibility Principle)**: Uma classe deve ter apenas uma razão para mudar.
    
2. **O - Princípio do Aberto/Fechado (Open/Closed Principle)**: Entidades de software devem estar abertas para extensão, mas fechadas para modificação.
    
3. **L - Princípio da Substituição de Liskov (Liskov Substitution Principle)**: Objetos em um programa devem ser substituíveis por instâncias de seus subtipos sem alterar a correção do programa.
    
4. **I - Princípio da Segregação de Interface (Interface Segregation Principle)**: Muitas interfaces específicas são melhores do que uma interface única geral.
    
5. **D - Princípio da Inversão de Dependência (Dependency Inversion Principle)**: Dependa de abstrações, não de implementações concretas.
    

Como ressalta a Oracle: "O uso dos princípios SOLID ajuda a criar sistemas mais robustos, escaláveis e testáveis." ([Oracle Developer Blog](https://blogs.oracle.com/developers/post/solid-design-principles-in-java))

## Object Calisthenics

Object Calisthenics é um conjunto de regras ou exercícios de programação criados para melhorar a qualidade do código orientado a objetos. O termo foi introduzido por Jeff Bay em "The ThoughtWorks Anthology" e funciona como uma espécie de "calistenia" para programadores

## As 9 Regras do Object Calisthenics

### 1. Um nível de indentação por método

Cada método deve ter apenas um nível de indentação. Se você tem vários níveis, isso indica que o método está fazendo muitas coisas e deve ser dividido em métodos menores.

### 2. Não use a palavra-chave "else"

Condicionais podem ser expressas de forma mais elegante usando fail fast ou early return (Descritos abaixo).

### 3. Encapsule todos os primitivos e strings

Variáveis como strings, números e booleanos devem estar encapsuladas em objetos para adicionar comportamento e validação.

### 4. Coleções de primeira classe

Se você tem uma coleção (lista, array), encapsule-a em uma classe que forneça métodos para adicionar, remover e iterar sobre os elementos.

### 5. Um ponto por linha

Evite encadeamento de métodos como `objeto.getAlgo().getFazAlgo().metodo()`. Esta regra ajuda a evitar a Lei de Demeter (princípio de menor conhecimento).

### 6. Não abrevie nomes

Use nomes descritivos e completos para classes, métodos e variáveis. Isso facilita a leitura e entendimento do código.

### 7. Mantenha todas as entidades pequenas

Classes não devem ter mais que 50 linhas e pacotes não mais que 10 classes. Métodos devem ter no máximo 5 linhas de código.

### 8. No máximo 2 atributos de instância por classe

Limitar o número de atributos força você a criar objetos mais coesos e com responsabilidade única.

### 9. Sem getters e setters

Métodos devem representar comportamentos, não apenas obter ou definir estados. Em vez de perguntar por dados, peça ao objeto para fazer algo.

## Fail Fast

O princípio "Fail Fast" enfatiza a detecção e resposta rápida a falhas no início do ciclo de desenvolvimento.

A Google Cloud afirma: "O princípio Fail Fast incentiva a detecção precoce de problemas para que possam ser resolvidos rapidamente, economizando tempo e recursos." ([Google Cloud Architecture Center](https://cloud.google.com/architecture/framework/reliability/design-patterns))

Os principais benefícios incluem:

- Detecção precoce de problemas
- Redução do tempo de depuração
- Melhoria da qualidade do código
- Maior confiabilidade do sistema

## Early Return

Early Return é uma técnica de programação que visa simplificar o fluxo de código retornando resultados assim que possível.

Segundo a Microsoft:

"O padrão Early Return ajuda a reduzir a complexidade do código eliminando aninhamentos desnecessários e tornando o fluxo de execução mais linear." ([Microsoft Learn](https://learn.microsoft.com/en-us/windows/apps/design/controls/avoid-excessive-nesting))

Benefícios do Early Return:

- Código mais legível e manutenível
- Redução de aninhamentos e complexidade
- Melhoria do desempenho em alguns casos
- Clareza de intenção no código

## Domain Driven Design (DDD)

O Domain Driven Design é uma abordagem de desenvolvimento de software que coloca o domínio do problema no centro do processo de design. Criado por Eric Evans em seu livro "Domain-Driven Design: Tackling Complexity in the Heart of Software" (2003), o DDD fornece um conjunto de conceitos, princípios e padrões para lidar com sistemas complexos.

Elementos principais do DDD:


- **Domínio**: O domínio representa a área de conhecimento ou atividade à qual o software se aplica. É o "problema de negócio" que o software está tentando resolver. Compreender profundamente o domínio é essencial no DDD.
  
- **Linguagem Ubíqua**: É uma linguagem comum compartilhada entre desenvolvedores e especialistas do domínio. Todos os termos e conceitos do modelo de domínio devem refletir esta linguagem, criando um vocabulário consistente que é usado em discussões, código, documentação e todos os artefatos do projeto.
  
- **Contextos Limitados (Bounded Contexts)**: Um sistema complexo geralmente contém múltiplos modelos. Os contextos delimitados definem os limites dentro dos quais um modelo específico é aplicado. Cada contexto tem sua própria linguagem ubíqua e representa uma parte específica do domínio.
  
- **Mapa de Contexto (Context Map)**: Documenta como diferentes contextos delimitados se relacionam entre si, definindo as integrações e traduções necessárias entre eles.
  
- **Entidades: Objetos que têm uma identidade única que persiste ao longo do tempo, independentemente de suas atribuições. Por exemplo, um "Cliente" com um ID único é uma entidade.
  
- **Objetos de Valor**: Objetos sem identidade própria, definidos apenas por seus atributos. São imutáveis e usados para descrever aspectos do domínio. Exemplos: Endereço, Nome, Dinheiro.
  
- **Agregados**: Conjunto de entidades e objetos de valor tratados como uma unidade para alterações de dados. Cada agregado tem uma raiz (Aggregate Root) que é a única entidade acessível de fora do agregado.
  
- **Serviços de Domínio**: Quando uma operação não pertence naturalmente a uma entidade ou objeto de valor, ela é implementada como um serviço no domínio.
  
- **Repositórios**: Abstraem a infraestrutura necessária para persistência de dados, proporcionando uma ilusão de coleção em memória de todos os objetos de um determinado tipo.

#### Padrões Estratégicos do DDD

##### Núcleo do Domínio (Core Domain)

É a parte mais valiosa e diferenciada do seu software. É onde você deve investir mais esforços de modelagem e design.

##### Subdomínios

O domínio completo pode ser dividido em subdomínios menores e mais gerenciáveis:

- **Subdomínios Principais**: O coração do negócio
- **Subdomínios de Suporte**: Operações específicas importantes para o negócio
- **Subdomínios Genéricos**: Funcionalidades comuns que não são específicas do negócio

##### Camadas na Arquitetura DDD

- **Camada de Interface do Usuário**: Responsável pela interação com o usuário
- **Camada de Aplicação**: Orquestra objetos do domínio para realizar tarefas
- **Camada de Domínio**: Contém o modelo e a lógica de negócio
- **Camada de Infraestrutura**: Fornece capacidades técnicas para as outras camadas

##### Refatoração Orientada a Domínio

O DDD enfatiza a refatoração contínua do código para alinhar melhor o modelo de software ao modelo de domínio. Isso envolve:

1. **Destilação**: Separar o núcleo do domínio dos componentes genéricos
2. **Refinamento**: Melhorar continuamente o modelo à medida que o entendimento do domínio evolui
3. **Grandes refatorações**: Ocasionalmente, redesenhar partes significativas do sistema

##### Implementação do DDD

##### Event Sourcing

Padrão complementar onde todas as mudanças no estado de um aplicativo são armazenadas como uma sequência de eventos.

##### CQRS (Command Query Responsibility Segregation)

Separação entre operações que alteram o estado (comandos) e operações que leem o estado (consultas).

##### Arquitetura Hexagonal / Portas e Adaptadores

Arquitetura que isola o núcleo do domínio de tecnologias e frameworks externos.

##### Benefícios do DDD

- Melhor alinhamento entre software e negócio
- Modelos mais ricos e expressivos
- Design mais claro e organizado para domínios complexos
- Melhor comunicação entre especialistas técnicos e de domínio
- Software mais adaptável às mudanças de requisitos de negócio

O DDD não é indicado para todos os tipos de projetos, sendo mais valioso em domínios complexos onde a lógica de negócio é sofisticada e em constante evolução.

## Interface

Interfaces são contratos que definem um conjunto de métodos que uma classe deve implementar.

Segundo a Oracle: "Uma interface em Java é um tipo de referência, similar a uma classe, que pode conter apenas constantes, assinaturas de métodos, métodos default, métodos estáticos e tipos aninhados." ([Oracle Java Documentation](https://docs.oracle.com/javase/tutorial/java/IandI/createinterface.html))

Benefícios de usar interfaces:

- Permitem polimorfismo
- Promovem acoplamento fraco
- Facilitam testes unitários
- Permitem implementações múltiplas (em linguagens que não suportam herança múltipla)

## Abstract Class

Classes abstratas são classes que não podem ser instanciadas diretamente e geralmente contêm métodos abstratos que devem ser implementados por subclasses.

De acordo com a Microsoft: "As classes abstratas são usadas para criar uma hierarquia de classes onde uma classe base fornece uma implementação parcial que é compartilhada por várias classes derivadas." ([Microsoft .NET Documentation](https://learn.microsoft.com/pt-br/dotnet/csharp/programming-guide/classes-and-structs/abstract-and-sealed-classes-and-class-members))

Características das classes abstratas:

- Podem conter métodos abstratos e concretos
- Não podem ser instanciadas diretamente
- Podem implementar interfaces
- Podem conter construtores, campos e propriedades

## Client Code

Client Code refere-se ao código que utiliza uma API, biblioteca ou framework.

Segundo a Google Cloud: "O código cliente é a parte da aplicação que interage com serviços externos ou bibliotecas, e deve ser projetado para ser resiliente a falhas e mudanças nas dependências." ([Google Cloud API Design Guide](https://cloud.google.com/apis/design))

Boas práticas para código cliente:

- Usar interfaces em vez de implementações concretas
- Implementar tratamento adequado de erros
- Seguir o princípio de responsabilidade única
- Usar padrões de design apropriados para interações com sistemas externos

## Design Patterns

Padrões de Design são soluções típicas para problemas comuns no design de software.

A Oracle define: "Padrões de design são soluções comprovadas para problemas recorrentes que surgem durante o desenvolvimento de software." ([Oracle Java Documentation](https://www.oracle.com/technical-resources/articles/java/patterns-api-design.html))

Categorias de padrões de design:

- **Padrões Criacionais**: Lidam com mecanismos de criação de objetos
- **Padrões Estruturais**: Lidam com composição de classes e objetos
- **Padrões Comportamentais**: Lidam com interações entre objetos e responsabilidades

## Padrão de Design Decorator

O Decorator é um padrão estrutural que permite adicionar comportamentos a objetos dinamicamente, envolvendo-os em objetos wrapper.

Segundo a Microsoft: "O padrão Decorator permite adicionar responsabilidades a objetos individuais de forma dinâmica e transparente, sem afetar outros objetos." ([Microsoft .NET Design Patterns](https://learn.microsoft.com/pt-br/dotnet/standard/design-guidelines/))

Componentes do padrão Decorator:

- **Componente**: Interface para objetos que podem ter responsabilidades adicionadas
- **Componente Concreto**: Implementação básica do componente
- **Decorator**: Classe abstrata que implementa a interface do componente e mantém uma referência a um objeto componente
- **Decoradores Concretos**: Adicionam responsabilidades ao componente

## Padrão de Design Strategy

O Strategy é um padrão comportamental que define uma família de algoritmos, encapsula cada um deles e os torna intercambiáveis.

A AWS descreve: "O padrão Strategy permite que o algoritmo varie independentemente dos clientes que o utilizam, facilitando a troca de comportamentos em tempo de execução." ([AWS Architecture Center](https://aws.amazon.com/architecture/))

Componentes do padrão Strategy:

- **Contexto**: Mantém uma referência a uma estratégia
- **Estratégia**: Interface comum para todas as estratégias concretas
- **Estratégias Concretas**: Implementações diferentes do algoritmo

## JSON (JavaScript Object Notation)

JSON é um formato leve de intercâmbio de dados, fácil para humanos lerem e escreverem e fácil para máquinas analisarem e gerarem.

De acordo com a Google Cloud: "JSON é um formato de texto independente de linguagem que é amplamente utilizado para transmitir dados estruturados pela web." ([Google Cloud JSON API](https://cloud.google.com/apis/design/resources#json))

Características do JSON:

- Baseado em pares chave-valor
- Suporta tipos de dados como strings, números, booleanos, arrays e objetos
- Amplamente suportado em diversas linguagens de programação
- Formato padrão para APIs modernas

## API REST (Representational State Transfer)

REST é um estilo arquitetural para sistemas distribuídos, amplamente utilizado para criar serviços web.

Segundo a Microsoft Azure: "REST é um estilo de arquitetura que define um conjunto de restrições a serem usadas para criar serviços web que fornecem interoperabilidade entre sistemas de computador na Internet." ([Microsoft Azure REST API Guidelines](https://github.com/microsoft/api-guidelines/blob/vNext/azure/Guidelines.md))

Princípios das APIs REST:

- **Interface Uniforme**: Simplifica e desacopla a arquitetura
- **Stateless**: Cada requisição do cliente contém todas as informações necessárias
- **Cacheable**: As respostas devem definir se são cacheáveis ou não
- **Cliente-Servidor**: Separação de responsabilidades
- **Sistema em Camadas**: Hierarquia de componentes
- **Código sob Demanda (opcional)**: Capacidade de enviar código executável do servidor para o cliente

A AWS complementa: "APIs RESTful tiram proveito dos métodos HTTP padrão (GET, POST, PUT, DELETE) para operações em recursos, identificados por URIs específicas." ([AWS API Gateway](https://aws.amazon.com/api-gateway/))
