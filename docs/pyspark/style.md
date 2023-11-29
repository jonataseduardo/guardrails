
### Column selection 

Use implicit column access on PySpark functions because this style is simpler, shorter, and visually less cluttered.

```python
df = df.select('col_a', F.lower('col_b'), F.upper('col_c'))
```

It is recommended to use `.withColumn()` for single columns. However, when adding or manipulating tens or hundreds of columns, it is advisable to use a single `.select()` for better performance.

## Empty and constant columns

If you need to add an empty column to satisfy a schema, always use `F.lit(None)` for populating that column. To set a columns to a constant value you should user `F.lit(value)`

```python
df = df.withColumn('foo', F.lit(None))
```

### Specify a schema contract

Use the `select` statement before transformations. This specifies a data contract. 
Keep the select statements as simple as possible. Use an `alias` to name a new column or rename an old one. To change the type of a column, use the `cast` operator.

```python
df = df.select(
    "column0",
    F.col("column1").alias("new_column1"),
    F.col("column2").cast("int").alias("new_column2"),
    F.col("column3").cast("float").alias("new_column3")
)

```

### Logical operations

Logical operations, which are often found within `.filter()` or `F.when()`, should be easy to read. It is recommended to keep logic expressions within the same code block to a maximum of three (3) expressions. If they become longer, it is advisable to extract complex logical operations into variables for improved readability.

```python
has_operator = ((F.col('originalOperator') != '') | (F.col('currentOperator') != ''))
delivery_date_passed = (F.datediff('deliveryDate_actual', 'current_date') < 0)
has_registration = (F.col('currentRegistration').rlike('.+'))
is_delivered = (F.col('prod_status') == 'Delivered')
is_active = (has_registration | has_operator)

F.when(is_delivered | (delivery_date_passed & is_active), 'In Service')
```

## Joins

When joining dataframes, it is recommended to specify the `how` option explicitly, even if the default is `(inner)`. It is preferable to use left joins instead of right joins for better code readability. Similar to SQL, to avoid collisions in column names, assign an alias to the entire dataframe and select the desired columns using the alias.


```python
flights = flights.alias('flights')
parking = parking.alias('parking')

flights = flights.join(parking, on='flight_code', how='left')

flights = flights.select(
    F.col('flights.start_time').alias('flight_start_time'),
    F.col('flights.end_time').alias('flight_end_time'),
    F.col('parking.total_time').alias('client_parking_total_time')
)
```

## Chaining of expressions

Please exercise caution when chaining expressions into multi-line expressions if they have different behaviors or contexts. For example, it is advisable to avoid combining column creation or joining with selecting and filtering. This will enhance the readability of your code and will not impact performance if you utilize Spark's lazy evaluation.

```python
df = (
    df
    .select('a', 'b', 'c', 'key')
    .filter(F.col('a') == 'truthiness')
)

df = df.withColumn('boverc', F.col('b') / F.col('c'))

df = (
    df
    .join(df2, 'key', how='inner')
    .join(df3, 'key', how='left')
    .drop('c')
)
```

If you find you are making longer chains, or having trouble because of the size of your variables, consider extracting the logic into a separate function:

```python
def join_customers_with_shipping_address(customers, df_to_join):

    customers = (
        customers
        .select('a', 'b', 'c', 'key')
        .filter(F.col('a') == 'truthiness')
    )

    customers = customers.withColumn('boverc', F.col('b') / F.col('c'))
    customers = customers.join(df_to_join, 'key', how='inner')
    return customers
```


## Window Functions

Always specify an explicit frame when using window functions, using either [row frames](https://spark.apache.org/docs/latest/api/java/org/apache/spark/sql/expressions/WindowSpec.html#rowsBetween-long-long-) or [range frames](https://spark.apache.org/docs/latest/api/java/org/apache/spark/sql/expressions/WindowSpec.html#rangeBetween-long-long-). If you do not specify a frame, Spark will generate one, in a way that might not be easy to predict. In particular, the generated frame will change depending on whether the window is ordered (see [here](https://github.com/apache/spark/blob/v3.0.1/sql/catalyst/src/main/scala/org/apache/spark/sql/catalyst/analysis/Analyzer.scala#L2899)). To see how this can be confusing, consider the following example:

```python
from pyspark.sql import functions as F, Window as W
df = spark.createDataFrame([('a', 1), ('a', 2), ('a', 3), ('a', 4)], ['key', 'num'])

# bad
w1 = W.partitionBy('key')
w2 = W.partitionBy('key').orderBy('num')
 
df.select('key', F.sum('num').over(w1).alias('sum')).collect()
# => [Row(key='a', sum=10), Row(key='a', sum=10), Row(key='a', sum=10), Row(key='a', sum=10)]

df.select('key', F.sum('num').over(w2).alias('sum')).collect()
# => [Row(key='a', sum=1), Row(key='a', sum=3), Row(key='a', sum=6), Row(key='a', sum=10)]

df.select('key', F.first('num').over(w2).alias('first')).collect()
# => [Row(key='a', first=1), Row(key='a', first=1), Row(key='a', first=1), Row(key='a', first=1)]

df.select('key', F.last('num').over(w2).alias('last')).collect()
# => [Row(key='a', last=1), Row(key='a', last=2), Row(key='a', last=3), Row(key='a', last=4)]
```

It is much safer to always specify an explicit frame:
```python
# good
w3 = W.partitionBy('key').orderBy('num').rowsBetween(W.unboundedPreceding, 0)
w4 = W.partitionBy('key').orderBy('num').rowsBetween(W.unboundedPreceding, W.unboundedFollowing)
 
df.select('key', F.sum('num').over(w3).alias('sum')).collect()
# => [Row(key='a', sum=1), Row(key='a', sum=3), Row(key='a', sum=6), Row(key='a', sum=10)]

df.select('key', F.sum('num').over(w4).alias('sum')).collect()
# => [Row(key='a', sum=10), Row(key='a', sum=10), Row(key='a', sum=10), Row(key='a', sum=10)]

df.select('key', F.first('num').over(w4).alias('first')).collect()
# => [Row(key='a', first=1), Row(key='a', first=1), Row(key='a', first=1), Row(key='a', first=1)]

df.select('key', F.last('num').over(w4).alias('last')).collect()
# => [Row(key='a', last=4), Row(key='a', last=4), Row(key='a', last=4), Row(key='a', last=4)]
```

### Dealing with nulls

Be explicit about the `ignorenulls` flag.

```python
df_nulls.select('key', F.first('num', ignorenulls=True).over(w4).alias('first')).collect()
# => [Row(key='a', first=1), Row(key='a', first=1), Row(key='a', first=1), Row(key='a', first=1)]

df_nulls.select('key', F.last('num', ignorenulls=True).over(w4).alias('last')).collect()
# => [Row(key='a', last=2), Row(key='a', last=2), Row(key='a', last=2), Row(key='a', last=2)]
```

Also be mindful of explicit ordering of nulls to make sure the expected results are obtained:
```python
w5 = W.partitionBy('key').orderBy(F.asc_nulls_first('num')).rowsBetween(W.currentRow, W.unboundedFollowing)
w6 = W.partitionBy('key').orderBy(F.asc_nulls_last('num')).rowsBetween(W.currentRow, W.unboundedFollowing)

df_nulls.select('key', F.lead('num').over(w5).alias('lead')).collect()
# => [Row(key='a', lead=None), Row(key='a', lead=1), Row(key='a', lead=2), Row(key='a', lead=None)]

df_nulls.select('key', F.lead('num').over(w6).alias('lead')).collect()
# => [Row(key='a', lead=1), Row(key='a', lead=2), Row(key='a', lead=None), Row(key='a', lead=None)]
```

### Empty `partitionBy()`

Spark window functions can be applied to all rows using a global frame. This is achieved by specifying zero columns in the partition by expression, like `W.partitionBy()`.

However, it is advisable to avoid using code like this as it compels Spark to merge all data into a single partition, which can severely impact performance.

Whenever possible, it is recommended to use aggregations instead.

```python
# bad
w = W.partitionBy()
df = df.select(F.sum('num').over(w).alias('sum'))

# good
df = df.agg(F.sum('num').alias('sum'))
```
