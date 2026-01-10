# IDC Databricks 14 Days AI Challenge


## Day 1 :

- Today wasn’t just about creating an account or running Spark commands.
- It was the day I truly understood why Databricks exists. At the start, I was honestly overwhelmed.

### I had zero idea about :
- What a workspace, catalog, or volume actually means
- How data is stored and accessed inside Databricks
- What serverless compute is and why it matters
- What Spark really does behind the scenes
- Even loading data felt intimidating at first
- But step by step, things started clicking.

### 🔍 What I learned today :
- Workspace → where notebooks, workflows, and experiments live
- Catalog → the governed layer that organizes data and metadata
- Volume → a managed storage location for large datasets
- Serverless compute → Databricks manages infra so you focus on analysis
- Spark → my first time using it, and I finally understood why it’s built for scale

### 🛠️ What I worked on :
- Created a Databricks Community Edition account
- Explored Workspace, Compute, and Data Explorer
- Created my first Databricks notebook
- Ran basic PySpark commands

### 💡 Big “aha” moment :
- I was able to load a massive Kaggle dataset directly into a Databricks volume using just 2–3 lines of code — something I never imagined was possible without complex setup.
- Using the Kaggle API inside Databricks to download data straight into a volume felt like magic 
- This completely changed how I think about data ingestion.

### ⚠️ What challenged me the most :
 - Understanding how compute, storage, and workspace are connected.
 - Once this clicked, Databricks started making sense as a platform, not just a tool.

------------------------------
-----------------------------

## Day 2 – Apache Spark Fundamentals⚡

### Read the files from Volume
```python
oct_events = spark.read.csv(
    "/Volumes/workspace/ecommerce/ecommerce_data/2019-Oct.csv",
    header=True,
    inferSchema=True
)

nov_events = spark.read.csv(
    "/Volumes/workspace/ecommerce/ecommerce_data/2019-Nov.csv",
    header=True,
    inferSchema=True
)
```

### Check the schema of Dataframe and total values
```python
print(f"October 2019 - Total Events: {oct_events.count():,}")
oct_events.printSchema()

print(f"November 2019 - Total Events: {nov_events.count():,}")
nov_events.printSchema()
```

### Print sample rows
```python
display(oct_events.limit(10))
display(nov_events.limit(10))
```

### Use col() and select() function to fetch particular column and gets its count through distinct() 
```python
from pyspark.sql.functions import col
brand_count = oct_events.select(col("brand")).distinct().count()
brand_count
```

```python
sample_data = oct_events.select(col("brand"),col("price"),col("event_type"))
display(sample_data.limit(30))
```

### Creates new column by withColumn()
```python
high_price_flag = oct_events.withColumn("high_price", col("price") > 1000)
display(high_price_flag.limit(30))
```

### filtering rows by filter() 
```python
high_price_products = high_price_flag.filter(col("high_price") == True)
display(high_price_products.limit(30))
```
### Renamed the column by withColumnRenamed()
```python
high_price_flag_renamed = high_price_flag.withColumnRenamed("high_price", "high_price_renamed")
display(high_price_flag_renamed.limit(30))
```

### Droping the created column
```python
high_price_flag_renamed = high_price_flag_renamed.drop("high_price_renamed")
display(high_price_flag_renamed.limit(10))
```

```python
filter_data = oct_events.filter((col("brand") == "lenovo"))
display(filter_data.limit(10))
```


```python
filter_data = oct_events.filter((col("brand") == "lenovo") & (col("price") > 1000))
display(filter_data.limit(10))
```

```python
unique_events = oct_events.select(col("event_type")).distinct()
display(unique_events.limit(10))
```

### Aggregating the data using groupBy()
```python
from pyspark.sql.functions import count, sum, round, col

brand_metrics = (
    oct_events
    .filter((col("event_type") == "purchase") & col("brand").isNotNull())
    .groupBy("brand")
    .agg(
        sum("price").alias("total_sales"),
        count("*").alias("total_orders")
    )
    .withColumn(
        "total_sales_millions",
        round(col("total_sales") / 1_000_000, 2)
    )
    .orderBy("total_sales_millions", ascending=False)
)

display(brand_metrics.limit(10))

```

### Magical Keywords %sql,%fs 

```sql
%sql
SELECT * FROM oct_events
LIMIT 10;
```
```sql
%sql
SELECT COUNT(DISTINCT brand) AS total_brands
FROM oct_events;
```
```sql
%sql
SELECT
  brand,
  ROUND(SUM(price) / 1000000, 2) AS sales_mn,
  COUNT(*) AS total_orders
FROM oct_events
WHERE event_type = 'purchase' and brand IS NOT NULL
GROUP BY brand
ORDER BY sales_mn DESC
LIMIT 10;

```

```sql
%sql
SELECT
  event_type,
  COUNT(*) AS events
FROM oct_events
GROUP BY event_type;
```

### List of Volumes
```bash
%fs
ls /Volumes/workspace/ecommerce/ecommerce_data/
```

### Preview CSV (first few lines)
```bash
%fs
head /Volumes/workspace/ecommerce/ecommerce_data/2019-Oct.csv
```
### Count files
```bash
files = dbutils.fs.ls("/Volumes/workspace/ecommerce/ecommerce_data")
len(files)
```
