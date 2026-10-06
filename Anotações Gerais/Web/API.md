Uma API (Interface de Programação de Aplicações) é um conjunto de regras que permite que diferentes softwares se comuniquem entre si. Funciona como um garçom em um restaurante: você faz um pedido (requisição), o garçom leva seu pedido para a cozinha (servidor) e depois traz o que você pediu (resposta). As APIs permitem que aplicativos, sites e sistemas acessem funcionalidades ou dados de outros serviços sem precisar conhecer como esses serviços funcionam internamente.

Basicamente uma API é contrato de como os dados serão enviados e quais respostas vamos obter quando um dado específico for solicitado.
# API REST e API RESTful

Uma API (Interface de Programação de Aplicações) REST e uma API RESTful são conceitos fundamentais na comunicação entre sistemas de software modernos. Vou explicar cada uma detalhadamente.

## API REST

REST (Representational State Transfer) é um estilo arquitetural para sistemas distribuídos criado por Roy Fielding em 2000 na sua tese de doutorado. Uma API REST é uma interface que segue os princípios e restrições do estilo REST para permitir a comunicação entre sistemas via HTTP.

### Princípios fundamentais de uma API REST:

1. **Arquitetura Cliente-Servidor**: Separação clara entre cliente (solicitante) e servidor (provedor dos recursos).
    
2. **Stateless (Sem Estado)**: Cada requisição contém toda informação necessária para ser compreendida e processada pelo servidor, sem depender de informações de requisições anteriores.
    
3. **Cache**: As respostas devem ser explicitamente marcadas como cacheáveis ou não, para melhorar a eficiência da rede.
    
4. **Interface Uniforme**: Simplifica a arquitetura e desacopla o cliente do servidor. Consiste em:
    
    - Identificação de recursos (através de URIs)
    - Manipulação de recursos através de representações
    - Mensagens auto-descritivas
    - HATEOAS (Hypermedia as the Engine of Application State)
5. **Sistema em Camadas**: A arquitetura é composta por camadas hierárquicas, onde cada camada só conhece a camada com a qual interage diretamente.
    
6. **Código sob Demanda (opcional)**: O servidor pode temporariamente estender a funcionalidade do cliente transferindo código executável.
    

## API RESTful

Uma API RESTful é uma implementação que segue rigorosamente todos os princípios e restrições do estilo arquitetural REST. Muitas APIs que se declaram REST na verdade são "REST-like" ou "RESTish", pois não implementam todos os princípios.

### Características de uma API RESTful verdadeira:

1. **Utilização correta dos métodos HTTP**:
    
    - GET: Recuperar dados
    - POST: Criar novos recursos
    - PUT: Atualizar recursos existentes (substituição completa)
    - PATCH: Atualização parcial de recursos
    - DELETE: Remover recursos
2. **Recursos bem definidos**: Cada recurso é identificado por uma URI específica.
    
    ```
    /usuarios           (coleção de usuários)
    /usuarios/123       (usuário específico)
    /usuarios/123/posts (posts do usuário 123)
    ```
    
3. **Utilização adequada dos códigos de status HTTP**:
    
    - 2xx: Sucesso (200 OK, 201 Created, 204 No Content)
    - 3xx: Redirecionamento
    - 4xx: Erro do cliente (400 Bad Request, 401 Unauthorized, 404 Not Found)
    - 5xx: Erro do servidor
4. **HATEOAS (Hypermedia as the Engine of Application State)**: As respostas incluem links para ações relacionadas, permitindo que o cliente navegue pela API sem conhecimento prévio de sua estrutura.
    
5. **Representações múltiplas**: O mesmo recurso pode ser representado em diferentes formatos (JSON, XML, HTML) usando negociação de conteúdo.
    
6. **Versionamento**: Normalmente implementado via URL (/api/v1/recursos) ou cabeçalhos HTTP.
    

## Exemplo prático

### Requisição em uma API RESTful:

```
GET /api/v1/produtos/45 HTTP/1.1
Host: minhalojaonline.com
Accept: application/json
```

### Resposta:

```
HTTP/1.1 200 OK
Content-Type: application/json

{
  "id": 45,
  "nome": "Smartphone XYZ",
  "preco": 999.99,
  "estoque": 50,
  "_links": {
    "self": {"href": "/api/v1/produtos/45"},
    "categorias": {"href": "/api/v1/produtos/45/categorias"},
    "avaliações": {"href": "/api/v1/produtos/45/avaliacoes"},
    "relacionados": {"href": "/api/v1/produtos/45/relacionados"}
  }
}
```

Note que a resposta inclui links HATEOAS que permitem ao cliente navegar para outros recursos relacionados.

## Diferenças entre REST e RESTful

- **REST** é um estilo arquitetural, um conjunto de princípios e restrições.
- **RESTful** se refere a uma API que implementa completamente os princípios REST, especialmente HATEOAS.

## Vantagens das APIs RESTful

1. **Escalabilidade**: Por ser stateless, facilita a escalabilidade horizontal.
2. **Independência de plataforma**: Pode ser utilizada por qualquer cliente que entenda HTTP.
3. **Flexibilidade**: Suporta diferentes formatos de dados (JSON, XML).
4. **Visibilidade**: As operações são explícitas através dos verbos HTTP.
5. **Confiabilidade**: Por usar HTTP, pode aproveitar características como cache e segurança.

## Limitações

1. **HATEOAS**: É raramente implementado completamente, devido à complexidade.
2. **Overhead**: Para operações simples, pode haver sobrecarga de processamento.
3. **Stateless**: Para algumas aplicações, manter estado pode ser necessário.

Uma API REST bem projetada é fundamental para sistemas modernos distribuídos, microserviços e aplicações web, facilitando a integração e comunicação entre diferentes sistemas e plataformas.