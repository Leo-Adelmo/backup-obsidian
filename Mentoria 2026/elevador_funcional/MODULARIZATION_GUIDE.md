# Elevator Modularization Guide

This document describes the current modular architecture extracted from `elevador_teste_4.py`.

## Current Directory Structure

```text
elevador/
  src/
    main.py
    queue_control/
      __init__.py
      passenger_generation.py
      queue_operations.py
    elevator_movement/
      __init__.py
      movement_flow.py
      movement_events.py
    ui/
      __init__.py
      event_handlers.py
      ui_rich.py
```

## Why `src/elevator` Was Removed

The package wrapper `src/elevator` was intentionally removed so the main modules now live directly under `src/`:

- `queue_control`
- `elevator_movement`
- `ui`
- `main.py`

This keeps the structure flatter and makes imports shorter.

## Module Responsibilities

### `queue_control`

- `passenger_generation.py`
  - Creates random passenger events.
  - Generates up/down passengers.
  - Populates queues through `populate_queues`.
- `queue_operations.py`
  - Queue selection, limits, and queue mutations.
  - Boarding/exiting filters.
  - Snapshot and floor aggregation helpers.

### `elevator_movement`

- `movement_events.py`
  - Builds event payloads (`emit_*`).
  - Has no UI rendering responsibility.
- `movement_flow.py`
  - Implements the movement flow/state transitions.
  - Emits events for up trip, down service, and idle return.

### `ui`

- `event_handlers.py`
  - Event dispatcher and per-event handlers.
  - Console messages and pacing (`sleep`).
- `ui_rich.py`
  - Rich table rendering (`render_status`).

### `main.py`

- Coordinates the full simulation loop.
- Connects queue generation, movement event production, and UI handling.

## Import Strategy (No Relative Imports)

All imports are absolute and start from top-level modules in `src/`.

Examples:

- `from queue_control import populate_queues`
- `from elevator_movement import up_trip_events`
- `from ui import apply_ui_for_event`

No file uses relative imports such as `from .module import ...`.

## `__init__.py` Organization

Each package exposes its public API through `__init__.py`:

- `queue_control/__init__.py`
- `elevator_movement/__init__.py`
- `ui/__init__.py`

This allows `main.py` to import stable package boundaries instead of internal file paths.

## Docstring Standard

All modular files were updated with English docstrings using the Google style:

- module-level summaries
- `Args:` and `Returns:` sections for standard functions
- `Yields:` sections for generator-based movement flows

## Running the Modular Version

From project root:

```bash
source venv/bin/activate
pip install faker rich
PYTHONPATH=src python -m main
```

This makes `src/` available for imports and runs the modular entrypoint in `src/main.py`.

## Legacy Files Kept Intact

Original files were preserved in project root:

- `elevador_teste_4.py`
- `ui_rich.py`
