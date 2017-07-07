# 15.2. 啓用時機？[^1]

There are several settings which can cause the query planner not to generate a parallel query plan under any circumstances. In order for any parallel query plans whatsoever to be generated, the following settings must be configured as indicated.

* [max\_parallel\_workers\_per\_gather](https://www.postgresql.org/docs/10/static/runtime-config-resource.html#guc-max-parallel-workers-per-gather)must be set to a value which is greater than zero. This is a special case of the more general principle that no more workers should be used than the number configured via`max_parallel_workers_per_gather`.

* [dynamic\_shared\_memory\_type](https://www.postgresql.org/docs/10/static/runtime-config-resource.html#guc-dynamic-shared-memory-type)must be set to a value other than`none`. Parallel query requires dynamic shared memory in order to pass data between cooperating processes.

In addition, the system must not be running in single-user mode. Since the entire database system is running in single process in this situation, no background workers will be available.

Even when it is in general possible for parallel query plans to be generated, the planner will not generate them for a given query if any of the following are true:

* The query writes any data or locks any database rows. If a query contains a data-modifying operation either at the top level or within a CTE, no parallel plans for that query will be generated. This is a limitation of the current implementation which could be lifted in a future release.

* The query might be suspended during execution. In any situation in which the system thinks that partial or incremental execution might occur, no parallel plan is generated. For example, a cursor created using[DECLARE CURSOR](https://www.postgresql.org/docs/10/static/sql-declare.html)will never use a parallel plan. Similarly, a PL/pgSQL loop of the form`FOR x IN query LOOP .. END LOOP`will never use a parallel plan, because the parallel query system is unable to verify that the code in the loop is safe to execute while parallel query is active.

* The query uses any function marked`PARALLEL UNSAFE`. Most system-defined functions are`PARALLEL SAFE`, but user-defined functions are marked`PARALLEL UNSAFE`by default. See the discussion of[Section 15.4](https://www.postgresql.org/docs/10/static/parallel-safety.html).

* The query is running inside of another query that is already parallel. For example, if a function called by a parallel query issues an SQL query itself, that query will never use a parallel plan. This is a limitation of the current implementation, but it may not be desirable to remove this limitation, since it could result in a single query using a very large number of processes.

* The transaction isolation level is serializable. This is a limitation of the current implementation.

Even when parallel query plan is generated for a particular query, there are several circumstances under which it will be impossible to execute that plan in parallel at execution time. If this occurs, the leader will execute the portion of the plan below the`Gather`node entirely by itself, almost as if the`Gather`node were not present. This will happen if any of the following conditions are met:

* No background workers can be obtained because of the limitation that the total number of background workers cannot exceed[max\_worker\_processes](https://www.postgresql.org/docs/10/static/runtime-config-resource.html#guc-max-worker-processes).

* No background workers can be obtained because of the limitation that the total number of background workers launched for purposes of parallel query cannot exceed[max\_parallel\_workers](https://www.postgresql.org/docs/10/static/runtime-config-resource.html#guc-max-parallel-workers).

* The client sends an Execute message with a non-zero fetch count. See the discussion of the[extended query protocol](https://www.postgresql.org/docs/10/static/protocol-flow.html#protocol-flow-ext-query). Since[libpq](https://www.postgresql.org/docs/10/static/libpq.html)currently provides no way to send such a message, this can only occur when using a client that does not rely on libpq. If this is a frequent occurrence, it may be a good idea to set[max\_parallel\_workers\_per\_gather](https://www.postgresql.org/docs/10/static/runtime-config-resource.html#guc-max-parallel-workers-per-gather)in sessions where it is likely, so as to avoid generating query plans that may be suboptimal when run serially.

* A prepared statement is executed using a`CREATE TABLE .. AS EXECUTE ..`statement. This construct converts what otherwise would have been a read-only operation into a read-write operation, making it ineligible for parallel query.

* The transaction isolation level is serializable. This situation does not normally arise, because parallel query plans are not generated when the transaction isolation level is serializable. However, it can happen if the transaction isolation level is changed to serializable after the plan is generated and before it is executed.

---



[^1]: 　[PostgreSQL: Documentation: 10: 15.2. When Can Parallel Query Be Used?](https://www.postgresql.org/docs/10/static/when-can-parallel-query-be-used.html)
