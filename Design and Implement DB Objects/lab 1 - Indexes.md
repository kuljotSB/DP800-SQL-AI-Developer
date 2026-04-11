### Creating Clustered and Non-Clustered Indexes for Performance Optimization

#### Scenario
You are a SQL AI Developer at CarbonOps.

As your ESG platform scales, queries analyzing emissions data are becoming slow.

The analytics team reports:

1) Queries filtering by CompanyID are slow
2) Sorting by EmissionDate takes time
3) Aggregations over large datasets are inefficient

Your task is to:

1) Implement clustered and non-clustered indexes
2) Measure performance improvements
3) Understand how indexing affects query execution

#### Create a High Volume Table

Create ESG.EmissionRecords_Large Table:
```sql
CREATE TABLE ESG.EmissionRecords_Large (
    RecordID INT IDENTITY(1,1),
    CompanyID INT NOT NULL,
    EmissionDate DATE NOT NULL,
    Scope TINYINT NOT NULL,
    CO2_Emissions DECIMAL(10,2) NOT NULL,
    Source NVARCHAR(50),

    CONSTRAINT PK_Emission_Large PRIMARY KEY NONCLUSTERED (RecordID)
);
```

Insert Multiple Rows to simulate a large dataset:
```sql
SET NOCOUNT ON;

DECLARE @i INT = 1;

WHILE @i <= 1000
BEGIN
    INSERT INTO ESG.EmissionRecords_Large (CompanyID, EmissionDate, Scope, CO2_Emissions, Source)
    VALUES (
        (ABS(CHECKSUM(NEWID())) % 8) + 1,
        DATEADD(DAY, - (ABS(CHECKSUM(NEWID())) % 365), GETDATE()),
        (ABS(CHECKSUM(NEWID())) % 3) + 1,
        CAST((RAND(CHECKSUM(NEWID())) * 5000) AS DECIMAL(10,2)),
        'Generated Data'
    );

    SET @i = @i + 1;
END;
```

#### Run Query without Index (Baseline)

```sql
SET STATISTICS TIME ON;
SET STATISTICS IO ON;

SELECT *
FROM ESG.EmissionRecords_Large
WHERE CompanyID = 3
ORDER BY EmissionDate;
```

#### Create Clustered Index

Cluster on EmissionDate for faster sorting
```sql
CREATE CLUSTERED INDEX IX_EmissionDate
ON ESG.EmissionRecords_Large (EmissionDate);
```

Re-run query (optimized for sorting)
```sql
SELECT *
FROM ESG.EmissionRecords_Large
WHERE CompanyID = 3
ORDER BY EmissionDate;
```

#### Create Non-Clustered Index

Index on CompanyID for faster filtering
```sql
CREATE NONCLUSTERED INDEX IX_CompanyID
ON ESG.EmissionRecords_Large (CompanyID);
```

Re-run query (optimized for filtering)
```sql
SELECT CompanyID
FROM ESG.EmissionRecords_Large
WHERE CompanyID = 3;
```

#### View Execution Plan

```sql
SET SHOWPLAN_ALL ON;
```

now run query again to see how indexes are used in the execution plan.
