# Compute Affinities and Populate Edge Table

1. Create the stored procedure to compute the affinities between tables. This procedure reads the information from the SQL Tuning Set and calculates the affinity between tables based on how many times they are used in SQL statements together and how they are used. The procedure does the following:

    For each table in the set that we are interested in and was accessed during the workload capture:
    * Get a list of the SQL statements that used that table, either by reading an index or the table itself
    * For each table that table was joined with, i.e., each pair of tables, work out how many times that pair were joined,
    * Work out what fraction of statements this pair was joined in
    * Apply weights for joins and executions (50% each) and calculate the total affinity between the tables,
    * Work out how many other tables it was joined to in total

    After running this procedure, we have data similar to this in the `EDGES` table (some rows and columns omitted):

    | TABLE1 | TABLE2 |  JOIN_COUNT | JOIN_EXECUTIONS | STATIC_COEFFICIENT | DYNAMIC_COEFFICIENT | TOTAL_AFFINITY 
    | --- | --- | --- | --- | --- | --- | --- 
    | STUDENTS	| COURSES	| 2	| 110	| 0.33333	| 0.45643	| 0.39488
    | STUDENTS	| ENROLLMENTS	| 2	| 120	| 0.5	| 0.52174 |	0.51087
    | FACULTY	| ROLES	| 1	| 10	| 0.09091	| 0.15385	| 0.12238


    The Procedure below computes the following affinity metrics

    **Compute Affinity Metrics**: For each table, the procedure performs the following calculations:
    - Calculates the total number of SQL statements (`all_sql`) and total number of executions (`all_executions`) for the current table and the table it is joined with.
    - Counts the number of distinct SQL statements (`join_count`) and the total number of executions (`join_executions`) where the current table and the other table are joined.
    - Computes the "static coefficient" as the ratio of `join_count` to `all_sql - join_count`.
    - Computes the "dynamic coefficient" as the ratio of `join_executions` to `all_executions - join_executions`.
    - Calculates the `total affinity` as the average of the static and dynamic coefficients.


   Modify MY_SQLTUNE_DATASET_NAME and USER_NAME for your database and Execute the following statements to create the procedure:

    ```sql
    create or replace procedure compute_affinity as
    cursor c is
    select table_name, schema from nodes;
    tblnm varchar2(128);
    ins_sql varchar2(4000);
    upd_sql varchar2(4000);
    begin
        for r in c loop
            ins_sql:= q'{
                insert into USER_NAME.edges 
                ( table_set_name
                , table1
                , schema1
                , table2
                , schema2
                , join_count
                , join_executions
                , static_coefficient
                , dynamic_coefficient
                , total_affinity) 
                select 
                    'MY_SQLTUNE_DATASET_NAME' table_set_name,
                    tbl1, 
                    'MY_SQLTUNE_DATASET_NAME', 
                    tbl2, 
                    'MY_SQLTUNE_DATASET_NAME', 
                    join_count, 
                    join_executions, 
                    round(join_count/(all_sql-join_count),5) static_coefficient, 
                    round(join_executions/(all_executions-join_executions),5) dynamic_coefficient, 
                    (round(join_count/(all_sql-join_count),5)*0.5 + 
                     round(join_executions/(all_executions-join_executions),5)*0.5) total_affinity
                from (
                    select 
                        v2.tbl1, 
                        v2.tbl2, 
                        (select sum(total_sql) 
                            from nodes 
                            where table_name=v2.tbl1 
                            or table_name=v2.tbl2 ) all_sql,
                        (select sum(total_executions) 
                            from nodes 
                            where table_name=v2.tbl1 
                            or table_name=v2.tbl2 ) all_executions,
                        v2.join_count, 
                        v2.join_executions 
                    from (
                        select 
                            v1.tbl1, 
                            v1.tbl2, 
                            count(distinct v1.sql_id) join_count, 
                            sum(v1.executions) join_executions 
                        from (
                            select distinct 
                                v.tbl1, 
                                case when v.operation='INDEX' then v.TABLE_NAME  
                                    when v.operation='TABLE ACCESS' then v.tbl2 
                                    else NULL end tbl2,
                                sql_id,
                                executions 
                            from ( 
                                select 
                                    '}'||r.table_name||q'{' tbl1, 
                                    s.object_name tbl2, 
                                    i.table_name table_name, 
                                    sql_id, 
                                    operation, 
                                    executions 
                                from dba_sqlset_plans s, all_indexes i 
                                where sqlset_name='MY_SQLTUNE_DATASET_NAME' 
                                and object_owner=upper('USER_NAME') 
                                and s.object_name = i.index_name(+) 
                                and sql_id in (
                                    select distinct sql_id 
                                    from dba_sqlset_plans 
                                    where sqlset_name='MY_SQLTUNE_DATASET_NAME' 
                                    and object_name='}'||r.table_name||q'{' 
                                    and  object_owner=upper('USER_NAME')
                                ) 
                            ) v 
                        ) v1  
                        group by v1.tbl1, v1.tbl2   
                        having v1.tbl2 is not null 
                        and v1.tbl1 <> v1.tbl2 
                    ) v2 
                )
            }';
            execute immediate ins_sql;

            upd_sql:= q'{
                update USER_NAME.nodes 
                set tables_joined=(select count(distinct table_name) 
                from (
                    select 
                        'MY_SQLTUNE_DATASET_NAME' table_set_name,
                        case when v.operation='INDEX' then v.TABLE_NAME 
                            when v.operation='TABLE ACCESS' then v.object_name 
                            else NULL end table_name,
                        v.object_owner as table_owner,
                        v.sql_id, 
                        v.executions 
                    from ( 
                        select 
                            p.object_name, 
                            p.operation, 
                            p.object_owner, 
                            p.sql_id, 
                            p.executions, 
                            i.table_name 
                        from dba_sqlset_plans p, all_indexes i 
                        where p.object_name=i.index_name(+) 
                        and sqlset_name='MY_SQLTUNE_DATASET_NAME' 
                        and sql_id in (
                            select sql_id 
                            from tableset_sql 
                            where table_name='}'||r.table_name||q'{') 
                            and object_owner = upper('USER_NAME')
                        ) v
                    )
                ) where table_name ='}' || r.table_name || q'{'
            }';
            execute immediate upd_sql;
        end loop;

    end;
    /
    ```

	**Procedure `COMPUTE_AFFINITY_TKDRA compiled`** is the expected output.

     this stored procedure is part of a system that analyzes the relationships and affinities between tables in a database, based on the SQL statements that access those tables. The computed affinity metrics are stored in the `USER_NAME.edges` table, and the number of joined tables is updated in the `USER_NAME.nodes` table

2. Run the procedure to compute affinities.

    ```text
    exec compute_affinity();
    ```

    This may take a few minutes to complete. **PL/SQL procedure successfully completed** is the expected output. 
	Once it is done, we can see the output in the `EDGE` table, For example:

    ```sql
    select * from edges where table1 = 'STUDENTS';
    ```

3. Add the constraints for the newly created tables, where TABLE1 and TABLE2 of EDGES table are foreign keys referencing the TABLE_NAME column of the NODES table.
	```sql
	ALTER TABLE NODES ADD PRIMARY KEY (TABLE_NAME);

	ALTER TABLE EDGES ADD TABLE_MAP_ID NUMBER;
	UPDATE EDGES SET TABLE_MAP_ID = ROWNUM;
	COMMIT;

	ALTER TABLE EDGES ADD PRIMARY KEY (TABLE_MAP_ID);
	ALTER TABLE EDGES MODIFY TABLE1 REFERENCES NODES (TABLE_NAME);
	ALTER TABLE EDGES MODIFY TABLE2 REFERENCES NODES (TABLE_NAME);
	COMMIT;
    ```


Congratulations! You have successfully populated the tables used to create the graph.  Next Step, create a graph