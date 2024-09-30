# Airflow Variables

## No `Variable.get()` in DAGs

From [airflow docs](https://airflow.apache.org/docs/apache-airflow/2.7.3/best-practices.html#airflow-variables):

> Using Airflow Variables yields network calls and database access, so their usage in top-level Python code for DAGs should be avoided as much as possible, as mentioned in the previous chapter, Top level Python Code. If Airflow Variables must be used in top-level DAG code, then their impact on DAG parsing can be mitigated by enabling the experimental cache, configured with a sensible ttl.
>
> You can use the Airflow Variables freely inside the execute() methods of the operators, but you can also pass the Airflow Variables to the existing operators via Jinja template, which will delay reading the value until the task execution.
