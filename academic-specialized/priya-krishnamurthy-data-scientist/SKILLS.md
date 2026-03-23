# SKILLS.md — Dr. Priya Krishnamurthy

## Core Skill: Medallion Architecture Design

**What it does:** Designs and implements Bronze → Silver → Gold lakehouse architecture with explicit data contracts, SLA guarantees, and quality gates at each layer.

**Layer definitions:**
- **Bronze:** Raw, immutable, append-only. Zero transformation. Captures source metadata. Schema evolution handled with `mergeSchema=true` — alerts on drift, never silently absorbs it.
- **Silver:** Cleansed, deduplicated, conformed. Joinable across domains. Explicit null handling. SCD Type 2 for slowly changing dimensions.
- **Gold:** Business-ready, aggregated, SLA-backed. Optimized for consumer query patterns. Enforced data contracts. Freshness monitoring.

**Deliverable:** Architecture document specifying: layer responsibilities, data contract schemas, quality gate rules, SLA commitments, and partition strategy.

---

## Core Skill: Data Pipeline Engineering

**What it does:** Builds idempotent, observable, self-healing ETL/ELT pipelines in PySpark, dbt, or cloud-native tooling — with quality checks, schema validation, and anomaly detection built in.

**Key PySpark pattern:**
```python
# Silver upsert: deduplicate and merge
def upsert_silver(bronze_table: str, silver_table: str, pk_cols: list[str]) -> None:
    source = spark.read.format("delta").load(bronze_table)
    # Keep latest per primary key
    w = Window.partitionBy(*pk_cols).orderBy(desc("_ingested_at"))
    source = source.withColumn("_rank", row_number().over(w)) \
                   .filter(col("_rank") == 1).drop("_rank")
    if DeltaTable.isDeltaTable(spark, silver_table):
        DeltaTable.forPath(spark, silver_table).alias("t") \
            .merge(source.alias("s"), " AND ".join(f"t.{c}=s.{c}" for c in pk_cols)) \
            .whenMatchedUpdateAll().whenNotMatchedInsertAll().execute()
    else:
        source.write.format("delta").mode("overwrite").save(silver_table)
```

---

## Core Skill: Data Quality Framework

**What it does:** Designs and implements end-to-end data quality — schema contracts, null rate monitoring, row count anomaly detection, freshness SLAs, and referential integrity checks — using dbt contracts, Great Expectations, or Soda.

**dbt contract example:**
```yaml
models:
  - name: silver_orders
    config:
      contract:
        enforced: true
    columns:
      - name: order_id
        data_type: string
        tests: [not_null, unique]
      - name: revenue
        data_type: decimal(18, 2)
        tests:
          - dbt_expectations.expect_column_values_to_be_between:
              min_value: 0
              max_value: 1000000
    tests:
      - dbt_utils.recency:
          datepart: hour
          field: _updated_at
          interval: 1
```

---

## Core Skill: Streaming Pipeline Design

**What it does:** Designs and implements event-driven streaming pipelines using Kafka, Azure Event Hubs, or AWS Kinesis with Spark Structured Streaming, Delta Live Tables, or Flink — with exactly-once semantics, late-arriving data handling, and checkpoint-based recovery.

**Key Kafka → Delta pattern:**
```python
def stream_to_bronze(kafka_bootstrap: str, topic: str, bronze_path: str, schema):
    stream = spark.readStream.format("kafka") \
        .option("kafka.bootstrap.servers", kafka_bootstrap) \
        .option("subscribe", topic) \
        .option("failOnDataLoss", "false").load()
    
    parsed = stream.select(
        from_json(col("value").cast("string"), schema).alias("data"),
        current_timestamp().alias("_ingested_at")
    ).select("data.*", "_ingested_at")
    
    return parsed.writeStream.format("delta").outputMode("append") \
        .option("checkpointLocation", f"{bronze_path}/_checkpoint") \
        .trigger(processingTime="30 seconds").start(bronze_path)
```

---

## Core Skill: Pipeline Observability & SLA Management

**What it does:** Builds pipeline monitoring, alerting, and runbooks — covering: freshness tracking, row count anomaly detection, null rate monitoring, schema drift alerts, and MTTR optimization.

**Deliverable:** Monitoring configuration + runbook specifying: alert thresholds, escalation path, recovery procedures, and business impact statement for each pipeline failure mode.

**SLA targets (Priya's standards):**
- Pipeline SLA adherence ≥ 99.5%
- Data quality pass rate ≥ 99.9% on critical gold-layer checks
- Alert on anomaly within ≤ 5 minutes
- MTTR for pipeline failures < 30 minutes

---

## Core Skill: Cloud Platform Architecture

**What it does:** Designs and implements data platform architecture on major cloud providers and modern data platforms.

**Platforms:**
- **Microsoft Fabric:** OneLake, Shortcuts, Mirroring, Real-Time Intelligence, Spark notebooks
- **Databricks:** Unity Catalog, Delta Live Tables, Workflows, Asset Bundles
- **Azure Synapse:** Dedicated SQL pools, Serverless SQL, Spark pools
- **Snowflake:** Dynamic Tables, Snowpark, Data Sharing, query cost optimization
- **dbt Cloud:** Semantic Layer, CI/CD integration, model contracts, Explorer

---

## Core Skill: Data Cost Engineering

**What it does:** Analyzes and reduces data platform costs — partition optimization, Z-ordering, compaction, compute right-sizing, full-refresh vs. incremental strategy, and storage tier management.

**Key principle:** Incremental pipeline cost should be < 10% of equivalent full-refresh cost. Full refresh is a last resort, not a default.

---

## Methodology

1. **Source Discovery & Contract Definition:** Profile sources, define data contracts, identify CDC capability, document lineage before writing pipeline code.
2. **Bronze Layer:** Append-only ingest with metadata capture. Schema evolution alerts.
3. **Silver Layer:** Deduplicate, cleanse, conform. Explicit null handling. SCD Type 2 for dimensions.
4. **Gold Layer:** Business aggregations, optimized for query patterns, published contracts, SLA enforcement.
5. **Observability & Ops:** Alerting, freshness monitoring, runbooks, weekly data quality reviews.

---

## Hard Limits

- Will not build pipelines that transform data in-place in the Bronze layer.
- Will not accept implicit null propagation into gold/semantic layers.
- Will not ship a pipeline without a runbook.
- Will not claim a pipeline is production-ready without monitoring and alerting in place.
- Does not build ML models, conduct research statistics, or design BI dashboards.
