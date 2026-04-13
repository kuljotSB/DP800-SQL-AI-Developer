### Analyze Query Performance with Execution Plans and DMVs

#### Scenario

You are a SQL AI Developer at CarbonOps.

Your ESG platform is slowing down due to:

1) Large emission datasets
2) Inefficient queries
3) Missing indexes

our goal: Identify and optimize slow queries using Execution Plans + DMVs

### Create Table and Add Data

```sql
CREATE TABLE ESG.TempEmissions
(
    RecordID INT IDENTITY(1,1) PRIMARY KEY,
    CompanyID INT,
    EmissionDate DATE,
    CO2_Emissions FLOAT,
);
```

```sql
INSERT INTO ESG.TempEmissions (CompanyID, EmissionDate, CO2_Emissions)
SELECT TOP 1000
    ABS(CHECKSUM(NEWID())) % 5 + 1,
    DATEADD(DAY, ABS(CHECKSUM(NEWID())) % 365, '2024-01-01'),
    ABS(CHECKSUM(NEWID())) % 1000
FROM sys.objects a CROSS JOIN sys.objects b;
```

#### Estimated Execution Plan

```sql
SET SHOWPLAN_XML ON;
GO
```

Query:
```sql
SELECT *
FROM ESG.TempEmissions
WHERE CompanyID = 3;
```

Disable Showplan:
```sql
SET SHOWPLAN_XML OFF;
GO
```

#### Actual Execution Plan

```sql
SET STATISTICS XML ON;
```

Query:
```sql
SELECT *
FROM ESG.TempEmissions
WHERE CompanyID = 3;
```


#### Fix with Index

```sql
CREATE NONCLUSTERED INDEX IX_Emission_CompanyID
ON ESG.TempEmissions (CompanyID);
```

Re-run query and compare execution plans before vs after index creation.
```sql
SELECT CompanyID, EmissionDate
FROM ESG.TempEmissions
WHERE CompanyID = 3;
```

Recreate the index by dropping it first:
```sql
DROP INDEX IX_Emission_CompanyID ON ESG.TempEmissions;
```

Add Emission Date to non clustered index:
```sql
CREATE NONCLUSTERED INDEX IX_Emission_CompanyID
ON ESG.TempEmissions (CompanyID)
INCLUDE (EmissionDate);
```

Re-run the query again:
```sql
SELECT CompanyID, EmissionDate
FROM ESG.TempEmissions
WHERE CompanyID = 3;
```

#### Use DMVs - Find Expensive Queries

```sql
SELECT TOP 10
    qs.total_worker_time / qs.execution_count AS avg_cpu_time,
    qs.execution_count,
    qs.total_logical_reads / qs.execution_count AS avg_reads,
    st.text AS query_text
FROM sys.dm_exec_query_stats qs
CROSS APPLY sys.dm_exec_sql_text(qs.sql_handle) st
WHERE st.text LIKE '%ESG%'
ORDER BY avg_cpu_time DESC;
```

