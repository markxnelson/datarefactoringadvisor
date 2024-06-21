# Create Graph Tables

Create metadata tables (Nodes and Edges) and populate the data using data from the SQL Tuning Set created in the previous lab. Compute the affinities based on `JOIN` activity between the tables.

## Task 1: Create Graph Metadata Tables

In this task, we will create a set of metadata tables that we will use to store the information we need to perform community detection in the next lab. We will create a table called `NODES` that will contain a list of all the tables used in the workload capture and how many times each table was accessed and/or participated in a join. A second table called `EDGES` is used to store the affinities between pairs of tables. Later, when we create a graph, the first table will describe the vertices, and the second table will define the edges.

1. Create the graph metadata tables by running the following statements - make sure you run these from `YOUR` login SQL Worksheet (not the `ADMIN` user's worksheet):

    ```sql
    drop table nodes;
    drop table edges;

    create table nodes 
    ( table_set_name       varchar2(128)
    , schema               varchar2(128)
    , table_name           varchar2(128)
    , total_sql            number(10)
    , total_executions     number(10)
    , tables_joined        number(10));

    create table edges 
    ( table_set_name       varchar2(128)
    , table1               varchar2(128)
    , schema1              varchar2(128)
    , table2               varchar2(128)
    , schema2              varchar2(128)
    , join_count           number(10)
    , join_executions      number(10)
    , static_coefficient   decimal(10,5)
    , dynamic_coefficient  decimal(10,5)
    , total_affinity       decimal(10,5));
    ```


These two tables, `nodes` and `edges` are designed to store information about tables, their relationships (joins), and various metrics related to SQL statements and executions involving these tables.

The `nodes` table stores information about individual tables, such as their names, schemas, and metrics like the total number of SQL statements and executions involving each table.

The `edges` table stores information about join relationships between pairs of tables, including the table names, schemas, join counts, execution counts, and various coefficient and affinity values related to the join relationships.

These tables are used for identifying frequently joined tables and understanding the relationships and dependencies between tables in a database schema. 


2\. In this step, we will populate the `NODES` table, which will become the vertices in our graph. Execute the following commands to populate the table:

    ```sql
    truncate table nodes;

    insert into nodes (table_set_name, schema, table_name, total_sql, total_executions) 
    select table_set_name, table_owner, table_name, count(distinct sql_id), sum(executions)
    from ( 
        select distinct table_set_name, table_owner, table_name, sql_id, executions 
        from (
            select 'MY_SQLTUNE_DATASET_NAME' table_set_name,
                case when v.operation='INDEX' then v.TABLE_NAME
                     when v.operation='TABLE ACCESS' then v.object_name
                     else NULL end table_name,
                v.object_owner as table_owner,
                v.sql_id,
                v.executions
            from (
                select p.object_name, p.operation, p.object_owner, 
                    p.sql_id, p.executions, i.table_name
                from dba_sqlset_plans p, all_indexes i
                where p.object_name=i.index_name(+) 
                and sqlset_name='MY_SQLTUNE_DATASET_NAME'
                and object_owner = upper('USER_NAME')
            ) v  
        )
    ) 
    group by table_set_name, table_owner, table_name
    having table_name is not null;
    ```

In summary, this SQL code populates the `nodes` table with data derived from the `DBA_SQLSET_PLANS` and `ALL_INDEXES` views, specifically for the SQL Tuning Set named `MY_SQLTUNE_DATASET_NAME` and the schema `USER_NAME`. It calculates the total number of distinct SQL statements (`total_sql`) and the total number of executions (`total_executions`) for each table involved in the SQL Tuning Set, and inserts these values into the `nodes` table along with the table set name, schema, and table name.


3\. Create a helper view that we will use in the affinity calculation. Execute the following command to create the view:

```sql
create view tableset_sql as 
    select distinct table_name, sql_id 
    from (
        select 'MY_SQLTUNE_DATASET_NAME' table_set_name,
        case when v.operation='INDEX' then v.TABLE_NAME
            when v.operation='TABLE ACCESS' then v.object_name
            else NULL end table_name,
        v.object_owner as table_owner,
        v.sql_id,
        v.executions
        from ( 
            select p.object_name, p.operation, p.object_owner,
                p.sql_id, p.executions, i.table_name
            from dba_sqlset_plans p, all_indexes i
            where p.object_name=i.index_name(+) 
            and sqlset_name='MY_SQLTUNE_DATASET_NAME' 
            and object_owner = 'USER_NAME'
        ) v
    )
```
The resulting `tableset_sql` view contains distinct combinations of `table_name` and `sql_id` for the tables involved in the SQL Tuning Set named `MY_SQLTUNE_DATASET_NAME` and the schema `USER_NAME`. This view can be useful for analyzing the relationships between tables and SQL statements within the SQL Tuning Set and for performing further queries or analysis on the data.


Node populated. View created. Create the EDGES Table.

## Task 2: Compute Affinities and Populate Edge Table

[Compute Affinities and Populate Edge Table](./compute-affinities/README.md)