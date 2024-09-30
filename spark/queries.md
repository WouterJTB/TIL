# Queries

## .withColumn

Using `.withColumn` repeatedly on a single DF can cause memory problems. From [pyspark docs](https://spark.apache.org/docs/latest/api/python/reference/pyspark.sql/api/pyspark.sql.DataFrame.withColumn.html):

> This method introduces a projection internally. Therefore, calling it multiple times, for instance, via loops in order to add multiple columns can generate big plans which can cause performance issues and even StackOverflowException. To avoid this, use select() with multiple columns at once.

There is a flake8 plugin to detect these: [flake8-pyspark-with-column](https://github.com/SemyonSinchenko/flake8-pyspark-with-column)

Workarounds:

- Use [.withColumns](https://spark.apache.org/docs/latest/api/python/reference/pyspark.sql/api/pyspark.sql.DataFrame.withColumns.html) __(only since v3.3.0)__
- Use [select](https://stackoverflow.com/a/59803508/13654956), i.e.:

    ```python
    # Use this
    df.select(
        f.max(col("original1")).over(w1).alias("some1"),
        f.lag("some1").over(w2)
    ).show()

    # Not this
    w1 = ...rangeBetween(-300, 0)
    w2 = ...rowsBetween(-1,0)

    (
        df.withColumn("some1", col(f.max("original1").over(w1)))
        .withColumn("some2", lag("some1")).over(w2)
    ).show()
    ```
