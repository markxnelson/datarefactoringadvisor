# Collect Database Workload

## CREATE SQL TUNING SET

The provided PL/SQL code is using the `DBMS_SQLTUNE` package in Oracle to create a SQL Tuning Set (STS) named `tkdradata`. A SQL Tuning Set is a database object that contains a set of SQL statements along with their execution statistics and context information.

Here's a breakdown of what the code does:

```sql
BEGIN
  DBMS_SQLTUNE.CREATE_SQLSET (
      sqlset_name => 'MY_SQLTUNE_DATASET_NAME', 
      description => 'SQL data from MY_DATABASE_NAME schema'
  );
END;
/
```

1. `BEGIN` and `END;` are the delimiters that define the start and end of the PL/SQL block.

2. `DBMS_SQLTUNE.CREATE_SQLSET` is a procedure from the `DBMS_SQLTUNE` package that creates a new SQL Tuning Set.

3. `sqlset_name => 'MY_SQLTUNE_DATASET_NAME'` is a parameter that specifies the name of the SQL Tuning Set to be created. In this case, the name is `MY_SQLTUNE_DATASET_NAME`.

4. `description => 'SQL data from MY_DATABASRE_NAME schema'` is an optional parameter that provides a description for the SQL Tuning Set. Here, the description is "SQL data from tkdradata schema".

5. The `/` at the end is a terminator that signals the end of the PL/SQL block in Oracle.

After executing this code, a new SQL Tuning Set named `MY_SQLTUNE_DATASET_NAME` will be created in the database. Initially, it will be empty, but you can populate it with SQL statements and their execution statistics using other procedures from the `DBMS_SQLTUNE` package.

SQL Tuning Sets are useful for various purposes, such as:

1. **Data Refactoring**: You can analyze the SQL statements in the tuning set to create a graph of join activity between tables.  The more join activity, the higher the affinity between the 2 tables.  Run community detection on the graph to iddentify communities (i.e. a bounded context for a microservice)

2. **SQL Tuning**: You can analyze the SQL statements in the tuning set to identify performance issues and apply tuning techniques like creating indexes, restructuring queries, or using hints.

3. **Workload Capture and Replay**: SQL Tuning Sets can be used to capture a production workload and replay it in a test environment for testing or tuning purposes.

Overall, the provided PL/SQL code is a preparatory step for working with SQL Tuning Sets in Oracle, which can be a valuable tool for SQL performance analysis and tuning.
