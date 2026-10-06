## 1) Visão geral

Este repositório implementa uma API REST em Python para registrar leituras de energia de smart meters e comparar planos de energia. A arquitetura foi organizada por camadas para separar responsabilidades:

- Entrada HTTP (controllers)
- Regras de negócio (services)
- Persistência em memória (repositories)
- Modelos de domínio (domain)
- Geração de dados de exemplo (generator)

A aplicação utiliza **Connexion + Flask**, com contrato OpenAPI definido em `swagger.yml`.

---

## 2) Estrutura de pastas e arquivos principais
 
├── README.md  
├── LICENSE  
├── requirements.txt  
├── src/  
│ ├── app.py  
│ ├── config.py  
│ ├── app_initializer.py  
│ ├── swagger.yml  
│ ├── controller/  
│ ├── service/  
│ ├── repository/  
│ ├── domain/  
│ └── generator/  
└── test/  
├── controller/  
├── service/  
├── repository/  
├── domain/  
└── generator/


---

## 3) Camada de inicialização e configuração

### Arquivo: `src/config.py`

- Cria o objeto Connexion apontando o diretório de especificação.
- Expõe o app Flask interno para execução.

### Arquivo: `src/app.py`

- Ponto de entrada da aplicação.
- Registra o contrato da API via `swagger.yml`.
- No boot principal:
  - Inicializa dados mock em memória.
  - Sobe o servidor na porta padrão (5000, conforme README).

### Arquivo: `src/app_initializer.py`

- Popula dados iniciais para facilitar testes locais:
  - Leituras aleatórias para vários medidores.
  - Planos de preço pré-definidos.

---

## 4) Contrato da API

### Arquivo: `src/swagger.yml`

- Define schemas de entrada e saída.
- Mapeia endpoints para funções Python via `operationIds`.

### Endpoints principais

- `POST /readings/store`  
  Armazena leituras de um smart meter.

- `GET /readings/read/{smart_meter_id}`  
  Recupera leituras armazenadas para um smart meter.

- `GET /price-plans/compare-all/{smart_meter_id}`  
  Compara custo entre planos para o medidor.

- `GET /price-plans/recommend/{smart_meter_id}`  
  Retorna recomendações de planos com parâmetro opcional `limit`.

---

## 5) Controllers (camada HTTP)

**Pasta:** `src/controller/`

### Arquivos

- `electricity_reading_controller.py`
  - Recebe payload de leitura e delega ao service.
  - Busca leituras por smart meter e retorna 404 quando vazio.

- `price_plan_comparator_controller.py`
  - Realiza comparação de custos por plano.
  - Retorna plano atual e comparativos.
  - Exponibiliza recomendação de planos.

### Responsabilidade da camada

- Traduz requisição HTTP em chamadas de service.
- Controla códigos de erro HTTP (ex.: 404 quando não há dados).
- Serializa resposta para JSON.

---

## 6) Services (regras de negócio)

**Pasta:** `src/service/`

### Arquivos

- `electricity_reading_service.py`
  - Converte JSON recebido em objetos de domínio `ElectricityReading`.
  - Persiste e consulta leituras no repositório.

- `price_plan_service.py`
  - Calcula consumo e custo para cada plano.
  - Ordena planos por menor custo.
  - Aplica `limit` quando informado.

- `account_service.py`
  - Mapeia smart meter para plano contratado.

- `time_converter.py`
  - Funções utilitárias para conversão de tempo.

### Responsabilidade da camada

- Centralizar lógica de negócio.
- Impedir que controllers conheçam detalhes de persistência.

---

## 7) Repositories (persistência em memória)

**Pasta:** `src/repository/`

### Arquivos

- `electricity_reading_repository.py`
  - Estrutura em memória para associar `smart_meter -> lista de leituras`.
  - Operações: `store`, `find`, `clear`.

- `price_plan_repository.py`
  - Lista em memória com planos de preço.
  - Operações: `store`, `get`, `clear`.
  - Exponibiliza instância global `price_plan_repository`.

### Responsabilidade da camada

- Isolar acesso e armazenamento de dados.
- Facilitar testes sem banco de dados externo.

---

## 8) Domain (modelos centrais)

**Pasta:** `src/domain/`

### Arquivos

- `electricity_reading.py`
  - Modelo de leitura com campos `time` e `reading`.
  - Conversão para JSON via `to_json()`.

- `price_plan.py`
  - Modelo de plano com nome, fornecedor, tarifa base e multiplicadores opcionais.
  - Método `get_price` para calcular tarifa efetiva por data.

### Responsabilidade da camada

- Representar entidades do negócio com estrutura clara.

---

## 9) Generator (dados de suporte)

**Pasta:** `src/generator/`

### Arquivo

- `electricity_reading_generator.py`
  - Gera leituras aleatórias para popular dados iniciais.
  - Simula histórico temporal para cada smart meter.

### Responsabilidade da camada

- Apoiar boot local e cenários de teste com dados sintéticos.

---

## 10) Testes

**Pasta:** `test/`

Subpastas espelham `src`:

- `test/controller/`
- `test/service/`
- `test/repository/`
- `test/domain/`
- `test/generator/`

### Objetivo

- Validar comportamento de cada camada de forma isolada.
- Reduzir regressões ao alterar regras de negócio ou endpoints.

---

## 11) Fluxo de execução da requisição (ponta a ponta)

### Exemplo: `POST /readings/store`

1. Endpoint definido em `swagger.yml`.
2. Connexion encaminha para `controller.electricity_reading_controller.store`.
3. Controller chama `ElectricityReadingService`.
4. Service transforma JSON em objetos de domínio.
5. Repository grava dados em memória.
6. Controller devolve resposta JSON.

### Exemplo: comparação de planos

1. Requisição chega ao `price_plan_comparator_controller`.
2. Controller consulta `PricePlanService`.
3. Service busca leituras, calcula consumo e custo por plano.
4. Controller monta resposta com comparativos e recomendações.

---

## 12) Resumo arquitetural

- Organização em camadas simples e didáticas.
- API dirigida por contrato (OpenAPI em `swagger.yml`).
- Persistência em memória, adequada para desafio técnico e testes locais.
- Estrutura facilita manutenção, extensão e cobertura por testes.



