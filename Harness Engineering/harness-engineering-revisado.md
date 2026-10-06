# Harness Engineering para agentes de programação

## 1. O que é Harness Engineering?

Em agentes de programação, *Harness Engineering* é a prática de projetar o ambiente ao redor do modelo para aumentar a probabilidade de que ele produza software correto, verificável, manutenível e repetível.

Uma forma simples de representar a ideia é:

~~~text
Agente de programação = Modelo + Harness
~~~

O modelo fornece capacidade de raciocínio e geração. O *harness* reúne o que permite que essa capacidade seja aplicada ao repositório: contexto, instruções, ferramentas, estado persistente, ambiente de execução, regras arquiteturais, testes, observabilidade, revisão e mecanismos de correção.

Martin Fowler usa “harness” em sentido amplo: tudo que envolve o modelo e ajuda a direcionar e verificar o trabalho do agente. Isso inclui tanto os mecanismos fornecidos pela ferramenta quanto aqueles construídos pela equipe para adaptar o agente ao seu código, à sua arquitetura e ao seu processo. O objetivo não é tornar o modelo infalível, mas aumentar a confiança no resultado e reduzir a necessidade de supervisão manual repetitiva. [Martin Fowler](https://martinfowler.com/articles/harness-engineering.html)

### A analogia com o harness horse racing

![[Pasted image 20260723203537.png]]
Na analogia:

- o cavalo representa o modelo, que possui capacidade de execução;
- o condutor representa a direção humana e a definição de intenção;
- os arreios e o veículo representam ferramentas, instruções, limites e mecanismos de controle;
- a pista representa o ambiente do projeto;
- as regras, sensores e resultados da corrida representam os critérios de validação.
## 2. A mudança no papel do engenheiro

O artigo da OpenAI descreve um experimento em que uma equipe construiu um produto com código, testes, documentação, configuração de CI, observabilidade e ferramentas internas produzidos pelo Codex. O resultado não é apresentado como uma receita universal, mas como um estudo de caso de uma equipe que investiu fortemente no ambiente do agente. [OpenAI](https://openai.com/pt-BR/index/harness-engineering/)

Nesse cenário, o trabalho do engenheiro muda de foco. Em vez de escrever diretamente cada linha de código, ele passa a:

- especificar a intenção e os critérios de aceitação;
- decompor objetivos grandes em unidades que o agente consiga executar;
- organizar o conhecimento necessário no repositório;
- projetar ferramentas, abstrações e ambientes de execução;
- estabelecer limites arquiteturais;
- construir testes, linters e outros sensores de feedback;
- observar falhas recorrentes e transformar suas soluções em regras permanentes;
- decidir onde a supervisão humana continua sendo necessária.

Na experiência da OpenAI, quando o agente encontrava dificuldades, a resposta normalmente não era apenas pedir uma nova tentativa. A pergunta era: qual informação, ferramenta, abstração ou verificação está faltando para que o agente consiga realizar a tarefa? A solução era incorporada ao próprio repositório, fazendo com que futuras execuções pudessem se beneficiar dela.

Isso transforma o trabalho do engenheiro em um ciclo de direcionamento: observar um problema, entender por que o ambiente permitiu que ele ocorresse e melhorar o *harness* para reduzir a probabilidade de repetição.

## 3. Como o Harness orienta e verifica o agente

Fowler organiza os mecanismos do *harness* em dois papéis complementares: *guides* e *sensors*. Eles também podem ser classificados pela forma de execução: controles computacionais ou inferenciais. São classificações diferentes; um *guide* ou *sensor* pode usar uma técnica computacional ou inferencial. [Martin Fowler](https://martinfowler.com/articles/harness-engineering.html)

### Guides: controles de orientação

*Guides* são mecanismos de *feedforward*: atuam antes ou durante a geração para orientar o comportamento do agente e aumentar a chance de uma boa primeira tentativa.

Exemplos:

- AGENTS.md;
- documentação de arquitetura;
- convenções de código;
- *skills* e instruções de processo;
- especificações funcionais;
- ferramentas de bootstrap;
- scripts e *codemods*;
- servidores MCP e outras fontes de contexto.

### Sensors: sensores de feedback

*Sensors* observam o resultado produzido e devolvem sinais que permitem ao agente detectar e corrigir problemas.

Exemplos:

- testes;
- linters e formatadores;
- verificação de tipos;
- análise estática;
- testes estruturais;
- logs, métricas e rastreamentos;
- revisão de código;
- testes de arquitetura;
- testes de mutação;
- métricas de execução e qualidade em produção.

Uma mensagem de erro bem escrita não apenas informa que algo falhou: ela também explica o que precisa ser corrigido. Por isso, mensagens de linters podem funcionar como uma forma de instrução automática para o agente.

Somente orientação pode criar regras que nunca são verificadas. Somente feedback pode fazer o agente repetir uma abordagem ruim até encontrar, por acaso, uma solução. Um *harness* precisa dos dois.

### Controles computacionais e inferenciais

Controles computacionais são determinísticos, rápidos e executados por ferramentas tradicionais. Testes, linters, *type checkers* e análises estruturais são exemplos. Por serem baratos, podem ser executados com frequência, inclusive a cada alteração.

Controles inferenciais dependem de análise semântica, de um modelo ou de um agente revisor. Revisão de código por LLM e “LLM como juiz” são exemplos. Eles podem avaliar aspectos mais subjetivos, mas tendem a ser mais lentos, caros e não determinísticos.

Uma estratégia prática é usar controles computacionais sempre que o problema puder ser descrito por uma regra verificável e reservar controles inferenciais para situações que exigem julgamento semântico. [Martin Fowler](https://martinfowler.com/articles/harness-engineering.html)

## 4. O problema do excesso e do mau gerenciamento de contexto

Um agente depende do contexto que consegue acessar e utilizar durante a execução. Isso não significa que fornecer mais texto sempre produza melhores resultados.

O estudo da Chroma sobre *context rot* avaliou diversos modelos e observou que o desempenho pode se tornar menos confiável à medida que o tamanho da entrada aumenta. Os modelos não utilizam necessariamente todas as partes do contexto de maneira uniforme; comprimento, distrações e distância entre a informação e a pergunta podem afetar o resultado. [Chroma](https://www.trychroma.com/research/context-rot)

Na prática, um repositório pode sofrer um problema semelhante quando suas instruções são:

- longas demais;
- repetitivas;
- contraditórias;
- difíceis de localizar;
- desatualizadas;
- misturadas com detalhes irrelevantes para a tarefa atual.

O artigo da OpenAI relata uma tentativa de concentrar todo o conhecimento em um único arquivo grande, o “one big AGENTS.md”. A abordagem falhou porque o contexto é um recurso limitado, porque o excesso de orientação dificulta a identificação do que é realmente importante, porque regras antigas envelhecem e porque um arquivo monolítico é difícil de verificar automaticamente. [OpenAI](https://openai.com/pt-BR/index/harness-engineering/)

O problema, portanto, não é apenas “ter muito contexto”. É não possuir uma organização que permita ao agente começar por um ponto pequeno, encontrar a informação relevante e distinguir instruções atuais de conhecimento histórico ou específico.

## 5. AGENTS.md como índice e docs/ como sistema de registro

Como resposta ao problema do arquivo monolítico, a OpenAI passou a tratar AGENTS.md como um mapa de navegação, não como uma enciclopédia. O arquivo contém orientações essenciais e aponta o agente para documentos mais específicos. O conhecimento mais amplo do projeto fica organizado em docs/, tratado como sistema de registro. [OpenAI](https://openai.com/pt-BR/index/harness-engineering/)

Uma estrutura apresentada no artigo é semelhante a esta:

~~~text
AGENTS.md
ARCHITECTURE.md
docs/
├── design-docs/
├── exec-plans/
│   ├── active/
│   ├── completed/
│   └── tech-debt-tracker.md
├── generated/
├── product-specs/
├── references/
├── DESIGN.md
├── FRONTEND.md
├── PLANS.md
├── PRODUCT_SENSE.md
├── QUALITY_SCORE.md
├── RELIABILITY.md
└── SECURITY.md
~~~

O papel de cada parte pode ser resumido assim:

| Elemento         | Papel                                                                                          |
| ---------------- | ---------------------------------------------------------------------------------------------- |
| AGENTS.md        | Ponto de entrada com regras essenciais e indicações de onde buscar mais contexto.              |
| ARCHITECTURE.md  | Mapa de alto nível da estrutura do sistema e de suas fronteiras.                               |
| docs/            | Local organizado para especificações, decisões, referências, planos e documentação do projeto. |
| docs/exec-plans/ | Registro de planos de execução ativos e concluídos.                                            |
| docs/references/ | Materiais auxiliares que o agente pode consultar quando necessário.                            |
| PLANS.md         | Regras para criar e manter planos de execução, quando essa convenção for adotada.              |

Essa organização permite divulgação progressiva de contexto: o agente começa por um arquivo pequeno e estável, consulta a documentação relacionada à tarefa e só então carrega detalhes adicionais.

### docs/ deve ficar na raiz ou em .agents/?

No exemplo da OpenAI, docs/ fica na raiz do repositório, ao lado de AGENTS.md e ARCHITECTURE.md. Portanto, essa é a localização correta para reproduzir a estrutura descrita no artigo.

Isso não transforma a posição em uma regra universal. .agents/ pode ser utilizado por uma ferramenta para armazenar *skills*, configurações ou instruções específicas do agente. Porém, a documentação do projeto deve ficar em um local versionado, navegável e compreensível para toda a equipe, independentemente da ferramenta usada. Nesse modelo, docs/ é a base de conhecimento do repositório, enquanto .agents/ é uma possível área de configuração da ferramenta.

O ARCHITECTURE.md e o PLANS.md aparecem nessa organização como documentos fundamentais, mas seus objetivos são diferentes e merecem explicação própria.

## 6. ARCHITECTURE.md: um mapa, não um manual

Aleksey Kladov recomenda um documento ARCHITECTURE curto e de alto nível para projetos relevantes. A finalidade não é repetir a implementação nem criar mais um manual extenso. É fornecer um mapa mental do sistema para que uma pessoa — ou um agente — descubra rapidamente onde uma mudança deve ser feita. [Aleksey Kladov](https://matklad.github.io/2021/02/06/ARCHITECTURE.md.html)

Um bom ARCHITECTURE.md deve explicar:

- qual problema o projeto resolve;
- quais são seus módulos ou domínios principais;
- como esses módulos se relacionam;
- onde procurar a implementação de uma determinada responsabilidade;
- quais fronteiras e invariantes arquiteturais não podem ser violadas;
- quais preocupações atravessam várias partes do sistema.

O documento deve permanecer curto e relativamente estável. Detalhes que mudam com frequência devem ficar em documentos específicos ou na própria documentação do código. Kladov também recomenda evitar links diretos para entidades que podem mudar; nomes de módulos, arquivos e tipos podem ser mais duráveis quando utilizados como pontos de busca.

O artigo usa como exemplo o documento de arquitetura do [rust-analyzer](https://github.com/rust-lang/rust-analyzer/blob/d7c99931d05e3723d878bea5dc26766791fa4e69/docs/dev/architecture.md). O exemplo mostra como um mapa de componentes e dependências pode reduzir o tempo necessário para localizar o ponto correto de uma alteração.

## 7. PLANS.md e os planos de execução

PLANS.md define como escrever planos de execução — chamados de *ExecPlans* no artigo do OpenAI Cookbook. Não é, necessariamente, o plano de uma tarefa específica. Ele funciona como uma convenção ou um modelo para produzir planos consistentes. Os planos concretos podem ser armazenados em diretórios como docs/exec-plans/active/ e docs/exec-plans/completed/. [OpenAI Cookbook](https://developers.openai.com/cookbook/articles/codex_exec_plans)

Um ExecPlan é um documento vivo para mudanças complexas ou demoradas. Ele deve permitir que o trabalho seja retomado sem depender da memória de uma conversa anterior. Para isso, normalmente registra:

- objetivo e resultado esperado;
- contexto do problema;
- decisões de design;
- arquivos e componentes envolvidos;
- etapas de implementação;
- comandos e verificações;
- marcos de progresso;
- decisões tomadas durante a execução;
- problemas encontrados e como foram resolvidos.

Essa prática é particularmente útil para agentes porque transforma o estado do trabalho em um artefato versionado. O agente não precisa reconstruir todo o raciocínio a partir de uma conversa perdida: ele pode ler o plano, atualizar seu progresso e continuar.

ARCHITECTURE.md responde principalmente “como o sistema está organizado?”. PLANS.md e os ExecPlans respondem “como uma mudança complexa será planejada, executada e acompanhada?”.

## 8. Estratégias para impor arquitetura, implementação e qualidade

A documentação orienta, mas não garante que o código seguirá as regras. Por isso, a OpenAI combina contexto legível com restrições verificáveis.

### Limites arquiteturais

Uma arquitetura com módulos e direções de dependência explícitas reduz o espaço de soluções possíveis. Em vez de permitir que qualquer módulo dependa de qualquer outro, a equipe define quais relações são válidas e verifica essas relações automaticamente.

No estudo de caso da OpenAI, os domínios são divididos em camadas com uma direção de dependência previsível. Linters personalizados e testes estruturais verificam limites entre módulos, convenções de nomenclatura, formato de logs, tamanho de arquivos e outras invariantes. A equipe mantém liberdade na implementação interna, mas restringe as fronteiras que preservam a coerência do sistema. [OpenAI](https://openai.com/pt-BR/index/harness-engineering/)

### Parse, don’t validate

O princípio “parse, don’t validate”, apresentado por Alexis King, recomenda transformar dados externos e pouco estruturados em valores internos que carreguem as garantias já verificadas. Uma função que apenas valida retorna pouca informação; uma função que interpreta e produz um tipo mais específico preserva a garantia no próprio programa. [Alexis King](https://lexi-lambda.github.io/blog/2019/11/05/parse-don-t-validate/)

Aplicado ao *harness*, o princípio favorece:

- validar dados nas fronteiras do sistema;
- converter entradas para tipos ou estruturas mais fortes;
- evitar espalhar verificações repetidas pelo código;
- permitir que o compilador, o linter e os testes detectem violações;
- reduzir estados inválidos que o agente poderia criar.

Essa abordagem não elimina toda validação. Ela organiza a validação no lugar correto e preserva o conhecimento obtido, em vez de descartá-lo após um teste pontual.

### Linters e testes estruturais

Linters personalizados são úteis quando uma regra específica do projeto não é coberta por ferramentas genéricas. Para serem eficazes com agentes, suas mensagens devem indicar o problema e, quando possível, sugerir a correção.

Testes estruturais verificam propriedades da organização do código, como dependências permitidas, fronteiras entre módulos e presença de determinados padrões. Eles não testam apenas se uma função retorna o valor esperado; verificam se a base continua obedecendo à arquitetura definida.

### Feedback rápido e ambientes legíveis

Steve Krenzel argumenta que agentes tornam mais valiosas práticas que antes eram tratadas como opcionais: testes abrangentes, tipagem, módulos pequenos, documentação clara, ambientes fáceis de iniciar e verificações automáticas. O autor também destaca que o sistema de arquivos é uma interface de navegação para o agente: nomes e diretórios bem definidos ajudam o modelo a localizar o contexto correto. [Steve Krenzel](https://bits.logic.inc/p/ai-is-forcing-us-to-write-good-code)

Isso sugere três características práticas:

1. **Feedback rápido:** verificações precisam ser baratas para que possam ser executadas repetidamente durante a tarefa.
2. **Ambientes reproduzíveis:** o agente deve conseguir iniciar, testar e descartar ambientes de trabalho sem configuração manual extensa.
3. **Estrutura explícita:** arquivos pequenos, nomes específicos e módulos bem delimitados reduzem ambiguidades e facilitam a recuperação de contexto.

Essas práticas não são garantias absolutas. Cobertura alta pode coexistir com testes ruins, tipos não representam toda a regra de negócio e uma arquitetura rígida pode ser mal desenhada. O papel do *harness* é tornar as decisões importantes explícitas e verificáveis, não substituir o julgamento sobre o produto.

## 9. Conclusão

Harness Engineering é uma forma de tratar o desenvolvimento com agentes como um problema de engenharia de sistemas. O modelo é apenas uma parte do agente. A confiabilidade depende também da qualidade do contexto, das ferramentas disponíveis, da organização do repositório, dos limites arquiteturais e dos ciclos de feedback.

O modelo resumido é:

~~~text
Intenção humana
      ↓
Guides → Agente → Sensors → Correção
      ↑                         ↓
      └──── melhoria do Harness ┘
~~~

O caminho apresentado pelas fontes é progressivo:

- manter AGENTS.md pequeno e usá-lo como índice;
- organizar o conhecimento versionado em docs/;
- utilizar ARCHITECTURE.md para oferecer um mapa estável do sistema;
- utilizar PLANS.md e ExecPlans para registrar mudanças complexas;
- transformar limites e invariantes em linters e testes estruturais;
- colocar verificações rápidas no início do ciclo;
- observar falhas recorrentes e incorporá-las ao *harness*;
- preservar a supervisão humana onde o comportamento ou o julgamento ainda não podem ser verificados com confiança.

O objetivo não é eliminar o engenheiro. É deslocar sua atenção da correção repetitiva de sintomas para o projeto do ambiente que permite ao agente produzir resultados confiáveis em escala.

## Referências

- [OpenAI — Alavancando o Codex em um mundo centrado no agente](https://openai.com/pt-BR/index/harness-engineering/)
- [Martin Fowler — Harness engineering for coding agent users](https://martinfowler.com/articles/harness-engineering.html)
- [Chroma — Context Rot: How Increasing Input Tokens Impacts LLM Performance](https://www.trychroma.com/research/context-rot)
- [Aleksey Kladov — ARCHITECTURE.md](https://matklad.github.io/2021/02/06/ARCHITECTURE.md.html)
- [matklad — Example of ARCHITECTURE.md no rust-analyzer](https://github.com/rust-lang/rust-analyzer/blob/d7c99931d05e3723d878bea5dc26766791fa4e69/docs/dev/architecture.md)
- [OpenAI Cookbook — Using PLANS.md for multi-hour problem solving](https://developers.openai.com/cookbook/articles/codex_exec_plans)
- [Alexis King — Parse, don’t validate](https://lexi-lambda.github.io/blog/2019/11/05/parse-don-t-validate/)
- [Steve Krenzel — AI Is Forcing Us To Write Good Code](https://bits.logic.inc/p/ai-is-forcing-us-to-write-good-code)
