## Purpose

This file defines repository-level operating rules for code agents working on Data Engineering projects.

The agent must prioritize practical implementation, local reproducibility, maintainable code, and lakehouse conventions.

---
## 1. Default Stack

Assume the project uses the following stack unless the repository says otherwise:

- Python
- PySpark
- Apache Spark
- Hadoop
- HDFS
- YARN
- Hive
- Hive Metastore
- Apache Airflow
- Docker
- Docker Compose
- PostgreSQL
- Apache Hudi
- Trino
- MinIO
- dbt

Do not assume Apache Iceberg as part of the default stack.

Cloud platforms, Kubernetes, Terraform, Databricks, Delta Lake, NoSQL, and Kafka may be discussed only when explicitly requested or when clearly necessary for comparison.

---
## 2. Default Environment

Assume the default runtime environment is:

```text
Local lakehouse platform
Docker Compose
Spark
PySpark
Hadoop
HDFS
YARN
Hive
Hive Metastore
PostgreSQL
Apache Hudi
Airflow
Trino
MinIO
```

Prioritize local, reproducible, on-premise solutions.

Do not suggest cloud-first architectures unless explicitly requested.

Do not suggest Kubernetes unless the complexity is justified or explicitly requested.

---

## 3. Lakehouse Architecture

Use the following layering model as the default:

```text
Raw -> Bronze -> Silver -> Gold
```

### Raw

Stores files as close as possible to the original source.

Responsibilities:

- Preserve original input.
- Avoid business transformations.
- Support reprocessing.
- Keep source-level traceability.

### Bronze

Stores ingested data in a queryable format.

Responsibilities:

- Apply technical ingestion structure.
- Add ingestion metadata.
- Normalize file reading issues.
- Preserve source semantics.
- Avoid heavy business rules.

### Silver

Stores cleaned, standardized, deduplicated, and validated data.

Responsibilities:

- Enforce schema.
- Apply type conversions.
- Standardize column names.
- Remove duplicates.
- Apply validation rules.
- Apply historical logic when required.
- Store trustworthy analytical entities.

### Gold

Stores business-ready data.

Responsibilities:

- Create facts and dimensions.
- Aggregate metrics.
- Serve dashboards, reports, APIs, and SQL users.
- Optimize for consumption and business logic.

---

## 4. Table Format

Use Apache Hudi as the default lakehouse table format.

Prefer Hudi when the pipeline requires:

- Upserts
- Slowly Changing Dimensions
- Incremental processing
- CDC-style ingestion
- Record-level updates
- Historical versions
- Current-state tables

Do not recommend plain Parquet when the use case requires updates, merges, or historical versioning behavior.

Do not recommend Iceberg as the default table format.

---

## 5. Catalog and Table Access

Always prefer catalog-based table access through Hive Metastore.

Use the `database.table` convention.

Do not suggest manual path-based access when Hive Metastore is available.

Avoid:

```python
customers_df = spark.read.format("hudi").load("/some/manual/path")
```

Prefer:

```python
customers_df = spark.table("silver.customers")
```

When creating or writing tables, ensure they are registered and queryable through the metastore.

---

## 6. Code Standards

When generating Python or PySpark code:

- Use simple code.
- Use explicit typing.
- Avoid excessive abstraction.
- Avoid clever or implicit syntax.
- Avoid ternary expressions.
- Prefer readable control flow.
- Use Google-style docstrings for all functions and classes.
- Keep examples focused on the requested task.
- Do not add unnecessary comments.
- Use Python or PySpark for examples.
- Avoid SQL examples unless explicitly requested.

Apply Clean Code principles:

- Use meaningful names.
- Keep functions small and focused.
- Keep modules organized by responsibility.
- Prefer explicit behavior over hidden side effects.
- Avoid deeply nested logic.
- Avoid duplicated business rules.
- Make dependencies visible.
- Separate I/O, transformation, validation, and write logic.

Apply DRY carefully:

- Extract repeated logic only when the duplication represents the same concept.
- Do not create premature abstractions.
- Do not hide simple transformations behind unnecessary layers.
- Prefer a small amount of duplication over a bad abstraction when responsibilities are unstable.

Apply Object Calisthenics when useful:

- One level of indentation per method when practical.
- Do not use `else` when early return is clearer.
- Wrap related primitives in meaningful types or structures when they represent domain concepts.
- Use first-class collections when collection behavior has business meaning.
- Keep one responsibility per class.
- Keep classes small.
- Avoid exposing internal mutable state.
- Prefer intention-revealing methods.
- Avoid long chains of calls.
- Avoid unnecessary getters and setters.

For Data Engineering code:

- Separate read, transform, validate, and write steps.
- Keep Spark session creation outside transformation functions.
- Transformation functions must receive and return DataFrames.
- Avoid actions such as `count()`, `collect()`, and `show()` inside reusable transformation functions.
- Avoid hardcoded database and table names inside generic functions.
- Avoid hardcoded paths when Hive Metastore is available.
- Use configuration objects or explicit parameters for table names and Hudi options.

Example:

```python
from pyspark.sql import DataFrame
from pyspark.sql import functions as F


def add_ingestion_metadata(df: DataFrame, source_name: str) -> DataFrame:
    """Add standard ingestion metadata columns.

    Args:
        df: Input Spark DataFrame.
        source_name: Name of the source system.

    Returns:
        Spark DataFrame with ingestion metadata columns.
    """
    result_df = df.withColumn("_source", F.lit(source_name))
    result_df = result_df.withColumn("_ingested_at", F.current_timestamp())

    return result_df
```

---

## 7. PySpark Pipeline Structure

When creating pipelines, prefer this structure:

```text
read source
standardize schema
add metadata
validate records
deduplicate
apply business rules
write to Hudi table
register or sync with Hive Metastore
validate output
```

For each pipeline, clarify:

- Source format
- Target layer
- Target database and table
- Natural key
- Partition strategy
- Write mode
- Hudi operation
- Precombine field
- Record key
- Hive sync behavior
- Validation approach

Examples must use Python or PySpark unless explicitly requested otherwise.

---

## 8. Hudi Write Defaults

When using Hudi, explicitly define:

- Table name
- Record key
- Precombine field
- Partition field when applicable
- Table type
- Operation
- Hive sync settings
- Target database
- Target table

Choose the Hudi table type according to the workload:

```text
COPY_ON_WRITE: simpler reads, fewer moving parts
MERGE_ON_READ: better for frequent updates, requires compaction strategy
```

Do not choose `MERGE_ON_READ` without explaining compaction and read implications.

Example:

```python
hudi_options = {
    "hoodie.table.name": "customers",
    "hoodie.datasource.write.table.type": "COPY_ON_WRITE",
    "hoodie.datasource.write.operation": "upsert",
    "hoodie.datasource.write.recordkey.field": "customer_id",
    "hoodie.datasource.write.precombine.field": "updated_at",
    "hoodie.datasource.hive_sync.enable": "true",
    "hoodie.datasource.hive_sync.mode": "hms",
    "hoodie.datasource.hive_sync.database": "silver",
    "hoodie.datasource.hive_sync.table": "customers",
}
```

---

## 9. Security and Reliability

For local and on-premise environments:

- Do not expose internal services directly to the public internet.
- Avoid committing secrets.
- Use environment variables carefully.
- Keep metastore database backups.
- Pin service versions.
- Use persistent Docker volumes for metadata and warehouse storage.
- Add health checks where useful.
- Use SSH keys instead of passwords for server access.
- Keep service ports intentional and documented.

---

## 10. Expected Deliverables

This repository may require creating or modifying:

- PySpark pipelines
- Hudi ingestion jobs
- Hudi upsert jobs
- Historical data pipelines
- Python transformation modules
- PySpark validation modules
- Airflow DAGs when explicitly requested
- Docker Compose data platforms when explicitly requested
- Hive Metastore table integration
- Lakehouse architecture documents
- C4 Model diagrams with Mermaid
- Troubleshooting guides
- README files for Data Engineering projects
- AGENTS.md files for Data Engineering repositories
