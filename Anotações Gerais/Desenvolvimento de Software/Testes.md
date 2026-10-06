## Testes Unitários

Os testes unitários são a base da pirâmide de testes de software. São testes automatizados que verificam o comportamento de pequenas unidades de código de forma isolada - geralmente uma função, método ou classe específica.

### Características dos Testes Unitários

**Isolamento**: Cada teste deve ser independente e não depender de outros testes ou sistemas externos como bancos de dados, APIs ou arquivos. Isso garante que falhas sejam facilmente identificadas e localizadas.

**Rapidez**: Devem executar muito rapidamente, permitindo que centenas ou milhares de testes rodem em segundos. Essa velocidade é crucial para feedback rápido durante o desenvolvimento.

**Determinismo**: Um teste unitário deve sempre produzir o mesmo resultado quando executado nas mesmas condições. Não pode haver aleatoriedade ou dependência de fatores externos variáveis.

**Granularidade**: Focam em testar uma única funcionalidade ou comportamento específico, não fluxos complexos ou integrações entre múltiplas partes do sistema.

### Estrutura dos Testes Unitários

A maioria dos testes unitários segue o padrão **AAA (Arrange, Act, Assert)**:

- **Arrange**: Preparação dos dados e configuração necessária para o teste
- **Act**: Execução da funcionalidade que está sendo testada
- **Assert**: Verificação se o resultado obtido corresponde ao esperado

Exemplo em Python:

```python
def test_calcular_desconto():
    # Arrange
    preco_original = 100.0
    percentual_desconto = 10
    
    # Act
    preco_com_desconto = calcular_desconto(preco_original, percentual_desconto)
    
    # Assert
    assert preco_com_desconto == 90.0
```

### Benefícios dos Testes Unitários

**Detecção Precoce de Bugs**: Problemas são identificados logo após serem introduzidos, quando o contexto ainda está fresco na mente do desenvolvedor e o custo de correção é menor.

**Documentação Viva**: Os testes servem como documentação executável do comportamento esperado do código, mostrando como as funções devem ser usadas e o que devem retornar.

**Facilita Refatoração**: Com uma boa cobertura de testes, você pode modificar a implementação interna com confiança, sabendo que se quebrar algo, os testes irão detectar.

**Design Melhor**: Escrever testes força você a pensar na interface pública das suas classes e métodos, levando naturalmente a um código mais modular e testável.

## Mock Objects

Mocks são objetos simulados que imitam o comportamento de objetos reais de forma controlada. São fundamentais para manter o isolamento nos testes unitários, especialmente quando o código interage com dependências externas.

### Tipos de Test Doubles

**Mock**: Objetos que verificam se foram chamados corretamente, com quais parâmetros e quantas vezes. Focam no comportamento e nas interações.

**Stub**: Objetos que retornam respostas pré-programadas para chamadas específicas. Focam no estado, fornecendo dados necessários para o teste.

**Spy**: Objetos reais que são "espionados" para verificar como foram utilizados, mantendo sua funcionalidade original.

**Fake**: Implementações simplificadas que funcionam, mas não são adequadas para produção (como um banco de dados em memória).

### Quando Usar Mocks

**Dependências Externas**: Quando seu código interage com APIs, bancos de dados, sistemas de arquivos ou qualquer recurso externo que tornaria o teste lento ou não-determinístico.

**Objetos Complexos**: Para simular comportamentos de objetos que seriam difíceis ou caros de criar no ambiente de teste.

**Verificação de Interações**: Quando você precisa garantir que determinados métodos foram chamados com os parâmetros corretos.

**Cenários de Erro**: Para simular falhas e condições excepcionais que seriam difíceis de reproduzir com objetos reais.

### Exemplo Prático com Mock

Considere um serviço que envia emails:

```python
import unittest
from unittest.mock import Mock, patch

class EmailService:
    def __init__(self, email_client):
        self.email_client = email_client
    
    def enviar_boas_vindas(self, usuario):
        if not usuario.email:
            raise ValueError("Usuário deve ter email")
        
        self.email_client.enviar(
            para=usuario.email,
            assunto="Bem-vindo!",
            corpo=f"Olá {usuario.nome}, bem-vindo ao nosso sistema!"
        )

class TestEmailService(unittest.TestCase):
    def test_enviar_boas_vindas_sucesso(self):
        # Arrange
        mock_client = Mock()
        email_service = EmailService(mock_client)
        usuario = Mock()
        usuario.email = "teste@email.com"
        usuario.nome = "João"
        
        # Act
        email_service.enviar_boas_vindas(usuario)
        
        # Assert
        mock_client.enviar.assert_called_once_with(
            para="teste@email.com",
            assunto="Bem-vindo!",
            corpo="Olá João, bem-vindo ao nosso sistema!"
        )
    
    def test_enviar_boas_vindas_sem_email(self):
        # Arrange
        mock_client = Mock()
        email_service = EmailService(mock_client)
        usuario = Mock()
        usuario.email = None
        
        # Act & Assert
        with self.assertRaises(ValueError):
            email_service.enviar_boas_vindas(usuario)
        
        # Verifica que o email não foi enviado
        mock_client.enviar.assert_not_called()
```

### Boas Práticas com Mocks

**Use com Moderação**: Mocks excessivos podem tornar os testes frágeis e acoplados à implementação. Prefira objetos reais quando possível.

**Mock nas Bordas**: Mocke apenas as dependências nas bordas do seu sistema (APIs, bancos de dados), não lógica de negócio interna.

**Verify Behavior, Not Implementation**: Foque em verificar o comportamento importante, não todos os detalhes de implementação.

**Keep It Simple**: Mocks devem ser simples e diretos. Se você está criando mocks muito complexos, pode ser um sinal de que o design precisa ser repensado.

### Ferramentas Populares

**Python**: unittest.mock (nativo), pytest-mock **JavaScript**: Jest, Sinon.js **Java**: Mockito, EasyMock **C#**: Moq, NSubstitute **Ruby**: RSpec mocks

Os testes unitários com mocks formam a base de uma estratégia de testes robusta, permitindo desenvolvimento mais rápido e confiável através de feedback imediato sobre a qualidade do código.