# Detailed Module Reference

Este documento é uma referência técnica detalhada da estrutura atual do projeto em `src/`.

## 1. Estrutura de diretórios

```text
src/
  main.py
  queue_control/
    __init__.py
    passenger_generation.py
    queue_operations.py
  elevator_movement/
    __init__.py
    movement_events.py
    movement_flow.py
  ui/
    __init__.py
    event_handlers.py
    ui_rich.py
```

## 2. Visão arquitetural

O projeto está organizado em três módulos principais:

- `queue_control`: geração e manipulação de filas de passageiros.
- `elevator_movement`: regras de movimento do elevador e geração de eventos de domínio.
- `ui`: tradução de eventos para saída visual/textual (Rich + mensagens de console).

`main.py` orquestra o ciclo completo:

1. gera demanda de passageiros;
2. executa fluxos de movimento;
3. despacha eventos para a UI.

## 3. Modelo de dados

### 3.1 Estado do elevador (`elevator_state`)

Estrutura esperada:

```python
{
  "current_floor": int,
  "up_queue": list[Passenger],
  "down_queue": list[Passenger],
  "down_riders": list[Passenger],
}
```

### 3.2 Estrutura de passageiro (`Passenger`)

```python
{
  "name": str,
  "origin": int,
  "destination": int,
}
```

## 4. Catálogo de eventos

Eventos de domínio emitidos por `elevator_movement`:

- `DEPART_UP`
- `MOVE` com direções: `UP`, `UP_EMPTY`, `DOWN`, `DOWN_EMPTY`
- `STOP_EXIT`
- `AFTER_EXIT`
- `SEEK_DOWN_CALLS`
- `STOP_ENTER`
- `AFTER_ENTER`
- `ARRIVE_GROUND_WITH_DOWN_RIDERS`
- `ARRIVE_GROUND_EMPTY`

## 5. Documentação por arquivo

## 5.1 `src/main.py`

### Função: `main() -> None`

| Item | Detalhe |
|---|---|
| Objetivo | Executar o loop principal da simulação. |
| Entradas | Não recebe parâmetros. Usa constantes de configuração do arquivo. |
| Saída | Não retorna valor. |
| Efeitos colaterais | Atualiza `elevator_state`, chama renderização de UI, escreve em console e aplica `sleep`. |
| Dependências principais | `populate_queues`, `create_initial_state`, `up_trip_events`, `down_service_events`, `idle_return_events`, `apply_ui_for_event`, `render_ground_status`. |

Constantes de configuração em `main.py`:

- `FLOOR_MIN = 1`
- `FLOOR_MAX = 5`
- `GROUND_FLOOR = 0`
- `SLEEP_TIME = 3`
- `QUEUE_MAX_LENGTH = 5`
- `MAX_EVENTS = 20`

## 5.2 `src/queue_control/__init__.py`

Este arquivo expõe a API pública de `queue_control` via `__all__`.

Exports:

- `create_down_passenger`
- `create_passenger`
- `create_random_direction`
- `create_random_event_count`
- `create_up_passenger`
- `populate_queues`
- `add_passenger_to_queue`
- `highest_destination`
- `highest_origin`
- `move_passengers_between_queues`
- `passengers_boarding_at_floor`
- `passengers_exiting_at_floor`
- `queue_is_full`
- `remove_passengers_from_queue`
- `select_queue_by_direction`
- `snapshot_passengers`

## 5.3 `src/queue_control/passenger_generation.py`

### Função: `create_random_event_count(max_events: int) -> int`

| Item | Detalhe |
|---|---|
| Objetivo | Gerar quantidade aleatória de eventos no ciclo atual. |
| Entradas | `max_events`: limite superior. |
| Saída | Número inteiro no intervalo `[0, max_events]`. |
| Usado por | `populate_queues`. |

### Função: `create_random_direction() -> int`

| Item | Detalhe |
|---|---|
| Objetivo | Gerar direção aleatória de viagem do passageiro. |
| Entradas | Nenhuma. |
| Saída | `1` (subida) ou `0` (descida). |
| Usado por | `populate_queues`. |

### Função: `create_up_passenger(floor_min: int, floor_max: int, ground_floor: int) -> dict`

| Item | Detalhe |
|---|---|
| Objetivo | Criar passageiro que sai do térreo para andares superiores. |
| Entradas | `floor_min`, `floor_max`, `ground_floor`. |
| Saída | `Passenger` com origem no térreo e destino aleatório acima. |
| Usado por | `create_passenger`. |

### Função: `create_down_passenger(floor_min: int, floor_max: int, ground_floor: int) -> dict`

| Item | Detalhe |
|---|---|
| Objetivo | Criar passageiro que sai de andares superiores para o térreo. |
| Entradas | `floor_min`, `floor_max`, `ground_floor`. |
| Saída | `Passenger` com destino no térreo. |
| Usado por | `create_passenger`. |

### Função: `create_passenger(direction_state: int, floor_min: int, floor_max: int, ground_floor: int) -> dict`

| Item | Detalhe |
|---|---|
| Objetivo | Selecionar tipo de passageiro com base na direção. |
| Entradas | `direction_state`, `floor_min`, `floor_max`, `ground_floor`. |
| Saída | `Passenger` de subida ou descida. |
| Usado por | `populate_queues`. |

### Função: `populate_queues(elevator_state: dict, max_events: int, queue_max_length: int, floor_min: int, floor_max: int, ground_floor: int) -> int`

| Item | Detalhe |
|---|---|
| Objetivo | Gerar eventos aleatórios e inserir passageiros nas filas corretas. |
| Entradas | Estado do elevador + parâmetros de limite e faixa de andares. |
| Saída | Quantidade de eventos gerados no ciclo. |
| Efeitos colaterais | Altera `up_queue` e `down_queue` dentro de `elevator_state`. |
| Dependências principais | `create_random_event_count`, `create_random_direction`, `create_passenger`, `select_queue_by_direction`, `add_passenger_to_queue`. |

## 5.4 `src/queue_control/queue_operations.py`

### Função: `select_queue_by_direction(elevator_state: dict, direction_state: int) -> list`

| Item | Detalhe |
|---|---|
| Objetivo | Escolher a fila de destino com base na direção (`up_queue` ou `down_queue`). |
| Entradas | `elevator_state`, `direction_state`. |
| Saída | Referência da lista de fila selecionada. |

### Função: `queue_is_full(queue: list, max_length: int) -> bool`

| Item | Detalhe |
|---|---|
| Objetivo | Verificar capacidade máxima da fila. |
| Entradas | `queue`, `max_length`. |
| Saída | `True` se cheia; caso contrário `False`. |

### Função: `add_passenger_to_queue(queue: list, passenger: dict, max_length: int) -> None`

| Item | Detalhe |
|---|---|
| Objetivo | Adicionar passageiro respeitando limite da fila. |
| Entradas | `queue`, `passenger`, `max_length`. |
| Saída | Nenhuma. |
| Efeitos colaterais | Pode mutar `queue` com `append`. |

### Função: `passengers_exiting_at_floor(queue: list, current_floor: int) -> list`

| Item | Detalhe |
|---|---|
| Objetivo | Filtrar passageiros cujo destino é o andar atual. |
| Entradas | `queue`, `current_floor`. |
| Saída | Lista de passageiros para desembarque. |

### Função: `passengers_boarding_at_floor(queue: list, current_floor: int) -> list`

| Item | Detalhe |
|---|---|
| Objetivo | Filtrar passageiros cuja origem é o andar atual. |
| Entradas | `queue`, `current_floor`. |
| Saída | Lista de passageiros para embarque. |

### Função: `remove_passengers_from_queue(queue: list, passengers: list) -> None`

| Item | Detalhe |
|---|---|
| Objetivo | Remover passageiros de uma fila. |
| Entradas | `queue`, `passengers`. |
| Saída | Nenhuma. |
| Efeitos colaterais | Remove itens de `queue` in-place. |

### Função: `move_passengers_between_queues(source_queue: list, target_queue: list, passengers: list) -> None`

| Item | Detalhe |
|---|---|
| Objetivo | Transferir passageiros de uma fila para outra. |
| Entradas | `source_queue`, `target_queue`, `passengers`. |
| Saída | Nenhuma. |
| Efeitos colaterais | Muta ambas as filas. |

### Função: `highest_destination(queue: list, fallback: int) -> int`

| Item | Detalhe |
|---|---|
| Objetivo | Obter maior destino presente na fila. |
| Entradas | `queue`, `fallback`. |
| Saída | Maior destino ou `fallback` se vazia. |

### Função: `highest_origin(queue: list, fallback: int) -> int`

| Item | Detalhe |
|---|---|
| Objetivo | Obter maior origem presente na fila. |
| Entradas | `queue`, `fallback`. |
| Saída | Maior origem ou `fallback` se vazia. |

### Função: `snapshot_passengers(passengers: list) -> list`

| Item | Detalhe |
|---|---|
| Objetivo | Criar cópia rasa da lista de passageiros para exibição segura na UI. |
| Entradas | `passengers`. |
| Saída | Nova lista com cópias de cada `dict` de passageiro. |

## 5.5 `src/elevator_movement/__init__.py`

Este arquivo expõe a API pública de `elevator_movement` via `__all__`.

Exports:

- `emit_after_enter_event`
- `emit_after_exit_event`
- `emit_arrive_ground_empty_event`
- `emit_arrive_ground_with_down_riders_event`
- `emit_move_event`
- `emit_seek_down_calls_event`
- `emit_stop_enter_event`
- `emit_stop_exit_event`
- `emit_up_depart_event`
- `create_initial_state`
- `down_service_events`
- `idle_return_events`
- `iter_boarding_floors_down`
- `iter_floors_up`
- `move_one_floor_down`
- `move_one_floor_up`
- `up_trip_events`

## 5.6 `src/elevator_movement/movement_events.py`

### Função: `emit_up_depart_event(elevator_state: dict, target_floor: int) -> dict`

| Item | Detalhe |
|---|---|
| Objetivo | Criar evento de partida da viagem de subida. |
| Entradas | `elevator_state`, `target_floor`. |
| Saída | Evento `DEPART_UP`. |

### Função: `emit_move_event(direction: str, elevator_state: dict) -> dict`

| Item | Detalhe |
|---|---|
| Objetivo | Criar evento de movimento com snapshots mínimos para a UI. |
| Entradas | `direction`, `elevator_state`. |
| Saída | Evento `MOVE`. |
| Observação | Para `UP`, inclui `up_queue`; para demais direções, inclui `down_queue` e `down_riders`. |

### Função: `emit_stop_exit_event(elevator_state: dict, exiting: list) -> dict`

| Item | Detalhe |
|---|---|
| Objetivo | Criar evento de parada com desembarque de passageiros. |
| Entradas | `elevator_state`, `exiting`. |
| Saída | Evento `STOP_EXIT`. |

### Função: `emit_after_exit_event(elevator_state: dict) -> dict`

| Item | Detalhe |
|---|---|
| Objetivo | Criar evento de estado após desembarque. |
| Entradas | `elevator_state`. |
| Saída | Evento `AFTER_EXIT`. |

### Função: `emit_seek_down_calls_event(elevator_state: dict, top_floor: int) -> dict`

| Item | Detalhe |
|---|---|
| Objetivo | Criar evento de planejamento para busca de chamadas de descida. |
| Entradas | `elevator_state`, `top_floor`. |
| Saída | Evento `SEEK_DOWN_CALLS`. |

### Função: `emit_stop_enter_event(elevator_state: dict, boarding: list) -> dict`

| Item | Detalhe |
|---|---|
| Objetivo | Criar evento de parada com embarque de passageiros. |
| Entradas | `elevator_state`, `boarding`. |
| Saída | Evento `STOP_ENTER`. |

### Função: `emit_after_enter_event(elevator_state: dict) -> dict`

| Item | Detalhe |
|---|---|
| Objetivo | Criar evento de estado após embarque. |
| Entradas | `elevator_state`. |
| Saída | Evento `AFTER_ENTER`. |

### Função: `emit_arrive_ground_with_down_riders_event(elevator_state: dict) -> dict`

| Item | Detalhe |
|---|---|
| Objetivo | Criar evento de chegada ao térreo com passageiros. |
| Entradas | `elevator_state`. |
| Saída | Evento `ARRIVE_GROUND_WITH_DOWN_RIDERS`. |

### Função: `emit_arrive_ground_empty_event(elevator_state: dict) -> dict`

| Item | Detalhe |
|---|---|
| Objetivo | Criar evento de chegada ao térreo vazio. |
| Entradas | `elevator_state`. |
| Saída | Evento `ARRIVE_GROUND_EMPTY`. |

## 5.7 `src/elevator_movement/movement_flow.py`

### Função: `create_initial_state(ground_floor: int) -> dict`

| Item | Detalhe |
|---|---|
| Objetivo | Inicializar estrutura de estado do elevador. |
| Entradas | `ground_floor`. |
| Saída | `dict` com chaves `current_floor`, `up_queue`, `down_queue`, `down_riders`. |

### Função: `move_one_floor_up(elevator_state: dict) -> None`

| Item | Detalhe |
|---|---|
| Objetivo | Incrementar um andar no estado atual. |
| Entradas | `elevator_state`. |
| Saída | Nenhuma. |
| Efeito | Muta `current_floor`. |

### Função: `move_one_floor_down(elevator_state: dict) -> None`

| Item | Detalhe |
|---|---|
| Objetivo | Decrementar um andar no estado atual. |
| Entradas | `elevator_state`. |
| Saída | Nenhuma. |
| Efeito | Muta `current_floor`. |

### Função: `iter_floors_up(start_floor: int, end_floor: int) -> Iterator[int]`

| Item | Detalhe |
|---|---|
| Objetivo | Iterar andares de subida. |
| Entradas | `start_floor`, `end_floor`. |
| Saída | Gera andares de `start_floor + 1` até `end_floor` (inclusivo). |

### Função: `iter_boarding_floors_down(start_floor: int, ground_floor: int) -> Iterator[int]`

| Item | Detalhe |
|---|---|
| Objetivo | Iterar andares de possível embarque durante descida. |
| Entradas | `start_floor`, `ground_floor`. |
| Saída | Gera andares de `start_floor` até `ground_floor + 1`. |

### Função: `emit_up_exit_events_at_current_floor(elevator_state: dict) -> Iterator[dict]`

| Item | Detalhe |
|---|---|
| Objetivo | Emitir eventos de desembarque na subida para o andar atual. |
| Entradas | `elevator_state`. |
| Saída | Pode gerar `STOP_EXIT` e `AFTER_EXIT`. |
| Efeito | Remove passageiros desembarcados de `up_queue`. |

### Função: `emit_down_boarding_events_at_current_floor(elevator_state: dict) -> Iterator[dict]`

| Item | Detalhe |
|---|---|
| Objetivo | Emitir eventos de embarque na descida para o andar atual. |
| Entradas | `elevator_state`. |
| Saída | Pode gerar `STOP_ENTER` e `AFTER_ENTER`. |
| Efeito | Move passageiros de `down_queue` para `down_riders`. |

### Função: `up_trip_events(elevator_state: dict) -> Iterator[dict]`

| Item | Detalhe |
|---|---|
| Objetivo | Gerar fluxo completo da viagem de subida. |
| Entradas | `elevator_state`. |
| Saída | Sequência de eventos de subida. |
| Fluxo | `DEPART_UP` -> `MOVE(UP)` -> eventos de parada e desembarque por andar. |

### Função: `down_service_events(elevator_state: dict, ground_floor: int) -> Iterator[dict]`

| Item | Detalhe |
|---|---|
| Objetivo | Gerar fluxo de busca de chamadas de descida e retorno ao térreo. |
| Entradas | `elevator_state`, `ground_floor`. |
| Saída | Sequência de eventos de subida vazia, embarque e descida. |
| Fluxo | `SEEK_DOWN_CALLS` -> `MOVE(UP_EMPTY)` -> paradas de embarque -> `MOVE(DOWN)` -> chegada ao térreo com passageiros. |

### Função: `idle_return_events(elevator_state: dict, ground_floor: int) -> Iterator[dict]`

| Item | Detalhe |
|---|---|
| Objetivo | Gerar retorno ocioso ao térreo. |
| Entradas | `elevator_state`, `ground_floor`. |
| Saída | Eventos de `MOVE(DOWN_EMPTY)` e `ARRIVE_GROUND_EMPTY`. |

## 5.8 `src/ui/__init__.py`

Este arquivo expõe a API pública de `ui` via `__all__`.

Exports:

- `apply_ui_for_event`
- `render_ground_status`
- `render_status`

## 5.9 `src/ui/ui_rich.py`

### Função: `render_status(current_floor: int, floor_label: str, right_table_data: list[dict], right_table_title: str, bottom_table_data: list[dict] | None = None, bottom_table_title: str | None = None, show_bottom_table: bool = False) -> None`

| Item | Detalhe |
|---|---|
| Objetivo | Renderizar o estado atual em colunas usando Rich. |
| Entradas | Dados do piso, tabela principal e tabela secundária opcional. |
| Saída | Nenhuma. |
| Efeitos colaterais | Limpa terminal (`console.clear`) e imprime tabelas (`console.print`). |

## 5.10 `src/ui/event_handlers.py`

### Funções de renderização (sem lógica de roteamento)

| Função | Objetivo principal |
|---|---|
| `render_ground_status` | Render da visão no térreo. |
| `render_up_move` | Render de movimento de subida com passageiros. |
| `render_up_stop_exit` | Render de parada com desembarque. |
| `render_down_move` | Render de movimento em contexto de descida/coleta. |
| `render_down_stop_enter` | Render de parada com embarque. |
| `render_idle_move` | Render de movimento vazio. |

### Funções handler (efeito de UI por tipo de evento)

| Função | Evento alvo | Ação |
|---|---|---|
| `handle_depart_up` | `DEPART_UP` | Mensagem de partida e `sleep`. |
| `handle_seek_down_calls` | `SEEK_DOWN_CALLS` | Mensagens de planejamento e `sleep`. |
| `handle_stop_exit` | `STOP_EXIT` | Render de parada, log de saída por passageiro, `sleep`. |
| `handle_stop_enter` | `STOP_ENTER` | Render de embarque e `sleep`. |
| `handle_arrive_ground_with_down_riders` | `ARRIVE_GROUND_WITH_DOWN_RIDERS` | Mensagem de chegada e `sleep`. |
| `handle_arrive_ground_empty` | `ARRIVE_GROUND_EMPTY` | Render de chegada vazia e `sleep`. |
| `handle_move` | `MOVE` | Resolve direção (`UP`, `UP_EMPTY`, `DOWN`, fallback) e renderiza. |
| `handle_after_exit` | `AFTER_EXIT` | Render do estado após desembarque. |
| `handle_after_enter` | `AFTER_ENTER` | Render do estado após embarque. |

### Função de roteamento principal

#### `apply_ui_for_event(event: dict, sleep_time: int) -> None`

| Item | Detalhe |
|---|---|
| Objetivo | Roteador central que mapeia `event["type"]` para o handler correto. |
| Entradas | `event`, `sleep_time`. |
| Saída | Nenhuma. |
| Observação | Implementa cadeia de `if` para todos os tipos de evento suportados. |

## 6. Fluxo ponta a ponta

1. `main.py` cria estado inicial com `create_initial_state`.
2. `populate_queues` atualiza `up_queue` e `down_queue`.
3. `up_trip_events` gera eventos de subida.
4. Se necessário, `down_service_events` busca passageiros e retorna ao térreo.
5. Se não houver passageiros, `idle_return_events` retorna vazio.
6. Cada evento é entregue a `apply_ui_for_event`.
7. A UI escolhe handler e usa `render_status` para exibir estado.

## 7. Dependências externas

- `faker`: geração de nomes de passageiros.
- `rich`: renderização visual no terminal.

## 8. Convenções técnicas adotadas

- Imports absolutos a partir de `src` (`queue_control`, `elevator_movement`, `ui`).
- Sem imports relativos (`from .`).
- Fluxo dirigido por eventos (domínio separado da UI).
- `__init__.py` com reexports explícitos para API pública dos pacotes.

