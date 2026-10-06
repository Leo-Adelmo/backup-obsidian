```mermaid
flowchart LR

    %% =====================================================
    %% ACTORS
    %% =====================================================

    ENGINEER(("Data Engineer<br/>[Person]<br/>Maintains and monitors data pipelines"))

    USER(("End User<br/>[Person]<br/>Searches veterinary establishments information"))

    %% =====================================================
    %% EXTERNAL SYSTEMS
    %% =====================================================

    GOOGLE["Google Places API<br/>[External System]<br/>Provides veterinary establishments data"]

    %% =====================================================
    %% SYSTEM BOUNDARY
    %% =====================================================

    subgraph VET_PLATFORM_BOUNDARY["Veterinary Search System"]

        direction LR

        %% =================================================
        %% ORCHESTRATION
        %% =================================================

        AIRFLOW["Pipeline Orchestrator<br/>[Container: Airflow]<br/>Schedules and orchestrates ingestion and processing pipelines"]

        %% =================================================
        %% INGESTION
        %% =================================================

        INGESTION["Data Ingestion<br/>[Container: Python]<br/>Collects veterinary establishments data from Google Places API"]

        %% =================================================
        %% STORAGE
        %% =================================================

        STORAGE[("Data Lake<br/>[Container: MinIO]<br/>Stores raw and processed datasets")]

        %% =================================================
        %% PROCESSING
        %% =================================================

        PROCESSING["Data Processing<br/>[Container: Spark + Hudi]<br/>Transforms and curates analytical datasets"]

        %% =================================================
        %% METADATA
        %% =================================================

        CATALOG[("Data Catalog<br/>[Container: Hive Metastore]<br/>Manages schemas and metadata from the data lake")]

    end

    %% =====================================================
    %% FLOWS
    %% =====================================================

    USER -->|Searches veterinary data| INGESTION

    ENGINEER -->|Monitors DAGs and pipelines| AIRFLOW

    GOOGLE -->|Provides establishments data| INGESTION

    AIRFLOW -->|Orchestrates ingestion| INGESTION

    AIRFLOW -->|Orchestrates processing| PROCESSING

    INGESTION -->|Stores raw datasets| STORAGE

    PROCESSING -->|Reads Bronze datasets| STORAGE

    PROCESSING -->|Writes Silver and Gold datasets| STORAGE

    PROCESSING -->|Updates table metadata| CATALOG

    %% =====================================================
    %% COLORS
    %% =====================================================

    %% Actors
    classDef actor fill:#0F172A,stroke:#1E3A8A,color:#FFFFFF,stroke-width:2px;

    %% Internal Containers
    classDef internal fill:#2563EB,stroke:#1D4ED8,color:#FFFFFF,stroke-width:2px;

    %% External Systems
    classDef external fill:#15803D,stroke:#166534,color:#FFFFFF,stroke-width:2px;

    %% Storage
    classDef storage fill:#065F46,stroke:#047857,color:#FFFFFF,stroke-width:2px;

    %% Orchestration
    classDef orchestration fill:#B45309,stroke:#D97706,color:#FFFFFF,stroke-width:2px;

    %% Metadata
    classDef metadata fill:#0F766E,stroke:#14B8A6,color:#FFFFFF,stroke-width:2px;

    %% Boundary
    style VET_PLATFORM_BOUNDARY fill:none,stroke:#2563EB,stroke-width:2px,stroke-dasharray: 5 5

    %% =====================================================
    %% CLASS ASSIGNMENT
    %% =====================================================

    class ENGINEER,USER actor;
    class GOOGLE external;
    class INGESTION,PROCESSING internal;
    class STORAGE storage;
    class AIRFLOW orchestration;
    class CATALOG metadata;

```

