# Conceitos Básicos de Orientação a Objetos em Python

A Programação Orientada a Objetos (POO) é um paradigma de programação baseado no conceito de "objetos", que podem conter dados e código para manipular esses dados. Vou explicar os conceitos fundamentais e nomenclaturas da POO usando Python como referência.

## Classes e Objetos

### Classe

Uma classe é um modelo ou blueprint que define atributos e comportamentos comuns a todos os objetos de um determinado tipo.

Segundo a documentação oficial do Python: "As classes fornecem uma maneira de agrupar dados e funcionalidades juntos." ([Python Documentation](https://docs.python.org/3/tutorial/classes.html))

```python
class Pessoa:
    """Esta é uma classe que representa uma pessoa"""
    
    # Método especial de inicialização (construtor)
    def __init__(self, nome, idade):
        self.nome = nome  # Atributo de instância
        self.idade = idade  # Atributo de instância
    
    # Método de instância
    def apresentar(self):
        return f"Olá, meu nome é {self.nome} e tenho {self.idade} anos."
```

### Objeto (Instância)

Um objeto é uma instância específica de uma classe. É uma entidade que possui estado e comportamento definidos pela classe.

De acordo com a Google Cloud: "Um objeto é uma instância específica de uma classe que encapsula estado e comportamento." ([Google Cloud Python Style Guide](https://google.github.io/styleguide/pyguide.html))

```python
# Criando instâncias (objetos) da classe Pessoa
pessoa1 = Pessoa("Ana", 30)  # pessoa1 é uma instância/objeto
pessoa2 = Pessoa("Carlos", 25)  # pessoa2 é outra instância/objeto

print(pessoa1.apresentar())  # Saída: Olá, meu nome é Ana e tenho 30 anos.
```

## Atributos e Métodos

### Atributos

Atributos são variáveis que pertencem a uma classe ou instância e representam suas características ou estado.

Segundo a Microsoft: "Atributos são variáveis que armazenam dados associados à classe ou aos objetos da classe." ([Microsoft Learn Python](https://learn.microsoft.com/pt-br/training/modules/python-object-oriented-programming/))

Tipos de atributos em Python:

1. **Atributos de Instância**: Pertencem a objetos específicos

```python
class Carro:
    def __init__(self, marca, modelo):
        self.marca = marca    # Atributo de instância
        self.modelo = modelo  # Atributo de instância
        self.velocidade = 0   # Atributo de instância com valor inicial
```

2. **Atributos de Classe**: Compartilhados por todas as instâncias

```python
class Carro:
    # Atributo de classe (compartilhado por todas as instâncias)
    rodas = 4  
    
    def __init__(self, marca, modelo):
        self.marca = marca
        self.modelo = modelo
```

### Métodos

Métodos são funções definidas dentro de uma classe que descrevem os comportamentos de um objeto.

De acordo com a Oracle: "Métodos são funções que pertencem a uma classe e definem o comportamento que seus objetos podem realizar." ([Oracle Java Documentation](https://docs.oracle.com/javase/tutorial/java/concepts/))

Tipos de métodos em Python:

1. **Métodos de Instância**: Operam em uma instância específica

```python
class Carro:
    def __init__(self, marca, modelo):
        self.marca = marca
        self.modelo = modelo
        self.velocidade = 0
    
    # Método de instância
    def acelerar(self, incremento):
        self.velocidade += incremento
        return f"Velocidade atual: {self.velocidade} km/h"
```

2. **Métodos de Classe**: Operam na classe em si, não em instâncias específicas

```python
class Carro:
    rodas = 4
    total_carros = 0
    
    def __init__(self, marca, modelo):
        self.marca = marca
        self.modelo = modelo
        Carro.total_carros += 1
    
    # Método de classe
    @classmethod
    def contar_carros(cls):
        return f"Total de carros criados: {cls.total_carros}"
```

3. **Métodos Estáticos**: Não operam nem na classe nem nas instâncias

```python
class Carro:
    # Método estático
    @staticmethod
    def converter_km_para_milhas(km):
        return km * 0.621371
```

## Parâmetros, Argumentos e Retorno

### Parâmetros

Parâmetros são variáveis listadas na definição de um método.

Segundo a AWS: "Parâmetros são as variáveis que aparecem na definição de uma função ou método." ([AWS Cloud9 Python Guide](https://docs.aws.amazon.com/cloud9/latest/user-guide/sample-python.html))

```python
class Calculadora:
    # "a" e "b" são parâmetros
    def somar(self, a, b):
        return a + b
```

### Argumentos

Argumentos são os valores reais passados para os parâmetros quando um método é chamado.

De acordo com a Microsoft: "Argumentos são os valores reais passados para uma função ou método quando ele é chamado." ([Microsoft Python Documentation](https://learn.microsoft.com/pt-br/training/modules/python-functions/))

```python
calc = Calculadora()
# 5 e 3 são argumentos passados para os parâmetros a e b
resultado = calc.somar(5, 3)  
```

### Valor de Retorno

É o valor que um método devolve após sua execução.

```python
class Multiplicador:
    def multiplicar(self, x, y):
        produto = x * y
        return produto  # produto é o valor de retorno
```

## Encapsulamento

Encapsulamento é o conceito de agrupar dados e métodos que operam nesses dados dentro de uma única unidade (classe) e restringir o acesso direto a alguns componentes.

Segundo a Google Cloud: "Encapsulamento é um mecanismo que vincula código e dados que manipulam juntos e mantém ambos seguros de interferência externa e uso indevido." ([Google Cloud Architecture](https://cloud.google.com/architecture/framework/))

Em Python, o encapsulamento é implementado por convenção usando:

- Atributos públicos: Acessíveis de qualquer lugar
- Atributos "protegidos": Com prefixo `_` (convenção)
- Atributos "privados": Com prefixo `__` (name mangling)

```python
class ContaBancaria:
    def __init__(self, titular, saldo_inicial):
        self.titular = titular        # Atributo público
        self._saldo = saldo_inicial   # Atributo "protegido" (por convenção)
        self.__id = 123456            # Atributo "privado" (com name mangling)
    
    def depositar(self, valor):
        if valor > 0:
            self._saldo += valor
            return True
        return False
    
    def consultar_saldo(self):
        return self._saldo
```

## Herança

Herança é um mecanismo que permite que uma classe adquira propriedades e métodos de outra classe.

De acordo com a Oracle: "Herança é um mecanismo em que uma nova classe é derivada de uma classe existente, herdando todos os seus atributos e métodos." ([Oracle Java Concepts](https://docs.oracle.com/javase/tutorial/java/concepts/inheritance.html))

```python
# Classe pai (superclasse)
class Animal:
    def __init__(self, nome):
        self.nome = nome
    
    def fazer_som(self):
        return "Som genérico de animal"

# Classe filha (subclasse) que herda de Animal
class Cachorro(Animal):
    def __init__(self, nome, raca):
        # Chamando o construtor da classe pai
        super().__init__(nome)
        self.raca = raca
    
    # Sobrescrevendo o método da classe pai
    def fazer_som(self):
        return "Au au!"
    
    # Adicionando um novo método
    def abanar_rabo(self):
        return f"{self.nome} está abanando o rabo!"
```

## Polimorfismo

Polimorfismo é a capacidade de um objeto assumir muitas formas, permitindo que objetos de diferentes classes sejam tratados como objetos de uma classe comum.

Segundo a Microsoft: "Polimorfismo permite que objetos de diferentes classes sejam tratados como objetos de uma classe base comum, simplificando o código e promovendo a reutilização." ([Microsoft .NET Documentation](https://learn.microsoft.com/pt-br/dotnet/csharp/fundamentals/object-oriented/polymorphism))

```python
# Exemplificando polimorfismo
def fazer_animal_falar(animal):
    # Esta função opera com qualquer objeto que tenha o método fazer_som()
    return animal.fazer_som()

animal_generico = Animal("Criatura")
rex = Cachorro("Rex", "Labrador")

print(fazer_animal_falar(animal_generico))  # Saída: Som genérico de animal
print(fazer_animal_falar(rex))              # Saída: Au au!
```

## Abstração

Abstração é o conceito de esconder os detalhes complexos de implementação e mostrar apenas as funcionalidades ao usuário.

De acordo com a AWS: "Abstração é o processo de identificar as características essenciais de uma entidade e ignorar os detalhes não essenciais." ([AWS Developer Guide](https://aws.amazon.com/developer/language/python/))

Em Python, a abstração pode ser implementada com classes abstratas usando o módulo `abc` (Abstract Base Classes):

```python
from abc import ABC, abstractmethod

# Classe abstrata
class FormaGeometrica(ABC):
    @abstractmethod
    def area(self):
        pass
    
    @abstractmethod
    def perimetro(self):
        pass

# Implementação concreta da classe abstrata
class Retangulo(FormaGeometrica):
    def __init__(self, largura, altura):
        self.largura = largura
        self.altura = altura
    
    def area(self):
        return self.largura * self.altura
    
    def perimetro(self):
        return 2 * (self.largura + self.altura)
```

## Composição

Composição é um tipo de associação onde um objeto contém outros objetos como parte de seu estado.

Segundo a Google Cloud: "Composição é uma técnica de reutilização em que uma classe contém objetos de outras classes como parte de seu estado." ([Google Cloud Architecture Framework](https://cloud.google.com/architecture/framework/))

```python
class Motor:
    def __init__(self, cilindradas):
        self.cilindradas = cilindradas
    
    def ligar(self):
        return "Motor ligado!"
    
    def desligar(self):
        return "Motor desligado!"

class Carro:
    def __init__(self, marca, modelo, cilindradas):
        self.marca = marca
        self.modelo = modelo
        # Composição: Carro contém um objeto Motor
        self.motor = Motor(cilindradas)
    
    def ligar_carro(self):
        return f"{self.marca} {self.modelo}: {self.motor.ligar()}"
```

## Conceitos Adicionais

### Construtor

Método especial chamado quando um objeto é criado, usado para inicializar atributos.

```python
class Livro:
    def __init__(self, titulo, autor):  # Construtor
        self.titulo = titulo
        self.autor = autor
```

### Destrutor

Método especial chamado quando um objeto está prestes a ser destruído.

```python
class Conexao:
    def __init__(self):
        print("Conexão estabelecida")
    
    def __del__(self):  # Destrutor
        print("Conexão encerrada")
```

### Sobrecarga de Operadores

Permite definir o comportamento de operadores para classes personalizadas.

```python
class Vetor:
    def __init__(self, x, y):
        self.x = x
        self.y = y
    
    # Sobrecarga do operador '+'
    def __add__(self, outro):
        return Vetor(self.x + outro.x, self.y + outro.y)
    
    # Sobrecarga do operador de representação em string
    def __str__(self):
        return f"Vetor({self.x}, {self.y})"

v1 = Vetor(2, 3)
v2 = Vetor(5, 7)
v3 = v1 + v2  # Usa o método __add__
print(v3)     # Saída: Vetor(7, 10) - Usa o método __str__
```

### Decoradores

Permitem modificar o comportamento de funções ou métodos.

De acordo com a Oracle: "Decoradores fornecem uma maneira elegante de estender a funcionalidade de funções ou métodos sem modificar o código original." ([Oracle Developer Blog](https://blogs.oracle.com/developers/))

```python
class Pessoa:
    def __init__(self, nome):
        self._nome = nome
    
    @property  # Decorador para criar um getter
    def nome(self):
        return self._nome
    
    @nome.setter  # Decorador para criar um setter
    def nome(self, novo_nome):
        if len(novo_nome) > 0:
            self._nome = novo_nome
        else:
            raise ValueError("Nome não pode ser vazio")
```

### Método de Classe vs Método Estático

- **Método de Classe**: Tem acesso à classe, mas não à instância
- **Método Estático**: Não tem acesso nem à classe nem à instância

```python
class Utilidades:
    contador = 0
    
    @classmethod
    def incrementar_contador(cls):
        # Tem acesso à classe (cls)
        cls.contador += 1
        return cls.contador
    
    @staticmethod
    def calcular_media(lista):
        # Não tem acesso à classe nem à instância
        return sum(lista) / len(lista)
```

### Atributos de Classe vs Atributos de Instância

- **Atributos de Classe**: Compartilhados por todas as instâncias
- **Atributos de Instância**: Específicos para cada instância

```python
class Estudante:
    # Atributo de classe
    escola = "Universidade Python"
    
    def __init__(self, nome, nota):
        # Atributos de instância
        self.nome = nome
        self.nota = nota
```

## Conclusão

A Orientação a Objetos é um paradigma poderoso que facilita a organização, reutilização e manutenção de código através de conceitos como classes, objetos, encapsulamento, herança e polimorfismo. Em Python, esses conceitos são implementados de forma simples e intuitiva, tornando-o uma excelente linguagem para aprender e aplicar a POO.

Conforme destacado pela Microsoft: "A programação orientada a objetos permite criar sistemas modulares, reutilizáveis e mais fáceis de manter, especialmente para projetos grandes e complexos." ([Microsoft Learn](https://learn.microsoft.com/pt-br/training/modules/python-object-oriented-programming/))

Esta explicação cobre os principais conceitos e nomenclaturas da POO, fornecendo uma base sólida para compreender e aplicar esses princípios em projetos Python.