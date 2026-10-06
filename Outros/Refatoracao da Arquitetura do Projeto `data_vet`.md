
## Objetivo desta mudanca



Este documento descreve, em detalhes, o que foi feito na refatoracao do codigo e por que cada decisao foi tomada.

  
A solicitacao foi:

  

1. Validar se o apontamento de dependencias (item 1) estava correto.

2. Aplicar uma nova estrutura modular de diretorios.

3. Corrigir os problemas tecnicos dos itens 4 a 7.

4. Aplicar boas praticas (Clean Code, Object Calisthenics e separacao de responsabilidades).

  

---

  

## Contexto tecnico inicial

  

O arquivo original `src/data_vet/cordinates.py` concentrava:

  

- leitura de arquivo GeoJSON;

- busca de territorio;

- transformacoes de CRS;

- simplificacao com buffer;

- geracao de grade hexagonal;

- renderizacao do mapa com Folium;

- gravacao do HTML;

- execucao imediata em nivel de modulo.

  

Esse formato funcionava como "script unico", mas dificultava:

  

- manutencao;

- extensao;

- teste unitario;

- reutilizacao de partes isoladas.

  

---

  

## Principais problemas identificados anteriormente

  

1. Dependencia `folium` ausente no ambiente de execucao.

2. Caminho de GeoJSON fragil no codigo legado.

3. Efeitos colaterais no import (codigo executava automaticamente).

4. Falha potencial com `StopIteration` na busca de territorio.

5. Risco de comparacao espacial com CRS inconsistente.

6. `while True` sem limite (risco de loop infinito).

7. Variavel de saida definida sem consolidacao clara de fluxo.

  

---

  

## O que foi feito

  

## 1) Nova estrutura de diretorios (modular)

  

Foi aplicada a estrutura modular sugerida:

  

```text

src/data_vet/

  cli/

    build_map.py

  config/

    paths.py

    settings.py

  geo/

    crs.py

    territory.py

    simplify.py

    points.py

  grid/

    hexagon.py

  io/

    geojson_loader.py

    shapefile_to_geojson.py

  maps/

    folium_renderer.py

  cordinates.py

  geopandas_file.py

  folium_config.py

```

  

### Por que essa estrutura

  

- `config/`: centraliza configuracao e evita "string magica" espalhada.

- `io/`: separa entrada/saida de arquivos da regra de negocio.

- `geo/`: encapsula operacoes geoespaciais puras.

- `grid/`: isola a logica matematica da grade hexagonal.

- `maps/`: concentra apenas renderizacao Folium.

- `cli/`: define o fluxo de orquestracao (caso de uso principal).

  

Essa separacao reduz acoplamento e facilita testes por unidade.

  

---

  

## 2) Configuracao e paths centralizados

  

### Arquivos criados

  

- `src/data_vet/config/settings.py`

- `src/data_vet/config/paths.py`

- `src/data_vet/config/__init__.py`

  

### O que mudou e por que

  

- Criadas `dataclasses` imutaveis (`frozen=True`, `slots=True`) para configuracoes:

  - `TerritoryConfig`

  - `BufferConfig`

  - `MapBuildConfig`

- Criada classe `ProjectPaths` para descobrir o root do projeto e normalizar caminhos.

- Adicionada validacao explicita para garantir existencia do GeoJSON.

  

### Beneficio

  

- Menos risco de erro de caminho em ambientes diferentes.

- Menos constantes globais espalhadas.

- Configuracao mais legivel e previsivel.

  

---

  

## 3) Separacao de I/O

  

### Arquivos criados

  

- `src/data_vet/io/geojson_loader.py`

- `src/data_vet/io/shapefile_to_geojson.py`

- `src/data_vet/io/__init__.py`

  

### O que mudou e por que

  

- Leitura de GeoJSON foi encapsulada em `GeoJsonLoader`.

- Conversao Shapefile -> GeoJSON foi encapsulada em `ShapefileToGeoJsonConverter`.

- `geopandas_file.py` deixou de conter logica inline e passou a usar o conversor.

  

### Beneficio

  

- I/O fica desacoplado da regra de negocio.

- Erros de arquivo ausente ficam centralizados e com mensagens claras.

  

---

  

## 4) Regra geoespacial em modulos especializados

  

### Arquivos criados

  

- `src/data_vet/geo/crs.py`

- `src/data_vet/geo/territory.py`

- `src/data_vet/geo/simplify.py`

- `src/data_vet/geo/points.py`

- `src/data_vet/geo/__init__.py`

  

### O que mudou e por que

  

- `crs.py`: funcoes para reprojecao de `GeoSeries` e `GeoDataFrame`.

- `territory.py`: `TerritoryFinder` com erro claro quando territorio nao existe.

- `simplify.py`: `GeometryBufferSimplifier` com limite de iteracoes.

- `points.py`: construcao de pontos com validacoes e boundary no CRS correto.

  

### Correcoes diretas dos itens 4 a 7

  

- Item 4 (erro de territorio): agora levanta `ValueError` com contexto.

- Item 5 (CRS inconsistente): boundary e pontos sao alinhados no mesmo CRS.

- Item 6 (loop infinito): adicionada condicao de parada por `max_iterations`.

- Item 7 (saida): fluxo de renderizacao finaliza com `save(...)` no renderer.

  

---

  

## 5) Modulo da grade hexagonal

  

### Arquivos criados

  

- `src/data_vet/grid/hexagon.py`

- `src/data_vet/grid/__init__.py`

  

### O que mudou e por que

  

- A geracao de centros da malha hexagonal foi movida para a classe `HexagonGrid`.

- Validacao de `radius_m > 0`.

- Metodo unico de responsabilidade: `centers_covering(...)`.

  

### Beneficio

  

- Reaproveitamento da grade sem depender de Folium ou IO.

- Mais facil testar somente o algoritmo de distribuicao.

  

---

  

## 6) Renderizacao do mapa isolada

  

### Arquivos criados

  

- `src/data_vet/maps/folium_renderer.py`

- `src/data_vet/maps/__init__.py`

  

### O que mudou e por que

  

- Toda logica Folium foi movida para `FoliumCoverageMapRenderer`.

- Renderizacao dividida em metodos pequenos:

  - `_add_territory`

  - `_add_expanded_territory`

  - `_add_centers`

  - `_add_circle`

  - `_add_draw_controls`

- `render(...)` virou ponto unico de geracao e salvamento.

  

### Beneficio

  

- Menor complexidade ciclomatica por metodo.

- Fluxo visual do mapa fica centralizado e mais simples de alterar.

  

---

  

## 7) Orquestracao em CLI

  

### Arquivos criados

  

- `src/data_vet/cli/build_map.py`

- `src/data_vet/cli/__init__.py`

  

### O que mudou e por que

  

- Criado fluxo principal `build_map(...)` que orquestra:

  1. descoberta de caminhos;

  2. leitura de dados;

  3. selecao do territorio;

  4. simplificacao;

  5. geracao da grade;

  6. renderizacao final.

- Criado `main()` dedicado para execucao por linha de comando.

  

### Beneficio

  

- Nao ha mais execucao acidental no import de modulos de dominio.

- Ponto de entrada claro para automacao.

  

---

  

## 8) Compatibilidade com o modulo legado

  

### Arquivos alterados

  

- `src/data_vet/cordinates.py`

- `src/data_vet/folium_config.py`

  

### O que mudou e por que

  

- `cordinates.py` foi mantido como fachada de compatibilidade.

- As funcoes antigas continuam disponiveis, mas delegam para os modulos novos.

- `folium_config.py` exporta o renderer novo para evitar modulo vazio.

  

### Beneficio

  

- Menor impacto para quem ainda importa caminhos antigos.

- Migracao gradual sem quebrar uso existente imediatamente.

  

---

  

## 9) Scripts de execucao no `pyproject.toml`

  

### Alteracao

  

Foi adicionado:

  

```toml

[project.scripts]

data-vet-build-map = "data_vet.cli.build_map:main"

data-vet-build-geojson = "data_vet.geopandas_file:main"

```

  

### Por que

  

- Facilita uso padronizado por CLI.

- Evita execucao direta por caminho de arquivo interno.

  

---

  

## Praticas de codigo aplicadas (Clean Code / Object Calisthenics)

  

As principais praticas aplicadas foram:

  

1. **Single Responsibility**

   Cada modulo cuida de um tipo de responsabilidade (config, IO, geo, grid, maps, cli).

  

2. **Funcoes e metodos menores**

   O fluxo grande foi decomposto em metodos pequenos e coesos.

  

3. **Encapsulamento**

   Estado interno de classes em atributos privados (`_campo`).

  

4. **Imutabilidade onde faz sentido**

   Configuracoes com `dataclass(frozen=True, slots=True)`.

  

5. **Validacao antecipada**

   Erros de entrada/estado sao detectados cedo com mensagens claras.

  

6. **Eliminacao de efeitos colaterais no import**

   Execucao principal movida para `main()` e funcoes de orquestracao.

  

7. **Nomes semanticos**

   Classes e metodos nomeados por intencao (`TerritoryFinder`, `GeometryBufferSimplifier`, `FoliumCoverageMapRenderer`).

  

8. **Dependencias explicitas**

   O orquestrador monta os objetos e deixa claro o fluxo.

  

---

  

## Validacoes realizadas durante a mudanca

  

1. Compilacao sintatica dos arquivos Python:

   - `python -m py_compile ...` em todos os modulos criados/alterados.

2. Verificacao de importacao de dependencias:

   - `folium` ainda ausente no ambiente do terminal desta sessao.

3. Verificacao estrutural:

   - nova arvore de diretorios presente em `src/data_vet`.

  

---

  

## Limitacoes atuais

  

1. O ambiente de execucao ainda nao possui `folium` instalado no interpretador usado pelo terminal.

2. Nao foi executado teste funcional end-to-end do mapa por essa dependencia ausente.

3. Nao foram adicionados testes automatizados nesta etapa (conforme combinado para outro momento).

  

---

  

## Como executar com a nova arquitetura

  

Depois de instalar dependencias no ambiente ativo, voce pode usar:

  

```bash

poetry run data-vet-build-geojson

poetry run data-vet-build-map

```

  

Ou, sem instalar como script:

  

```bash

PYTHONPATH=src python -m data_vet.cli.build_map

```


  

---

  

## Resumo executivo

  

A mudanca converteu um script monolitico em uma arquitetura modular, com responsabilidades claras, melhor legibilidade, melhor manutencao e base mais preparada para testes.  

Os itens tecnicos 4 a 7 foram corrigidos no processo.  

O item 1 esta correto em declaracao de dependencias, mas o ambiente atual ainda precisa da instalacao efetiva de `folium` para execucao completa.