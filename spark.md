# (Py)Spark

## Dependency matrix

[Python and (Py)Spark](https://community.cloudera.com/t5/Community-Articles/Spark-Python-Supportability-Matrix/ta-p/379144)

Some key takeaways:

- Python 3.11 can only be used with Spark >= 3.4.0
- Python 3.10 can only be used with Spark >= 3.3.0
- Python 3.9 can only be used with Spark >= 3.0.2

[Spark, Scala, JDK](https://community.cloudera.com/t5/Community-Articles/Spark-Scala-Version-Compatibility-Matrix/ta-p/383713)

## Queries

### .withColumn

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

## Clusters

### Spark on k8s

#### Resource management

- Drivers can not span multiple nodes
- Ideally, the Spark Executor resources (including overhead) are a divisor of the allocatable k8s node resources, so that you don't leave unused resources on a node (if this holds true for all jobs in the nodepool, resources can still be allocated somewhat efficiently). AWS has a [useful blog on this](https://aws.amazon.com/blogs/containers/optimizing-spark-performance-on-kubernetes/).
- Drivers and executors [can be assigned to a different nodepool](https://github.com/kubeflow/spark-operator/issues/1471). I.e. you can run executors on a spot instance nodepool while running drivers on on-demand instances.
- For (now only on) AWS, consider [Karpenter](https://karpenter.sh/)
- Few/fat nodes/executors vs. many/small nodes/executors:
  - Many/small nodes/executors still need to communicate, communication between driver/executors on the same pod will be fast (near instant). Communication between nodes will have a (likely very minor) latency cost.
  - Many/small nodes/executors should be more fault tolerant (less work lost if it crashes).
  - Take into account parellelization of the job vs number of shuffles.
