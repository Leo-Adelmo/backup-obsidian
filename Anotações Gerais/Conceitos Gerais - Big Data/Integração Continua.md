# O que é CI/CD?

## Construindo fluxos de trabalho automatizados para lançamentos mais rápidos

### O que é CI/CD?

CI/CD significa Integração Contínua e Deployment Contínuo (ou Entrega Contínua). É um conjunto de práticas e ferramentas projetadas para melhorar o processo de desenvolvimento de software ao automatizar builds, testes e deployment, permitindo que você entregue mudanças de código mais rapidamente e de forma confiável.

**Integração contínua (CI):** Constrói, testa e integra automaticamente mudanças de código dentro de um repositório compartilhado

**Entrega contínua (CD):** Entrega automaticamente mudanças de código para ambientes prontos para produção para aprovação

**Deployment contínuo (CD):** Faz deployment automaticamente de mudanças de código diretamente para clientes

CI/CD compreende integração contínua e entrega contínua ou deployment contínuo. Juntos, eles formam um "pipeline CI/CD" — uma série de fluxos de trabalho automatizados que ajudam equipes DevOps a reduzir tarefas manuais.

### Exemplo de um pipeline CI/CD

Um pipeline típico inclui as seguintes etapas: planejamento, codificação, build, teste, criação de artefatos, staging e produção.

### Entrega contínua vs. deployment contínuo

Quando alguém diz CI/CD, o "CD" ao qual se referem geralmente é entrega contínua, não deployment contínuo. Qual é a diferença? Em um pipeline CI/CD que usa entrega contínua, a automação pausa quando desenvolvedores fazem push para produção. Um humano — sua equipe de operações, segurança ou conformidade — ainda precisa aprovar manualmente antes do lançamento final, adicionando mais atrasos. Por outro lado, o deployment contínuo automatiza todo o processo de lançamento. Mudanças de código são deployadas para clientes assim que passam por todos os testes necessários.

O deployment contínuo é o exemplo definitivo de automação DevOps. Isso não significa que seja a única forma de fazer CI/CD, ou a forma "certa". Como o deployment contínuo depende de ferramentas de teste rigorosas e uma cultura de teste madura, a maioria das equipes de software começam com entrega contínua e integram mais testes automatizados ao longo do tempo.

## Por que CI/CD?

A resposta curta: Velocidade. O relatório State of DevOps descobriu que organizações que "dominaram" CI/CD fazem deploy 208 vezes mais frequentemente e têm um lead time que é 106 vezes mais rápido que o resto. Embora desenvolvimento mais rápido seja o benefício mais conhecido do CI/CD, um pipeline de integração contínua e entrega contínua possibilita muito mais.

**Velocidade de desenvolvimento:** Feedback contínuo permite que desenvolvedores façam commit de mudanças menores com mais frequência, versus esperar por um lançamento.

**Estabilidade e confiabilidade:** Testes automatizados e contínuos garantem que bases de código permaneçam estáveis e prontas para lançamento a qualquer momento.

**Crescimento do negócio:** Liberadas de tarefas manuais, organizações podem focar recursos em inovação, satisfação do cliente e pagamento de débito técnico.

> "A mentalidade que carregamos é que sempre queremos nos automatizar para um trabalho melhor. Queremos garantir que a tarefa que estamos fazendo manualmente hoje se torne principalmente automatizada."
> 
> — Andrew Mulholland, Diretor de Engenharia

## Construindo seu kit de ferramentas CI/CD

Equipes fazem CI/CD parte de seu fluxo de trabalho de desenvolvimento com uma combinação de processos automatizados, etapas e ferramentas.

### Controle de versão

CI começa em repositórios compartilhados, onde equipes colaboram em código usando sistemas de controle de versão (VCS) como Git. Um VCS rastreia mudanças de código, simplifica reversões e suporta configuração como código para gerenciar testes e infraestrutura.

### Builds

Ferramentas de build CI empacotam automaticamente arquivos e componentes em artefatos de lançamento e executam testes para qualidade, performance e outros requisitos. Após passar pelas verificações necessárias, ferramentas CD enviam builds para a equipe de operações para testes adicionais e staging.

### Revisões e aprovações

Tratar revisão de código como uma melhor prática melhora a qualidade do código, encoraja colaboração e ajuda até mesmo os desenvolvedores mais experientes a fazer commits melhores. Em um fluxo de trabalho CI/CD, equipes revisam e aprovam código ou aproveitam ambientes de desenvolvimento integrados para programação em pares.

### Ambientes

CI/CD testa e faz deploy de código em ambientes, desde onde desenvolvedores constroem código até onde equipes de operações tornam aplicações publicamente disponíveis. Ambientes frequentemente têm suas próprias variáveis específicas e regras de proteção para atender requisitos de segurança e conformidade.

## Exemplo de fluxo de trabalho CI/CD

CI/CD não precisa ser complicado, ou significar adicionar uma série de ferramentas em cima do seu fluxo de trabalho atual. Na mabl, desenvolvedores fazem deploy para produção cerca de 80 vezes por semana usando apenas duas integrações CI/CD: A suíte de testes mabl e GitHub Actions. Veja como funciona:

1. Desenvolvedores abrem pull requests para disparar builds iniciais e testes unitários
2. Commits aprovados são deployados para um ambiente de preview
3. GitHub Actions customizadas instalam a CLI mabl e executam testes headless
4. GitHub Apps fornecem resultados de verificação ao vivo dentro de pull requests
5. Commits aprovados são mesclados na branch principal para testes adicionais ou deployados para produção

## O que torna CI/CD bem-sucedido

Você encontrará diferentes ferramentas e integrações em todos os lugares que olhar, mas fluxos de trabalho CI/CD eficazes compartilham os mesmos marcadores de sucesso.

### Automação

CI/CD pode ser feito manualmente — mas esse não é o objetivo. Um bom fluxo de trabalho CI/CD automatiza builds, testes e deployment para que você tenha mais tempo para código, não mais tarefas para fazer.

### Transparência

Se um build falha, desenvolvedores precisam ser capazes de rapidamente avaliar o que deu errado e por quê. Logs, construtores de fluxo de trabalho visuais e ferramentas profundamente integradas tornam mais fácil para desenvolvedores solucionar problemas, entender fluxos de trabalho complexos e compartilhar seu status com a equipe maior.

### Velocidade

CI/CD contribui para sua performance DevOps geral, particularmente velocidade. Especialistas DevOps medem velocidade usando duas métricas DORA: Lead time para mudanças (quão rapidamente commits são feitos para código em produção) e frequência de deployment (com que frequência você faz commit de código).

### Resiliência

Quando usado com outras abordagens como cobertura de testes, ferramentas de observabilidade e feature flags, CI/CD torna software mais resistente a erros. DORA mede essa estabilidade rastreando tempo médio para resolução (quão rapidamente incidentes são resolvidos) e taxa de falha de mudanças (o número de rollbacks de software).

### Segurança

Automação inclui segurança. Com DevSecOps ganhando força, um pipeline CI/CD à prova de futuro tem verificações em lugar para código e permissões, e fornece uma trilha de papel virtual para auditar falhas, violações de segurança, eventos de não conformidade.

### Escalabilidade

CI/CD não é apenas sobre automação; também é sobre garantir escalabilidade. Uma configuração CI/CD robusta deve expandir sem esforço com sua equipe de desenvolvimento em crescimento e complexidade de projeto. Isso significa que pode lidar eficientemente com cargas de trabalho aumentadas conforme seus esforços de desenvolvimento de software crescem, mantendo produtividade e eficiência.

## Veja o que é possível com CI/CD: Histórias reais de clientes

Veja como equipes DevOps colocam automação contínua em prática.

**Blue Yonder:** Migrando de servidores internos para CI/CD baseado em nuvem.

**Plaid:** Melhorando tempo de deployment e produtividade do desenvolvedor.

**3M:** Quebrando silos com ferramentas compartilhadas e automação.

## O que você pode fazer com CI/CD?

Esteja você pronto para mergulhar ou ainda tenha dúvidas, nós temos você coberto.

- **Explore melhores práticas:** Aprenda mais sobre estratégias eficazes
- **Obtenha uma demo do GitHub:** Veja como CI/CD, automação e segurança de classe mundial podem apoiar seu fluxo de trabalho
- **Pergunte aos especialistas:** Construa uma estratégia personalizada para seus objetivos de negócio em uma sessão 1:1 com líderes de produto do GitHub
- **Compare soluções DevOps:** Veja como o GitHub se compara a outras ferramentas e plataformas DevOps

## Comece a construir seu fluxo de trabalho CI/CD

Esteja você pronto para mergulhar ou ainda tenha dúvidas, nós temos você coberto.

- Inicie um teste gratuito
- Entre em contato com vendas

## Mais recursos DevOps

### O que é DevOps?

Ao reunir pessoas, processos e produtos, DevOps permite que equipes de desenvolvimento entreguem valor continuamente.

### Explore os fundamentos da Integração Contínua

Descubra os fundamentos da Integração Contínua (CI) em DevOps. Aprenda como práticas CI simplificam a mesclagem, construção e teste de código de múltiplos contribuidores, facilitando desenvolvimento de software mais rápido e lançamentos de maior qualidade.

### Deployment Contínuo Explicado

Aprenda os essenciais do Deployment Contínuo (CD) no cenário DevOps. CD automatiza o processo de deployment, garantindo que mudanças de código sejam automaticamente enviadas para produção após passar nos testes.