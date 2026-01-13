# IDC Databricks 14 Days AI Challenge


## Day 1 - Platform Setup & First Steps✅

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

### Install Dependencies
```python
!pip install kaggle
```

### Configure Kaggle Credentials
```python
import os

os.environ["KAGGLE_USERNAME"] = "user_name"
os.environ["KAGGLE_KEY"] = "kaggle api key"

print("Kaggle credentials configured!")
```

### Create Database Schema
```python
spark.sql("""
CREATE SCHEMA IF NOT EXISTS workspace.ecommerce
""")
```

### Create Volume for Data Storage
```python
spark.sql("""
CREATE VOLUME IF NOT EXISTS workspace.ecommerce.ecommerce_data
""")
```

### Download Dataset from Kaggle
```bash
%sh
cd /Volumes/workspace/ecommerce/ecommerce_data
kaggle datasets download -d mkechinov/ecommerce-behavior-data-from-multi-category-store
```

### Extract Downloaded Dataset
```bash
%sh
cd /Volumes/workspace/ecommerce/ecommerce_data
unzip -o ecommerce-behavior-data-from-multi-category-store.zip
ls -lh
```

### Clean Up Zip File
```bash
%sh
cd /Volumes/workspace/ecommerce/ecommerce_data
rm -f ecommerce-behavior-data-from-multi-category-store.zip
ls -lh
```

### Restart Python Environment
```python
%restart_python
```

### Load Oct 2019 Data
```python
oct_events = spark.read.csv(
    "/Volumes/workspace/ecommerce/ecommerce_data/2019-Oct.csv",
    header=True,
    inferSchema=True
)
```

### Load Nov 2019 Data
```python
nov_events = spark.read.csv(
    "/Volumes/workspace/ecommerce/ecommerce_data/2019-Nov.csv",
    header=True,
    inferSchema=True
)
```

### Quick View & Quality Check of Loaded Data in Volume
```python
print(f"October 2019 - Total Events: {oct_events.count():,}")
oct_events.printSchema()

oct_events.show(5, truncate=False)

print(f"November 2019 - Total Events: {nov_events.count():,}")
nov_events.printSchema()

nov_events.show(5, truncate=False)
```

------------------------------
-----------------------------

## Day 2 – Apache Spark Fundamentals✅

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

-------------------------------
--------------------------------

## Day 3 – PySpark Transformations Deep Dive ✅
#### Pandas vs Spark 
#### Pandas :
 - Works on a single machine
 - Loads data entirely into memory
 - Best for small to medium datasets
 - Executes operations immediately
 - Limited by RAM & CPU

#### Apache Spark :
 - Works in a distributed environment
 - Processes data across multiple machines
 - Handles millions to billions of rows
 - Uses lazy evaluation (executes only when needed)
 - Optimized automatically via DAG & Catalyst
 - Built for production-scale analytics
### Sample Dataset for practise

```python
# 1. Define the raw data as a list of tuples
Salesdata = [
    ("Prod001", 10, 300, "Virginia"),
    ("Prod002", 20, 500, "Virginia"),
    ("Prod003", 30, 460, "Virginia"),
    ("Prod023", 30, 460, "Virginia"),
    ("Prod004", 40, 987, "Virginia"),
    ("Prod005", 40, 987, "Virginia"),
    ("Prod001", 10, 1300, "Georgia"),
    ("Prod002", 20, 550, "Georgia"),
    ("Prod003", 30, 480, "Georgia"),
    ("Prod004", 40, 240, "Georgia"),
    ("Prod001", 10, 1100, "New York"),
    ("Prod002", 20, 530, "New York")
]

# 2. Define the schema string
SalesdataColumns = "product string, quantity int, salesamount int, state string"

# 3. Create the DataFrame
salesdf = spark.createDataFrame(data=Salesdata, schema=SalesdataColumns)

# 4. Display the results
salesdf.display()
```

```python
from pyspark.sql.functions import *
from pyspark.sql.window import Window

window_criteria=Window.partitionBy("state").orderBy(desc("salesamount"))
final_result = salesdf.withColumn("rank",rank().over(window_criteria))
display(final_result)
```

```python
final_result = salesdf.withColumn("rank_dense",dense_rank().over(window_criteria))
display(final_result)
```

```python
final_result = salesdf.withColumn("row_num",row_number().over(window_criteria))
display(final_result)
```

```python
final_result = salesdf.withColumn("previous_Value",lag("salesamount",1).over(window_criteria)) \
                      .withColumn("next_Value",lead("salesamount",1).over(window_criteria))
display(final_result)
```

```python
window_criteria_2 = Window.partitionBy("state")
final_result = (
    salesdf
    .withColumn("total_sales", sum("salesamount").over(window_criteria_2))
    .withColumn("avg_sales", round(avg("salesamount").over(window_criteria_2), 2))
    .withColumn("minimum_sales", min("salesamount").over(window_criteria_2))
    .withColumn("maximum_sales", max("salesamount").over(window_criteria_2))
)
display(final_result.orderBy(desc("total_sales")))
```

```python
append_events = oct_events.unionByName(nov_events)
display(append_events.limit(10))
```

```python
display("oct_events total rows : ",oct_events.count())
display("nov_events total rows : ",nov_events.count())

display("appnd_tables rows : ",append_events.count())
```

```python
event_type_sales_million = (
    append_events
    .filter(col("event_type") == "purchase")
    .groupBy("event_type")
    .agg(
        round(sum("price") / 1_000_000, 2).alias("total_sales_mn"),
        round(count("*") / 1_000_000, 2).alias("total_orders_mn")
    )
)
display(event_type_sales_million)
```

```python
brand_sales = (
    append_events
    .filter(
        (col("event_type") == "purchase") &
        (col("brand").isNotNull())
    )
    .groupBy("brand")
    .agg(
        round(sum("price") / 1_000_000, 2).alias("total_sales_mn"),
        count("*").alias("total_orders")
    )
)
```

```python
display(brand_sales.orderBy(desc("total_sales_mn")))
```

```python
window_spec = (
    Window
    .partitionBy("brand")
    .orderBy("event_time")
    .rowsBetween(Window.unboundedPreceding, Window.currentRow)
)
```

```python
running_sales_df = (
    append_events
    .filter(
        (col("event_type") == "purchase") &
        (col("brand").isNotNull())
        )
    .withColumn("running_sales", sum("price").over(window_spec))
)
```

```python
display(running_sales_df.select("brand", "event_time", "price", "running_sales").limit(30))
```

```python
from pyspark.sql import functions as F, types as T

rows_customers = [
    (1, "Asha", "IN", True),
    (2, "Bob", "US", False),
    (3, "Chen", "CN", True),
    (4, "Diana", "US", None),
    (None, "Ghost", "UK", False),      # NULL key to dem
]

rows_orders = [
    (101, 1, 120.0, "IN"),
    (102, 1, 80.0, "IN"),
    (103, 2, 50.0, "US"),
    (104, 5, 30.0, "DE"),             # no matching cus
    (105, 3, 200.0, "CN"),
    (106, None, 15.0, "UK"),          # NULL key won't
    (107, 3, 40.0, "CN"),
    (108, 2, 75.0, "US"),
]

schema_customers = T.StructType([
    T.StructField("customer_id", T.IntegerType(), True),
    T.StructField("name",        T.StringType(),  True),
    T.StructField("country",     T.StringType(),  True),
    T.StructField("vip",         T.BooleanType(), True),
])

schema_orders = T.StructType([
    T.StructField("order_id",    T.IntegerType(), True),
    T.StructField("customer_id", T.IntegerType(), True),
    T.StructField("amount",      T.DoubleType(),  True),
    T.StructField("country",     T.StringType(),  True), # same column name to show collisions
])

df_customers = spark.createDataFrame(rows_customers, schema_customers)
df_orders    = spark.createDataFrame(rows_orders,    schema_orders)

display(df_customers)
display(df_orders)
```

```python
df_inner =df_orders.join(df_customers,on='customer_id',how='inner')
display(df_inner)
```
```python
df_left=df_orders.join(df_customers,on='customer_id',how='left')
display(df_left)
```

```python
df_full = df_orders.join(df_customers,on='customer_id',how='full')
display(df_full)
```

```python
df_full = df_orders.join(df_customers,on='customer_id',how='left_semi')
display(df_full)
```

```python
df_full = df_orders.join(df_customers,on='customer_id',how='left_anti')
display(df_full)
```
## Day 4 - Delta Lake Introduction✅

What I learned today :

What is Delta Lake ?
Delta Lake is a storage layer built on top of Parquet that makes data reliable, safe, and production-ready.

CSV / Parquet Problems :
 - No safety
 - No version history
 - Easy to break data

Delta Lake Benefits :
 - Reliable writes
 - Schema safety
 - Time travel (rollback)
-------------------------------------------------
Schema Enforcement (Delta = Strict Teacher)
Delta Lake enforces the schema you define, Bad data is rejected, not silently accepted.

Without schema enforcement :
 - CSV allows anything
 - Strings in numeric columns

With Delta :
 - Errors caught immediately
 - Cleaner pipelines
 - Less debugging
-------------------------------------------------
How ACID Transactions works in Delta Tables

1️⃣Atomicity (All or Nothing) :
 - ₹1,000 is debited from Account A
 - ₹1,000 is credited to Account B
 - If the system crashes after debiting A but before crediting B
 - Transaction is rolled back
 - Either both debit & credit happen or neither happen

2️⃣ Consistency (Rules Must Hold) :
 - A transaction must follow all business rules and constraints.
 - Account balance cannot be negative
 - Total money in system must remain constant
 - If Account A has only ₹500 →transfer of ₹1,000 is rejected

3️⃣Isolation (No dirty reads) :
 - Multiple transactions can run simultaneously without affecting each other
 - The user won’t see half-updated data
 - They see either Balance before transfer or balance after transfer

4️⃣Durability (Once Committed, Always Saved) :
 - Once a transaction is committed, it cannot be lost.
 - Let's consider transfer is successful and System crashes immediately
 - System restarts: Debit & credit still exist & Bank records remain accurate
------------------------------------------------------------------------
---------------------------------------------------
## Day 5 - Delta Lake Advanced✅

### What I learned today :

#### 1️⃣ Time Travel (Version History) :
 - Delta tables remember every change.
 - Every update or insert creates a new version
 - I can query past versions of data
 - This makes debugging, auditing, and rollback possible

#### 2️⃣ MERGE Operations (Upserts) :
 - Instead of deleting, rewriting or creating duplicates
 - I used MERGE to update existing records
 - insert new records in one command
 - This is how real-world CDC and SCD pipelines work.

#### 3️⃣ OPTIMIZE & ZORDER :
 - I learned that data isn’t just about correctness - performance matters.
 - OPTIMIZE : compacts small files
 - ZORDER : intelligently organizes data for faster filtering

#### 4️⃣ VACUUM (Cleanup) :
 - Behind every update or merge, old files remain.
 - VACUUM safely removes unused data
 - Reduces storage clutter
 - Keeps the system clean
 - Production systems need housekeeping too.

💡Using managed Delta tables (saveAsTable), I was able to :
 - avoid complex storage paths
 - skip volume-level headaches
 - focus purely on data logic
 - Databricks handled storage, metadata, versioning automatically.
------------------------------------------------------------------------
---------------------------------------------------

