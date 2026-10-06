
---

## O que é e para que serve

**Teste unitário** é um teste automatizado que verifica o comportamento de uma unidade isolada de código, geralmente uma função ou método de forma independente do restante do sistema.

**Para que serve:**

- Garantir que cada função faz o que deveria
- Detectar regressões ao modificar código
- Documentar o comportamento esperado
- Facilitar refatoração com segurança

**Vantagens:** feedback rápido, código mais confiável, e os testes servem como documentação viva do sistema.

---

## Padrão AAA — Arrange, Act, Assert

Todo teste bem escrito segue três etapas claras:

- **Arrange (preparar):** configure o estado inicial — crie objetos, defina variáveis de entrada, configure mocks.
- **Act (agir):** execute a unidade que está sendo testada — chame a função ou método.
- **Assert (verificar):** confirme que o resultado corresponde ao esperado. Aqui ficam todos os `assert`.

```python
def test_soma():
    # Arrange
    a, b = 2, 3

    # Act
    resultado = soma(a, b)

    # Assert
    assert resultado == 5
```

---

## Nomeação de testes

O nome do teste deve contar uma história: **o que está sendo testado**, **qual o cenário** e **qual o resultado esperado**.

```
# Padrão: test_[função]_[cenário]_[resultado esperado]

def test_dividir_por_zero_levanta_excecao(): ...
def test_login_credenciais_invalidas_retorna_false(): ...
def test_calcular_desconto_usuario_vip_retorna_20_porcento(): ...
```

> Um bom nome elimina a necessidade de ler o corpo do teste para entender o que ele verifica.

---

## Mock

**Mock** é um objeto substituto que imita o comportamento de uma dependência real (banco de dados, API, e-mail) para que o teste seja rápido e isolado.

```python
from unittest.mock import MagicMock, patch

def test_enviar_email():
    with patch("servico.enviar_email") as mock_email:
        mock_email.return_value = True

        resultado = processar_pedido(pedido_fake)

        mock_email.assert_called_once()
        assert resultado is True
```

### Exemplo com mocker (pytest-mock)

```python
# pip install pytest-mock

def test_buscar_usuario_api(mocker):
    # Arrange
    mock_get = mocker.patch("servico.requests.get")
    mock_get.return_value.json.return_value = {"id": 1, "nome": "Ana"}
    mock_get.return_value.status_code = 200

    # Act
    usuario = buscar_usuario(id=1)

    # Assert
    mock_get.assert_called_once_with("https://api.exemplo.com/usuarios/1")
    assert usuario["nome"] == "Ana"
```

> Com `mocker`, não é necessário o `with patch(...)` — a limpeza do mock acontece automaticamente ao fim do teste.

### MagicMock (unittest) vs mocker (pytest)

| MagicMock (unittest) | mocker (pytest)              |                          |
| -------------------- | ---------------------------- | ------------------------ |
| **Origem**           | Stdlib (`unittest.mock`)     | Biblioteca `pytest-mock` |
| **Sintaxe**          | `with patch("...") as mock:` | `mocker.patch("...")`    |
| **Integração**       | Manual                       | Nativa com pytest        |
| **Recomendação**     | Qualquer projeto Python      | Projetos usando pytest   |

Ambos fazem a mesma coisa. Com pytest, prefira `mocker.patch()` para uma sintaxe mais limpa.

---

## Fixtures

Fixtures são funções que preparam (e limpam) o contexto necessário para um teste. Substituem o `setUp/tearDown` do unittest de forma mais flexível.

```python
import pytest

@pytest.fixture
def usuario():
    return Usuario(nome="Ana", email="ana@email.com")

def test_nome_usuario(usuario):   # pytest injeta automaticamente
    assert usuario.nome == "Ana"
```

Fixtures podem ter escopos que controlam quando são criadas e destruídas:

|Escopo|Comportamento|
|---|---|
|`function` (padrão)|Recriada a cada teste|
|`class`|Compartilhada dentro da classe|
|`module`|Compartilhada dentro do arquivo|
|`session`|Criada uma vez por execução|

---
## Coverage

**Coverage** (cobertura) mede qual porcentagem do código é executada durante os testes. Alta cobertura não garante qualidade, mas baixa cobertura garante que há código não testado.

```bash
# Instalar
pip install pytest-cov

# Rodar com relatório no terminal
pytest --cov=src --cov-report=term-missing

# Gerar relatório HTML
pytest --cov=src --cov-report=html
```

> Mire em 80–90% de cobertura. 100% pode ser sinal de testes que verificam implementação ao invés de comportamento.

---

## Estrutura de diretórios de teste

```
projeto/
├── src/
│   ├── __init__.py
│   ├── servicos/
│   │   └── pedido.py
│   └── modelos/
│       └── usuario.py
├── tests/
│   ├── __init__.py
│   ├── conftest.py          # fixtures globais
│   ├── test_pedido.py       # espelha src/servicos/
│   └── test_usuario.py      # espelha src/modelos/
├── pytest.ini               # configurações do pytest
└── pyproject.toml
```

> Mantenha a estrutura de `tests/` espelhando `src/`. Facilita encontrar qual teste corresponde a qual módulo.

---

## TDD — Test-Driven Development (visão geral)

TDD é uma metodologia onde você escreve o teste **antes** do código de produção. O ciclo é:

1. 🔴 **Red** — escreva um teste que falha. Ele descreve o comportamento que você quer implementar.
2. 🟢 **Green** — escreva o mínimo de código para o teste passar.
3. 🔵 **Refactor** — melhore o código sem quebrar os testes. Repita o ciclo.

> Por ora, apenas entenda o conceito. O objetivo é que os testes guiem o design do código, não que você escreva código e depois teste.

---

## conftest.py

Arquivo especial do pytest onde você define **fixtures compartilhadas** entre múltiplos arquivos de teste. O pytest o descobre automaticamente — sem precisar importar.

```python
# tests/conftest.py
import pytest
from myapp.db import criar_db_teste

@pytest.fixture(scope="session")
def db():
    banco = criar_db_teste()
    yield banco          # tudo após yield = teardown
    banco.limpar()
```

> Use `yield` ao invés de `return` quando precisar de teardown (limpeza após o teste).

---

## pytest.mark.parametrize

Permite rodar o mesmo teste com **múltiplos conjuntos de dados** sem duplicar código. Cada conjunto vira um caso de teste independente.

```python
@pytest.mark.parametrize("entrada, esperado", [
    (2,   4),
    (3,   9),
    (-4,  16),
    (0,   0),
])
def test_quadrado(entrada, esperado):
    assert quadrado(entrada) == esperado
```

O pytest nomeia cada caso automaticamente: `test_quadrado[2-4]`, `test_quadrado[3-9]`, etc. Você pode adicionar IDs customizados com `ids=["zero", "positivo", ...]`.

### Exemplo com múltiplos parâmetros e IDs customizados

```python
@pytest.mark.parametrize("senha, esperado", [
    ("abc",          False),   # muito curta
    ("semNumero!",   False),   # sem dígito
    ("SemEspecial1", False),   # sem caractere especial
    ("Valida@123",   True),    # válida
], ids=["curta", "sem_numero", "sem_especial", "valida"])
def test_validar_senha(senha, esperado):
    assert validar_senha(senha) == esperado
```

Aqui cada parâmetro tem um ID legível. O relatório mostrará: `test_validar_senha[curta]`, `test_validar_senha[sem_numero]`, etc. — muito mais fácil de depurar do que índices numéricos.

---

## API e Parâmetros do Pytest

### Marks úteis

```python
@pytest.mark.skip(reason="não implementado ainda")
@pytest.mark.xfail(reason="bug conhecido")
@pytest.mark.slow  # agrupamento customizado
```

### pytest.raises

```python
def test_divisao_por_zero():
    with pytest.raises(ValueError):
        dividir(1, 0)
```

### Comandos de linha de comando

```bash
pytest                        # roda todos os testes
pytest tests/test_pedido.py   # arquivo específico
pytest -k "desconto"          # filtra por nome
pytest -v                     # verbose
pytest -x                     # para no primeiro erro
pytest --tb=short             # traceback resumido
pytest -s                     # exibe prints durante os testes
```

### Configuração básica (pytest.ini)

```ini
[pytest]
testpaths = tests
python_files = test_*.py
python_functions = test_*
```