# Local Spark Cluster Setup

This project provides scripts to set up a local Apache Spark cluster using Docker on macOS.

## Prerequisites

- macOS
- Homebrew (will be installed if not present)
- Docker (will be installed if not present)

## Quick Start

1. Set up the entire Spark cluster with one command:
```bash
make setup
```

This will:
- Check if Docker is installed (install if needed)
- Pull the Spark Docker image
- Start a Spark master node
- Start a Spark worker node
- Open the Spark UI in your browser

## Available Commands

- `make check-docker`: Check if Docker is installed
- `make install-docker`: Install Docker using Homebrew
- `make pull-spark`: Download the Spark Docker image
- `make start-master`: Start the Spark master node
- `make start-worker`: Start a Spark worker node
- `make open-ui`: Open the Spark UI in your default browser
- `make pyspark`: Start a PySpark shell
- `make spark-shell`: Start a Scala Spark shell
- `make stop`: Stop all Spark containers
- `make clean`: Stop and remove all Spark containers

## Working with Spark (Short tutorials for Python and Scala)

### 1. Using PySpark Shell

Start the PySpark shell:

```bash
make pyspark
```

Example PySpark operations:
```python
python
sc.parallelize([1, 2, 3, 4, 5]).collect()
```
Create a simple RDD:
```python
rdd = sc.parallelize([1, 2, 3, 4, 5])
rdd.collect()
print(rdd.count())
```

Create a DataFrame:
```python
from pyspark.sql import Row
df = spark.createDataFrame([
Row(id=1, name="John"),
Row(id=2, name="Jane")
])
df.show()
```

SQL Operations:
```python
df.createOrReplaceTempView("people")
spark.sql("SELECT * FROM people").show()
```


### 2. Using Spark-Shell (Scala)

Start the Scala shell:
```bash
make spark-shell
```

Example Scala operations:
```scala
// Create a simple RDD
val rdd = sc.parallelize(List(1, 2, 3, 4, 5))
println(rdd.count())
// Create a DataFrame
case class Person(id: Int, name: String)
val df = spark.createDataFrame(Seq(
Person(1, "John"),
Person(2, "Jane")
))
df.show()
// SQL Operations
df.createOrReplaceTempView("people")
spark.sql("SELECT FROM people WHERE id > 1").show()
```


### 3. Working with External Data

To work with external data files:

1. Copy data into the container:
```bash
docker cp ./your_data.csv spark-master:/tmp/
```

2. Read the data in PySpark:

CSV:
```python
df = spark.read.csv("/tmp/your_data.csv", header=True)
```

JSON:
```python
df = spark.read.json("/tmp/your_data.json")
```

Parquet:
```python
df = spark.read.parquet("/tmp/your_data.parquet")
```


### 4. Common Spark Operations

Here are some common operations you can try:

#### Data Transformation
```python
df = spark.read.csv("/tmp/your_data.csv", header=True)
```
#### Filter
```python
filtered_df = df.filter(df.column > 100)
```
#### Group By and Aggregate
```python
summary_df = df.groupBy("category").agg({"value": "sum"})
```
#### Join
```python
df2 = spark.read.csv("/tmp/other_data.csv", header=True)
joined_df = df.join(df2, "common_column")
```
#### Window Functions
```python
from pyspark.sql import Window
from pyspark.sql.functions import row_number
window_spec = Window.partitionBy("category").orderBy("value")
df_with_rank = df.withColumn("rank", row_number().over(window_spec))
```


## Monitoring and Management

### Spark UI
Access the Spark UI at http://localhost:8080 to monitor:
- Running applications
- Job progress
- Executor status
- Storage usage
- Environment settings

### Container Management

Check container status:
```bash
docker ps -a
```

View container logs:
```bash
docker logs -f spark-master
docker logs -f spark-worker
```


## Troubleshooting

1. If containers fail to start:
```bash
make clean
make setup
```

2. If PySpark shell doesn't connect:
- Ensure all containers are running
- Check container logs for errors
- Try restarting the containers

3. Common Issues:
- Port conflicts: Change ports in Makefile if 8090 or 7077 are in use
- Memory issues: Adjust Docker Desktop memory allocation
- Connection refused: Ensure Docker Desktop is running

## Best Practices

1. Always clean up when done:
```bash
make clean
```

2. Monitor resource usage in Docker Desktop
3. Use `df.cache()` for frequently accessed DataFrames
4. Write intermediate results for long computations
5. Use `.explain()` to understand query plans

## Next Steps

- Explore Spark ML library for machine learning
- Try Spark Streaming for real-time data processing
- Experiment with Spark GraphX for graph processing
- Practice writing and deploying Spark applications

