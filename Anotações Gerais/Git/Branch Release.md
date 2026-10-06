
## Definição

Uma **Release Branch** é um tipo de ramificação (branch) usada no controle de versão Git para isolar o trabalho de preparação de uma nova versão de software, separando-o do desenvolvimento de novas funcionalidades e da linha principal de produção. Seu objetivo principal é estabilizar o código antes da entrega, permitindo ajustes finais, correção de bugs e atividades de preparação sem interromper o fluxo de novas implementações.

## Contexto em Fluxos de Trabalho

### Gitflow

No **Gitflow Workflow**, a Release Branch é criada a partir da branch de desenvolvimento (`develop`) quando um conjunto de funcionalidades está pronto e uma entrega está prestes a ocorrer. A partir desse ponto:

1. **Novas funcionalidades não são incluídas** na Release Branch — apenas correções, ajustes de configuração ou documentação relacionados ao lançamento.  
2. O isolamento permite que equipes finalizem a versão atual enquanto outras continuam o desenvolvimento de funcionalidades futuras em `develop`.  
3. Quando pronta, a Release Branch é **mesclada em `main` (ou `master`)** e etiquetada (tag) com o número da versão; em seguida, também é mesclada de volta em `develop` para propagar eventuais correções.

Este processo cria fases bem definidas no ciclo de vida do desenvolvimento e facilita a coordenação entre equipes em projetos com ciclos de lançamento agendados. 

## Comportamento e Boas Práticas

- **Origem:** normalmente criada a partir da branch de desenvolvimento (`develop`) quando um conjunto relevante de funcionalidades está consolidado.
- **Tipos de commits permitidos:** foco em **correções de bugs**, ajustes de build, versão, documentação e outras tarefas de preparação de release, não em desenvolvimento de novas funcionalidades.
- **Mesclagens:** após estabilização e testes, a Release Branch é mesclada em `main` para representar o lançamento e em `develop` para que correções importantes retornem ao fluxo de desenvolvimento. 
- **Duração:** ramificações de release são temporárias; geralmente existem apenas durante o ciclo de preparação da versão. 

## Vantagens

- **Isolamento de preparação:** permite estabilizar o código sem interromper o desenvolvimento contínuo. 
- **Coordenação de equipes:** define claramente quando o foco muda para estabilização e entrega. 
- **Controle de versões:** facilita a marcação de versões e rastreamento de mudanças específicas da release. 

## Limitações

- **Processo mais pesado:** introduz etapas adicionais em comparação com estratégias mais simples
- **Integração de longo prazo:** pode demandar mais coordenação entre merges, principalmente em equipes grandes.


