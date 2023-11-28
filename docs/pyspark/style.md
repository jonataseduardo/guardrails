
# Use `select` statements to specify a schema contract

Create a PySpark snippet that adheres to the following instructions:

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

## Prefer implicit column selection to direct access

This style is simpler, shorter and visually less polluted. 
```python
df = df.select('colA', F.lower('colB'), F.upper('colC'))
```

It is recommended to use `.withColumn()` for single columns. However, when adding or manipulating tens or hundreds of columns, it is advisable to use a single `.select()` for better performance.

## Empty and constant columns

If you need to add an empty column to satisfy a schema, always use `F.lit(None)` for populating that column. To set a columns to a constant value you should user `F.lit(value)`

```python
df = df.withColumn('foo', F.lit(None))
```

# Logical operations

Logical operations, which are often found within `.filter()` or `F.when()`, should be easy to read. It is recommended to keep logic expressions within the same code block to a maximum of three (3) expressions. If they become longer, it is advisable to extract complex logical operations into variables for improved readability.

```python
has_operator = ((F.col('originalOperator') != '') | (F.col('currentOperator') != ''))
delivery_date_passed = (F.datediff('deliveryDate_actual', 'current_date') < 0)
has_registration = (F.col('currentRegistration').rlike('.+'))
is_delivered = (F.col('prod_status') == 'Delivered')
is_active = (has_registration | has_operator)

F.when(is_delivered | (delivery_date_passed & is_active), 'In Service')
```
