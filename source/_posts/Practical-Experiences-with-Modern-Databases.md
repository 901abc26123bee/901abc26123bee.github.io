---
title: Practical Experiences with Modern Databases
date: 2024-06-12 22:57:40
categories:
tags:
- Database
- PostgreSQL
- MSSQL
---
# Practical Experiences with Modern Databases
## DeadLock in Transaction
Prevention:
1. **Order Resource Acquisition Consistently:**
  Ensure that resources are acquired in a consistent order across transactions to avoid circular wait conditions.

    ```
    Transaction 1 (T1):
    update(Resource A)
    update(Resource B)

    Transaction 2 (T2):
    update(Resource A)
    update(Resource B)
    ```
    In this example, both T1 and T2 acquire Resource A first, ensuring there is no circular wait condition:

    - T1 acquires Resource A and waits for Resource B.
    - T2 acquires Resource B and waits for Resource A.

    Execute table or row operations in a consistent order within transactions to avoid deadlocks between different transactions.
    
    For more information on PostgreSQL lock monitoring and handling lock waits, refer to:
    -  [5mins of Postgres E25: Postgres lock monitoring, LWLocks and the log_lock_waits setting](https://pganalyze.com/blog/5mins-postgres-lock-monitoring-lwlock-log-lock-waits)
    - [How to stop/kill a query in postgresql?](https://stackoverflow.com/questions/35319597/how-to-stop-kill-a-query-in-postgresql)

2. **Implement a Timeout Mechanism:**
  Use a timeout mechanism to abort transactions that have been waiting too long, preventing long waits and potential deadlocks. For example, set `context.WithTimeout` in Golang for the root context.

3. **Move Query Operations Outside of Transactions if Possible:**
Whenever possible, perform query operations outside of transactions to reduce the risk of deadlocks. Additionally, shorten the duration of transactions.

4. **Avoid Long Transaction**

## Time and Timezone In Database

**Timestamp Format**

- Store Timestamps in UTC:
    It ensures consistency and avoids complications arising from daylight saving time changes and other timezone-related issues.
- Set the Database Timezone to UTC:
    If your database system allows setting the timezone, setting it to UTC can enforce consistency and avoid unexpected behavior in date and time functions.
- Handle Timezones at the Application Level:
    Manage timezone conversions within your application. Store timestamps in UTC in the database and convert to the appropriate local time zone when needed for display or user interactions

**Epoch Format**

- Epoch time, representing the number of seconds (or milliseconds) since a fixed point (1970-01-01 00:00:00 UTC), can be more efficient for storage and quicker for comparison operations. However, since epoch time does not inherently include time zone information or formatting, it should only be used if your application is completely time zone independent.
## Default Connection Limit

For microservices, it is crucial to be aware that each service has a default connection limit for both the database server and the client. As the number of services increases, you may encounter bottlenecks because all available connections can become occupied.

Use PostgreSQL as an example
- **Client Side:** 
  If you are using an ORM package, it may provide relevant settings for the client service.
  ```go
  ex: gorm:
    sqlDB.SetMaxIdleConns(defaultMaxIdleConn)
    sqlDB.SetMaxOpenConns(defaultMaxOpenConn)
    sqlDB.SetConnMaxLifetime(defaultLifetime)
  ```

- **Server Side:**
  - [How to increase the max connections in postgres?](https://stackoverflow.com/questions/30778015/how-to-increase-the-max-connections-in-postgres)
  - [20.3. Connections and Authentication](https://www.postgresql.org/docs/current/runtime-config-connection.html#RUNTIME-CONFIG-CONNECTION)

  ```
  max_connections (integer) 
  Determines the maximum number of concurrent connections to the database server. The default is typically 100 connections.
  ```

## Using Shorter Keys in Indexes
Using shorter keys in B-Tree indexes offers multiple benefits, including improved performance, efficient memory and storage usage, and better cache utilization.

  **MySQL B+ Tree Indexes:**
  MySQL uses B+ Tree indexes, which are a variation of B-Tree indexes with additional optimizations. These indexes ensure that all values are stored at the leaf level and linked in a doubly linked list, enhancing range query performance and efficiency.

  **PostgreSQL B-Tree Indexes:**
  PostgreSQL utilizes B-Tree indexes to efficiently manage and query large datasets. The structure and operation of these indexes can be explored in detail in the PostgreSQL documentation:
  [PostgreSQL B-Tree Implementation doc](https://www.postgresql.org/docs/current/btree-implementation.html#:~:text=PostgreSQL%20B%2DTree%20indexes%20are,leaf%20pages%20or%20internal%20pages)

## Create Views for Frequently Used Join Queries.

By encapsulating frequently used queries, views ensure consistent results and reduce the need for query repetition. They can be reused across different parts of the application.

[**Why is database view used?**](https://stackoverflow.com/questions/2454113/why-is-database-view-used)

## Database Parameter Limits
When dealing with large datasets, there's a risk of hitting database parameter limits, which can cause performance bottlenecks. By dividing large operations into smaller chunks, systems can avoid hitting these limits while maintaining performance and scalability.
  - **PostgreSQL**
  [Passing the Postgres 65535 parameter limit at 120x speed](https://klotzandrew.com/blog/postgres-passing-65535-parameter-limit)
    ```go
    // gorm provide CreateInBatches function to dividing large operations into smaller chunks
    // CreateInBatches insert the value in batches of batchSize
    CreateInBatches(value interface{}, batchSize int) DB
    ```

  - **MSSQL**
    ```go
    // define mssql parameter limit, not to exceed query limit 2100
    MSSQL_PARAMETER_LIMIT = 2048
    // ChunkSlice chunk slice into chunks with size
    func ChunkSlice(slice []int64, chunkSize int) [][]int64 {
      var chunks [][]int64
      for i := 0; i < len(slice); i += chunkSize {
        end := i + chunkSize
        // necessary check to avoid slicing beyond
        // slice capacity
        if end > len(slice) {
          end = len(slice)
        }
        chunks = append(chunks, slice[i:end])
      }

      return chunks
    }

    // usage
    chunks := common.ChunkSlice(req.GetIds(), common.MSSQL_PARAMETER_LIMIT)
    for _, chunk := range chunks {
      // query restored student plan IDs are all in same school plan
      var users []*db.Users
      if err := s.db.WithContext(ctx).Model(&db.Users{}).Where("id IN (?) AND deleted=0", chunk).Scan(&users).Error; err != nil {
        return nil, errors.Errorf("failed to query users by ids, err: %v", err)
      }
      // ... fellowing operation
    }
    ```

## Avoid Generating Temporary Tables for Small Joins
When performing a join between tables that are not large, it is generally advisable to avoid generating temporary tables by selecting columns in the join condition.

  **PostgreSQL**
  [Explaining Your Postgres Query Performance](https://www.crunchydata.com/blog/get-started-with-explain-analyze)
  
  **MSSQL**
  - **generate temporary tables**
    ```sql
    SET SHOWPLAN_XML ON;
    GO

    -- Your query here
    SELECT 
        users.*, 
        a.auth_id, 
        a.auth_deleted, 
        a.auth_create_user, 
        a.auth_create_time, 
        a.auth_belong_to
    FROM 
        users
    LEFT JOIN (
        SELECT 
            a.id AS auth_id,
            a.plan_id AS auth_plan_id,
            a.user_id AS auth_user_id,
            a.role AS auth_role,
            a.belong_to AS auth_belong_to,
            a.deleted AS auth_deleted,
            a.create_user AS auth_create_user,
            a.create_time AS auth_create_time
        FROM 
            auths a
        WHERE 
            a.plan_id = 3 
            AND a.role = 6 
            AND a.deleted = 0 
            AND a.belong_to = 3794
    ) AS a 
    ON 
        a.auth_user_id = users.id
    WHERE  
        users.role = 6 
        AND users.school_code = '014666';
    GO

    SET SHOWPLAN_XML OFF;
    GO

    ```
    <img src="./join_table_1.png"  width="100%" height="100%" title="join_table_1" alt="join_table_1"/>

  - **no temporary tables are generated**

    ```sql
    SET SHOWPLAN_XML ON;
    GO

    -- Your query here
    select u.id, u.name, u.email, a.*
            from users u
            left join auths a ON a.user_id=u.id and a.plan_id=3 and a.role=u.role and a.deleted=0 and a.belong_to=3794
            where u.role=6 and u.school_code='014666';
    GO

    SET SHOWPLAN_XML OFF;
    GO
    ```
    <img src="./join_table_2.png"  width="100%" height="100%" title="join_table_2" alt="join_table_2"/>
