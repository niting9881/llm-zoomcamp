
# 🚀 Getting Started with dlt and Cognee: A Beginner's Guide

Welcome to your study guide for **dlt** and **Cognee**! This guide will help you understand these powerful tools, their core concepts, and how they work together, even if you're just starting out.

---

## 📚 dlt: Data Load Tool

**dlt** (Data Load Tool) is an open-source Python library that helps load data from various messy sources into structured, live datasets.

### What dlt Does:
- Extracts data from REST APIs, SQL databases, cloud storage, or Python objects.
- Infers schemas, normalizes data, handles nested structures.
- Supports destinations like DuckDB, PostgreSQL, BigQuery, Snowflake, Oracle, etc.
- Handles incremental loading, schema evolution, and data contracts.
- Runs anywhere Python runs (e.g., Airflow, serverless functions).

### Getting Started with dlt:
1. **Python Version:** Use Python 3.9–3.13
2. **Virtual Environment:**
   ```bash
   uv venv --python 3.10
   source .venv/bin/activate  # macOS/Linux
   .\.venv\Scripts\activate  # Windows
   ```
3. **Install dlt:**
   ```bash
   uv pip install -U dlt
   uv pip install "dlt[duckdb]"  # Optional for DuckDB
   ```

### Basic Usage with dlt:
- **Define a pipeline:**
  ```python
  pipeline = dlt.pipeline(pipeline_name="example", destination="duckdb", dataset_name="my_dataset")
  ```
- **Define a source:** REST API, SQL, S3, local files, or Python generators
- **Run the pipeline:**
  ```python
  pipeline.run(source)
  ```

### Loading Strategies:
- `append` (default): Adds new data.
- `replace`: Deletes existing data.
- `merge`: Upserts using primary key.
- Supports **incremental loading** (cursor-based).

---

## 🧠 Cognee: Data to Memory

Cognee creates an intelligent **knowledge graph** from your data to improve LLM reasoning.

### Why Cognee is Different:
- Traditional RAG = vector databases only
- **Cognee = Graph RAG**: builds a semantic **knowledge graph** for context and relationship reasoning

### How Cognee Works:
- **Tasks:** Atomic operations like chunking, NER
- **Pipelines:** Chained tasks like `.add()` and `.cognify()`
- **Dual Storage:** Graph DB (Neo4j/Kuzu) + Vector DB (LanceDB)

### Core Concepts:
- **Node Sets:** Tags for organizing nodes (e.g., "chapter_1")
- **Ontologies:** Machine-readable schemas to guide graph construction

### Getting Started with Cognee:
1. **Install SDK**
   ```bash
   uv venv
   source .venv/bin/activate
   uv pip install cognee
   ```
2. **Add LLM API Key**
   ```env
   LLM_API_KEY="your_openai_key"
   ```
   or use Ollama/local LLMs
3. **Choose Database:** SQLite (default), Neo4j, PGVector, Kuzu

### Basic Usage:
```python
await cognee.add(text)       # Ingest content
await cognee.cognify()       # Build knowledge graph
await cognee.search(query_text="your question")
```

---

## 🤝 dlt + Cognee: A Powerful Combination

**Example:** Use `dlt` to load NYC Taxi data into DuckDB, then have `Cognee` ingest it and create searchable graph memory.

### Workflow:
1. `dlt` loads relational or JSON data
2. `Cognee` adds node sets for partitions (e.g., "day_1", "day_2")
3. `Cognee` builds semantic graph + embeddings

---

## ❓ Frequently Asked Questions (FAQ)

**Q: Can Cognee help with scattered information like legal docs?**  
> Yes! Cognee uses a graph to connect related info spread across pages.

**Q: What should I ask Cognee?**  
> Ask about relationships or context-based questions. Ontologies make it even smarter.

**Q: How much does Cognee cost?**  
> Costs can be < $0.01 for some queries depending on LLMs used.

**Q: AI Engineer vs Data Engineer?**  
> Data Engineer builds pipelines. AI Engineer does this + builds LLM pipelines (e.g., RAG).

**Q: Can dlt handle billions of records?**  
> Yes! Supports large-scale ingestion, optimized for performance.

---

## 🔗 Resources
- [Cognee Docs](https://docs.cognee.ai/))
- [dlt Docs](https://dlthub.com/docs/intro)

