## Builder

Imagine que você precisa criar um objeto com muitos parâmetros opcionais, como uma classe `Casa` que pode ter garagem, jardim, piscina, número variável de quartos, etc. Sem o Builder, você enfrentaria problemas como:

- Construtores com muitos parâmetros (telescoping constructor problem)
- Dificuldade para lembrar a ordem dos parâmetros
- Código pouco legível
- Impossibilidade de criar objetos parcialmente configurados

## Como funciona

O Builder encapsula a lógica de construção em uma classe separada, permitindo que você configure o objeto através de métodos encadeados antes de construí-lo definitivamente.

**Estrutura básica:**

- **Product**: O objeto complexo sendo construído
- **Builder**: Interface que define os passos de construção
- **ConcreteBuilder**: Implementação específica do builder
- **Director** (opcional): Classe que conhece a sequência de construção

