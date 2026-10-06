# Tradução do trecho da PEP 8 do Python Sobre Underscore

Além disso, as seguintes formas especiais que usam sublinhados no início ou no fim são reconhecidas (estas geralmente podem ser combinadas com qualquer convenção de capitalização):

- `_sublinhado_único_no_início`: indicador fraco de "uso interno". Ex.: `from M import *` não importa objetos cujos nomes começam com um sublinhado.
    
- `sublinhado_único_no_final_`: usado por convenção para evitar conflitos com palavras-chave do Python, por exemplo:
    

```
tkinter.Toplevel(master, class_='ClassName')
```

- `__sublinhado_duplo_no_início`: quando nomear um atributo de classe, invoca a modificação de nome (dentro da classe FooBar, `__boo` se torna `_FooBar__boo`; veja abaixo).
    
- `__sublinhado_duplo_no_início_e_no_final__`: objetos ou atributos "mágicos" que existem em espaços de nomes controlados pelo usuário. Ex.: `__init__`, `__import__` ou `__file__`. Nunca invente tais nomes; use-os apenas conforme documentado.

# Tradução do trecho Nomes de Métodos e Variáveis de Instância PEP 8 - Python

Use as regras de nomenclatura de funções: letras minúsculas com palavras separadas por sublinhados conforme necessário para melhorar a legibilidade.

Use apenas um sublinhado no início para métodos e variáveis de instância não públicos.

Para evitar conflitos de nomes com subclasses, use dois sublinhados no início para invocar as regras de modificação de nomes do Python.

O Python modifica esses nomes com o nome da classe: se a classe Foo tem um atributo chamado `__a`, ele não pode ser acessado por `Foo.__a`. (Um usuário insistente ainda poderia obter acesso chamando `Foo._Foo__a`.) Geralmente, sublinhados duplos no início devem ser usados apenas para evitar conflitos de nomes com atributos em classes projetadas para serem herdadas.

Nota: existe alguma controvérsia sobre o uso de nomes com __ (veja abaixo).

---

### Significado de `def _exemplo()`

O underscore no início (`_exemplo`) **não impede o acesso** ao método, mas **indica para outros desenvolvedores que ele é um método auxiliar (ou interno)** e **não deveria ser usado fora da classe ou módulo onde foi definido**.

---

### Exemplos práticos

```python
class MinhaClasse:
    def publico(self):
        print("Este é um método público")

    def _interno(self):
        print("Este é um método 'privado' (convenção)")
```

```python
obj = MinhaClasse()
obj.publico()   # OK
obj._interno()  # Funciona, mas não é recomendado usar diretamente
```

---
### Por que usar isso?

1. **Organização do código**: separa a lógica principal dos métodos auxiliares.
    
2. **Indicação clara para outros desenvolvedores**: "isso aqui é interno, não dependa disso".
    
---

### Diferença entre `_exemplo` e `__exemplo`

|Forma|Significado|
|---|---|
|`_exemplo`|Convenção de **uso interno** (protegido)|
|`__exemplo`|**Name mangling** – renomeia internamente para evitar sobrescrita em subclasses|
|`__exemplo__`|Método especial (dunder), como `__init__`, `__str__`, etc.|

---