# Iceberg Quickstart

A ready-to-use local environment to experiment with [Apache Iceberg](https://iceberg.apache.org/), using Spark, an Iceberg REST catalog, and S3-compatible object storage (MinIO).

Everything runs in Docker, and interaction happens through the provided Jupyter notebooks.

## Architecture

The `docker-compose.yml` starts four services on a shared network:

- **spark-iceberg** — `tabulario/spark-iceberg` image, exposes Jupyter (`8888`), Spark UI (`8080`), and Thrift server (`10000/10001`).
- **rest** — Iceberg REST catalog (`apache/iceberg-rest-fixture`) on port `8181`, backed by the S3 `warehouse` bucket.
- **minio** — S3-compatible object storage (console on `9001`, API on `9000`).
- **mc** — MinIO client that automatically creates the `warehouse` bucket on startup.

The `notebooks/` folder is mounted into the Spark container and contains example notebooks (table creation, bucket partitioning).

## Requirements

- Docker and Docker Compose

## Usage

1. **Configure environment variables**

   A `.env` file is already provided with default values (`admin` / `password`). You can copy it from `.env.example` if needed:

   ```sh
   cp .env.example .env
   ```

2. **Start the stack**

   ```sh
   docker compose up -d
   ```

3. **Open Jupyter**

   Go to [http://localhost:8888](http://localhost:8888). The example notebooks live under `notebooks/`:

   - `Create a table.ipynb` — create a namespace, an Iceberg table, insert and query data.
   - `Bucket Partitioning.ipynb` — demonstrate bucket partitioning with a TPC-H inspired dataset.

4. **MinIO console** (optional)

   [http://localhost:9001](http://localhost:9001) — credentials are defined in `.env`. Useful to inspect the Parquet files and Iceberg metadata written to the `warehouse` bucket.

5. **Stop the stack**

   ```sh
   docker compose down
   ```

   MinIO data persists in the `minio_data` Docker volume. To start fresh:

   ```sh
   docker compose down -v
   ```

## Quick example

In a Jupyter notebook:

```python
from pyspark.sql import SparkSession
spark = SparkSession.builder.appName("Jupyter").getOrCreate()
```

```sql
%%sql
CREATE NAMESPACE IF NOT EXISTS demo.example;

CREATE TABLE demo.example.nation (
    nationkey BIGINT,
    name      STRING
) USING iceberg;

INSERT INTO demo.example.nation VALUES (1, 'FRANCE'), (2, 'GERMANY');

SELECT * FROM demo.example.nation;
```

The `demo` catalog is preconfigured in the Spark image and points to the REST service, which itself is backed by MinIO.
